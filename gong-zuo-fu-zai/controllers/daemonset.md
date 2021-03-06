英文地址：[https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

## DaemonSet是什么？ {#what-is-a-daemonset}

---

_DaemonSet_用来确保所有（或部分）节点都会运行一个Pod的拷贝。每当有新节点添加到集群中，那么Pods也会添加到其中。当节点从集群中删除，这些Pods就会被回收。删除一个DaemonSet将会清除它创建的Pods。

一些DaemonSet典型的使用场景为：

* 在每个节点运行集群存储守护进程，比如`glusterd`，`ceph`。

* 在每个节点上运行日志收集守护进程，比如`fluentd`或者`logstash`。

* 在每个节点运行监控守护进程，比如[Prometheus Node Exporter](https://github.com/prometheus/node_exporter)，`collectd`，Datadog agent，New Relic agent，或者 Ganglia`gmond`。

在一个简单的情况下，一个DaemonSet，覆盖所有节点，会包含所有类型的守护进程。更复杂的设置的话，可以为每一种类型的守护进程设置多个DaemonSets，这些DaemonSets针对不同的硬件类型具有不同的标志和/或不同的内存和CPU请求。

## 编写DaemonSet Spec

---

#### 创建DaemonSet

你可以在YAML文件中描述DaemonSet。例如，下面的`daemonset.yaml`文件定义了一个运行fluentd-elasticsearch Docker镜像的DaemonSet。

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: gcr.io/google-containers/fluentd-elasticsearch:1.20
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

你可以基于该文件使用如下命令来创建一个DaemonSet：

```
kubectl create -f daemonset.yaml
```

#### 必需的属性

同其他Kubernetes配置一样，DaemonSet 需要`kind`和`metadata`属性。更多配置见[deploying applications](https://kubernetes.io/docs/user-guide/deploying-applications/),[configuring containers](https://kubernetes.io/docs/tasks/)和[object management using kubectl](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/overview/)文档。

DaemonSet同样需要[`.spec`](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status)部分。

#### Pod 模板

在`.spec`中，`.spec.template`是必须的。

`.spec.template`是Pod的模板。它与Pod定义完全一致，但不需要 `apiVersion`或者`kind`。

除了普通Pod中必须的属性外，DaemonSet中的Pod模板必须定义恰当的标签\(见[pod selector](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#pod-selector)\)。

除此之外，DaemonSet中的Pod模板的[`RestartPolicy`](https://kubernetes.io/docs/user-guide/pod-states)属性必须设置为`Always`，或者不去定义，它会默认为`Always`。

### Pod 选择器 {#pod-selector}

`.spec.selector`属性是一个pod选择器。它的工作方式与[Job](https://kubernetes.io/docs/concepts/jobs/run-to-completion-finite-workloads/)中`.spec.selector`一样。

As of Kubernetes 1.8, you must specify a pod selector that matches the labels of the`.spec.template`. The pod selector will no longer be defaulted when left empty. Selector defaulting was not compatible with`kubectl apply`. Also, once a DaemonSet is created, its`spec.selector`can not be mutated. Mutating the pod selector can lead to the unintentional orphaning of Pods, and it was found to be confusing to users.

The`spec.selector`is an object consisting of two fields:

* `matchLabels`
  * works the same as the`.spec.selector`of a [ReplicationController](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)。
* `matchExpressions`
  * allows to build more sophisticated selectors by specifying key, list of values and an operator that relates the key and values.

When the two are specified the result is ANDed.

If the`.spec.selector`is specified, it must match the`.spec.template.metadata.labels`. Config with these not matching will be rejected by the API.

Also you should not normally create any Pods whose labels match this selector, either directly, via another DaemonSet, or via other controller such as ReplicaSet. Otherwise, the DaemonSet controller will think that those Pods were created by it. Kubernetes will not stop you from doing this. One case where you might want to do this is manually create a Pod with a different value on a node for testing.

#### 仅在部分节点中运行Pods

如果你定义了`.spec.template.spec.nodeSelector`属性，那么DaemonSet controller 将只会在相匹配的节点上运行这些Pods。. 同样地，如果你定义了`.spec.template.spec.affinity`，那么DaemonSet controller 也只会在那些与定义的节点affinity相匹配的节点上创建Pods。如果这两个属性你都没有定义，那么DaemonSet controller 将会在所有节点上创建对应的Pods。

## Daemon Pods是如何被调度的 {#how-daemon-pods-are-scheduled}

---

正常情况下，Pod运行在哪个节点是由Kubernetes 的调度器进行选择的。但是，由DaemonSet controller创建的Pods已经指定了机器\(`.spec.nodeName`会在被创建的时候被指定，因此它会被调度器忽略\)。因此：

* DaemonSet controller不会理睬节点的[`unschedulable`](https://kubernetes.io/docs/admin/node/#manual-node-administration)属性 。
* DaemonSet controller可以在调度器还没有启动的时候就创建Pods，这一点可以help cluster bootstrap。

Daemon Pods确实会受 [taints and tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration)影响，但它们对下面两种污点的容忍会设置为`NoExecute`，并且没有 `tolerationSeconds`

* `node.kubernetes.io/not-ready`

* `node.alpha.kubernetes.io/unreachable`

这就保证了，当开启`TaintBasedEvictions`功能时，如果有节点问题，如网络分区，它们不会被evicted。 \(如果`TaintBasedEvictions`功能没有开启，它们也不会在这些场景下被evicted，但这是由NodeController的硬编码行为导致的，而不是容忍度\)。

它们也容忍以下两种`NoSchedule`污点：

* `node.kubernetes.io/memory-pressure`
* `node.kubernetes.io/disk-pressure`

当开启“关键Pod支持”功能，并且DaemonSet中的Pod被标记为“关键”，这些守护pods在创建时会添加对

`node.kubernetes.io/out-of-disk`污点的`NoSchedule`容忍度。

注意，上面所有的`NoSchedule` 相关污点只会在1.8户或之后的版本中提供，并且还要开启`TaintNodesByCondition`这个功能。

同时也要注意，`node-role.kubernetes.io/masterNoSchedule`容忍度定义定义需要在1.6及更高版本中。

## 与Daemon Pods进行通信 {#communicating-with-daemon-pods}

---

与DaemonSet中的Pods进行通信的方式有以下几种：

* **Push**
  DaemonSet中的Pods被配置成发送数据到另一个服务，比如统计服务器。此时，这些Pods没有客户端。
* **NodeIP and Known Port**
  DaemonSet中的Pods可以使用`hostPort`，这些就可以通过节点IP访问这些Pods。客户端通过某种方式获取所有IP，并按照事前约定获取端口。
* **DNS**
  以相同的Pod选择器创建一个[headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)，然后就可以使用`endpoints`资源发现DaemonSet或者从DNS中获取多个Pods的记录。
* **Service**
  以相同的Pod选择器创建一个服务，并且使用服务来随机访问其中一个节点中的Pods \(没办法访问指定的节点\)。

## Updating a DaemonSet {#updating-a-daemonset}

---

If node labels are changed, the DaemonSet will promptly add Pods to newly matching nodes and delete Pods from newly not-matching nodes.

You can modify the Pods that a DaemonSet creates. However, Pods do not allow all fields to be updated. Also, the DaemonSet controller will use the original template the next time a node \(even with the same name\) is created.

You can delete a DaemonSet. If you specify`--cascade=false`with`kubectl`, then the Pods will be left on the nodes. You can then create a new DaemonSet with a different template. The new DaemonSet with the different template will recognize all the existing Pods as having matching labels. It will not modify or delete them despite a mismatch in the Pod template. You will need to force new Pod creation by deleting the Pod or deleting the node.

In Kubernetes version 1.6 and later, you can[perform a rolling update](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)on a DaemonSet.

## Alternatives to DaemonSet {#alternatives-to-daemonset}

---

### Init Scripts {#init-scripts}

It is certainly possible to run daemon processes by directly starting them on a node \(e.g. using`init`,`upstartd`, or`systemd`\). This is perfectly fine. However, there are several advantages to running such processes via a DaemonSet:

* Ability to monitor and manage logs for daemons in the same way as applications.
* Same config language and tools \(e.g. Pod templates,
  `kubectl`
  \) for daemons and applications.
* Running daemons in containers with resource limits increases isolation between daemons from app containers. However, this can also be accomplished by running the daemons in a container but not in a Pod \(e.g. start directly via Docker\).

### Bare Pods {#bare-pods}

It is possible to create Pods directly which specify a particular node to run on. However, a DaemonSet replaces Pods that are deleted or terminated for any reason, such as in the case of node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, you should use a DaemonSet rather than creating individual Pods.

### Static Pods {#static-pods}

It is possible to create Pods by writing a file to a certain directory watched by Kubelet. These are called[static pods](https://kubernetes.io/docs/concepts/cluster-administration/static-pod/). Unlike DaemonSet, static Pods cannot be managed with kubectl or other Kubernetes API clients. Static Pods do not depend on the apiserver, making them useful in cluster bootstrapping cases. Also, static Pods may be deprecated in the future.

### Deployments {#deployments}

DaemonSets are similar to[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)in that they both create Pods, and those Pods have processes which are not expected to terminate \(e.g. web servers, storage servers\).

Use a Deployment for stateless services, like frontends, where scaling up and down the number of replicas and rolling out updates are more important than controlling exactly which host the Pod runs on. Use a DaemonSet when it is important that a copy of a Pod always run on all or certain hosts, and when it needs to start before other Pods.



