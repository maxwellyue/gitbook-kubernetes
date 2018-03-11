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

这就保证了，当开启`TaintBasedEvictions`功能时，如果有节点问题，如网络分区，它们不会被evicted。 \(when the`TaintBasedEvictions`feature is not enabled, they are also not evicted in these scenarios, but due to hard-coded behavior of the NodeController rather than due to tolerations\).

它们也容忍以下两种`NoSchedule`污点：

* `node.kubernetes.io/memory-pressure`
* `node.kubernetes.io/disk-pressure`

When the support to critical pods is enabled and the pods in a DaemonSet are labeled as critical, the Daemon pods are created with an additional`NoSchedule`toleration for the`node.kubernetes.io/out-of-disk`taint.

Note that all above`NoSchedule`taints above are created only in version 1.8 or later if the alpha feature`TaintNodesByCondition`is enabled.

Also note that the`node-role.kubernetes.io/masterNoSchedule`toleration specified in the above example is needed on 1.6 or later to schedule on\_master\_nodes as this is not a default toleration.

