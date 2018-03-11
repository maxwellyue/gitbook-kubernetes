英文地址：[https://kubernetes.io/docs/concepts/configuration/overview/](https://kubernetes.io/docs/concepts/configuration/overview/)

# 配置最佳实践

---

本节文档是对用户指南、Getting Started文档以及examples的提炼。

## 通用配置技巧 {#general-configuration-tips}

---

* 定义配置的时候，请指定最新的稳定API版本。

* 配置文件在推送到集群之前，应该存储在版本控制工具中。这样你就可以在需要的时候快速地回滚某个配置。同时，这也有助于群集重建和恢复。

* 使用YAML格式，而非JSON格式来编写配置文件。尽管这两种格式几乎在所有场景都可以互换，但YAML对用户更友好。

* 只要有意义，就将相关的对象定义在同一个文件中。管理一个文件通常比管理多个文件更容易。语法可以参考[guestbook-all-in-one.yaml](https://github.com/kubernetes/examples/tree/master/guestbook/all-in-one/guestbook-all-in-one.yaml)文件。

* 很多`kubectl`命令支持目录。比如，你可以在有多个配置文件的目录中使用`kubectl create`命令。

* 不要定义一些不必要的默认值：简单、最小化的配置会带来更少的错误。

* 将对象的描述放在注释（annotations）中，方便更好地观察。

## “Naked” Pods vs ReplicaSets, Deployments, and Jobs {#naked-pods-vs-replicasets-deployments-and-jobs}

---

* 不要使用裸Pod \(意思是，那些没有绑定到[ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)或着[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)的Pods \)。裸Pods不会在节点失败后重新调度。

  创建Pods的最好方式就是使用Deployment：创建确保Pods数量的ReplicaSet，并指定更换Pods的策略（比如[RollingUpdate](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)\)。当然，对于一些明确的[`restartPolicy: Never`](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)场景而言， [Job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)可能会更适合。

## Services {#services}

---

* 在创建与[Service](https://kubernetes.io/docs/concepts/services-networking/service/)相关的后台工作负载\(Deployments 或者 ReplicaSets\)之前，也在其他工作负载需要访问该服务之前，去创建Service。当Kubernetes启动一个容器，会在容器启动时为该容器提供所有运行中的服务的环境变流量。例如，如果名为`foo`的服务已经存在，所有的容器都会在它们的初始环境中获取以下变量：

```
FOO_SERVICE_HOST=<the host the Service is running on>
FOO_SERVICE_PORT=<the port the Service is running on>
```

如果你要在代码中访问Service，不要使用这些环境变量；使用服务的DNS名称（[DNS name of the Service](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)）来代替。服务的环境变量仅仅提供给那些陈旧的应用：这些应用不能修改来使用DNS查找，不能以灵活的方式访问服务。

* 除非绝对必要，否则不要为Pod定义`hostPort`属性。当你为Pod绑定了一个`hostPort`，就会限制该Pod可以被调度的节点的数量，因为每一个 &lt;`hostIP`,`hostPort`,`protocol`&gt; 组合都是唯一的。如果你不明确指定`hostIP`和`protocol`，Kubernetes 会使用`0.0.0.0`作为`hostIP`的默认值，使用`TCP`作为`protocol`的默认值。如果你为了调试需要访问pod，可以使用[apiserver proxy](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#manually-constructing-apiserver-proxy-urls)或者[`kubectl port-forward`](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)。如果你确实需要向外暴露某个节点上的Pod端口，在使用`hostPort`之前优先考虑使用 [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) Service。

* 避免使用`hostNetwork`，原因同`hostPort`。

* 如果你不需要`kube-proxy`的负载均衡功能，请使用[headless Services](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)\(which have a`ClusterIP`of`None`\) 进行服务发现。

## 使用标签 {#using-labels}

---

* 定义并使用标签来确定你的应用或者Deployment的语义属性，比如`{ app: myapp, tier: frontend, phase: test, deployment: v3 }`。你可以利用这些标签来为其他资源选择合适的Pods。查看[guestbook](https://github.com/kubernetes/examples/tree/master/guestbook/)来了解更多。

A Service can be made to span multiple Deployments by omitting release-specific labels from its selector.[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)make it easy to update a running service without downtime.

A desired state of an object is described by a Deployment, and if changes to that spec are_applied_, the deployment controller changes the actual state to the desired state at a controlled rate.

* You can manipulate labels for debugging. Because Kubernetes controllers \(such as ReplicaSet\) and Services match to Pods using selector labels, removing the relevant labels from a Pod will stop it from being considered by a controller or from being served traffic by a Service. If you remove the labels of an existing Pod, its controller will create a new Pod to take its place. This is a useful way to debug a previously “live” Pod in a “quarantine” environment. To interactively remove or add labels, use
  [`kubectl label`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#label)
  .

## 容器镜像 {#container-images}

---

* 容器默认的[imagePullPolicy](https://kubernetes.io/docs/concepts/containers/images/#updating-images)是`IfNotPresent`，当本地不存在时，[kubelet](https://kubernetes.io/docs/admin/kubelet/) 会进行拉取镜像操作。如果你想让Kubernetes每次启动容器的时候都去拉取镜像，请设置`imagePullPolicy: Always`。<br><br>另外一种可选但已被废弃的方式是：为镜像设置`:latest`tag，这样会隐式地将`imagePullPolicy`设置为`Always`。<br>
>**Note:**在生产环境中，你应该避免使用这种`:latest`tag 的方式，因此这样很难跟踪到底是哪个版本的镜像在运行，也很难回滚。

* 为确保容器总是使用同一版本的镜像，你可以定义镜像的[digest](https://docs.docker.com/engine/reference/commandline/pull/#pull-an-image-by-digest-immutable-identifier)\(比如  
  `sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2`\)。这样会唯一标识一个特定版本的镜像，它永远不会被Kubernetes更新，除非你改变了digest的值。

## 使用kubectl {#using-kubectl}

---

* 使用`kubectl apply -f <directory>`或者`kubectl create -f <directory>`这两个命令。它们会寻找`<directory>`目录下所有的`.yaml`、`.yml`和`.json`格式的Kubernetes配置文件，并将这些配置文件传递给`apply`或者`create`命令。

* 进行`get`和`delete`操作的时候，使用标签选择器，而不是特定的对象的名字。更多见[label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors)and[using labels effectively](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#using-labels-effectively)。

* 使用`kubectl run`和`kubectl expose`命令来快速创建一个单一容器的Deployments和Services。见[Use a Service to Access an Application in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/service-access-application-cluster/)。



