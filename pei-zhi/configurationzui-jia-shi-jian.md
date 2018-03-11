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

* 在创建与[Service](https://kubernetes.io/docs/concepts/services-networking/service/)相关的后台工作负载\(Deployments 或者 ReplicaSets\)之前，也在其他工作负载需要访问该服务之前，去创建服务。当Kubernetes启动一个容器，会在容器启动时为该容器提供所有运行中的服务的环境变流量。例如，如果名为`foo`的服务已经存在，所有的容器都会在它们的初始环境中获取以下变量：

```
FOO_SERVICE_HOST=<the host the Service is running on>
FOO_SERVICE_PORT=<the port the Service is running on>
```

       如果你

If you are writing code that talks to a Service, don’t use these environment variables; use the[DNS name of the Service](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)instead. Service environment variables are provided only for older software which can’t be modified to use DNS lookups, and are a much less flexible way of accessing Services.

* Don’t specify a`hostPort`for a Pod unless it is absolutely necessary. When you bind a Pod to a`hostPort`, it limits the number of places the Pod can be scheduled, because each &lt;`hostIP`,`hostPort`,`protocol`&gt; combination must be unique. If you don’t specify the`hostIP`and`protocol`explicitly, Kubernetes will use`0.0.0.0`as the default`hostIP`and`TCP`as the default`protocol`.

  If you only need access to the port for debugging purposes, you can use the[apiserver proxy](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#manually-constructing-apiserver-proxy-urls)or[`kubectl port-forward`](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

  If you explicitly need to expose a Pod’s port on the node, consider using a[NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)Service before resorting to`hostPort`.

* Avoid using`hostNetwork`, for the same reasons as`hostPort`.

* Use[headless Services](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)\(which have a`ClusterIP`of`None`\) for easy service discovery when you don’t need`kube-proxy`load balancing.

## Using Labels {#using-labels}

* Define and use
  [labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
  that identify
  **semantic attributes**
  of your application or Deployment, such as
  `{ app: myapp, tier: frontend, phase: test, deployment: v3 }`
  . You can use these labels to select the appropriate Pods for other resources; for example, a Service that selects all
  `tier: frontend`
  Pods, or all
  `phase: test`
  components of
  `app: myapp`
  . See the
  [guestbook](https://github.com/kubernetes/examples/tree/master/guestbook/)
  app for examples of this approach.

A Service can be made to span multiple Deployments by omitting release-specific labels from its selector.[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)make it easy to update a running service without downtime.

A desired state of an object is described by a Deployment, and if changes to that spec are_applied_, the deployment controller changes the actual state to the desired state at a controlled rate.

* You can manipulate labels for debugging. Because Kubernetes controllers \(such as ReplicaSet\) and Services match to Pods using selector labels, removing the relevant labels from a Pod will stop it from being considered by a controller or from being served traffic by a Service. If you remove the labels of an existing Pod, its controller will create a new Pod to take its place. This is a useful way to debug a previously “live” Pod in a “quarantine” environment. To interactively remove or add labels, use
  [`kubectl label`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#label)
  .

## Container Images {#container-images}

* The default[imagePullPolicy](https://kubernetes.io/docs/concepts/containers/images/#updating-images)for a container is`IfNotPresent`, which causes the[kubelet](https://kubernetes.io/docs/admin/kubelet/)to pull an image only if it does not already exist locally. If you want the image to be pulled every time Kubernetes starts the container, specify`imagePullPolicy: Always`.

  An alternative, but deprecated way to have Kubernetes always pull the image is to use the`:latest`tag, which will implicitly set the`imagePullPolicy`to`Always`.

  **Note:**You should avoid using the`:latest`tag when deploying containers in production, because this makes it hard to track which version of the image is running and hard to roll back.

* To make sure the container always uses the same version of the image, you can specify its
  [digest](https://docs.docker.com/engine/reference/commandline/pull/#pull-an-image-by-digest-immutable-identifier)
  \(for example
  `sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2`
  \). This uniquely identifies a specific version of the image, so it will never be updated by Kubernetes unless you change the digest value.

## Using kubectl {#using-kubectl}

* Use`kubectl apply -f <directory>`or`kubectl create -f <directory>`. This looks for Kubernetes configuration in all`.yaml`,`.yml`, and`.json`files in`<directory>`and passes it to`apply`or`create`.

* Use label selectors for`get`and`delete`operations instead of specific object names. See the sections on[label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors)and[using labels effectively](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#using-labels-effectively).

* Use`kubectl run`and`kubectl expose`to quickly create single-container Deployments and Services. See[Use a Service to Access an Application in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/service-access-application-cluster/)for an example.



