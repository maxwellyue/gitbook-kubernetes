# Node是什么?

---

一个node是指在Kubernetes中的一个工作机器（a worker machine），之前（较早版本中）被称为`minion`。节点可以是VM或者物理机，这取决于集群。每一个节点都有运行pod的必要能力，并由Master组件来管理。节点中的服务包括：Docker、kubelet 和kube-proxy. 更多细节见[The Kubernetes Node](https://git.k8s.io/community/contributors/design-proposals/architecture/architecture.md#the-kubernetes-node)。

# Node 的状态

---

节点的状态包含以下信息：

* Addresses
* ~~Phase~~    **deprecated**
* Condition
* Capacity
* Info

每一部分都将在下面的部分进行详细描述。

### Addresses {#addresses}

这些字段的用途很大程度上取决于你的云提供商或裸机配置。

* HostName: 主机名是由节点的kernel报告的。可以通过kubelet的
  `--hostname-override`参数进行重写。
* ExternalIP: 通常情况下，节点的外网IP地址是外部可路由的（即可以允许集群外的访问）。
* InternalIP: 通常情况下，节点的内网IP只能在集群内部进行路由。

### Phase {#phase}

Deprecated: node phase is no longer used.

### Condition {#condition}

`conditions`字段描述了所有运行中的结点的状态。

| 节点condition | 描述 |
| :--- | :--- |
| `OutOfDisk` | 如果没有足够的空间添加新的pods，则为true，否则为false。 |
| `Ready` | 如果节点时健康的，准备好接受pods，则为true。如果节点不健康，且没有准备好接受pods，则为false。如果节点controller超过40s没有收到该节点的消息，则为Unknown。 |
| `MemoryPressure` | 如果节点中有内存压力（节点内存降的很低），则返回true，否则返回false。 |
| `DiskPressure` | 如果节点中有磁盘压力（节点的磁盘空间很低），则返回true，否则返回false。 |
| `ConfigOK` | 如果kubelet 被正确配置，则返回true，否则返回false。 |

节点的condition会用JSON对象来表示，比如下面的响应内容描述了一个健康的结点：

```
"conditions": [
  {
    "type": "Ready",
    "status": "True"
  }
]
```

如果Ready的status是“Unknown” 或者“False”，且持续时间比pod-eviction-timeout还长，则[kube-controller-manager](https://kubernetes.io/docs/admin/kube-controller-manager/)会接收到一个参数，所有运行在该节点上的pods会被Node Contoller通过调度来删除。默认的eviction-timeout是5分钟。当节点不可达时，apiserver就不能与该节点上的kublet进行交互。删除pods的指令就不能传送到kublet，直到apiserver与该节点重新建立连接。在这段时间内，计划被删除的pods可能会在这个与集群断开的节点上继续运行。

在Kubernetes1.5之前的版本，node controller 会从apiserver中强制删除这些不可达的pods。但是，1.5及更高的版本中，node controller不会强制删除这些pods，直到它确认这些pods已经从集群中真正地停止运行。可以将这些在可能在不可达节点上运行的pods的状态看成是“Terminating” 或者 “Unknown”。当Kubernetes无法推断一个结点是否从集群中永久离开时，集群管理员可能需要手动删除该节点对象。从kubernetes删除节点对象会导致运行在该节点的Pod对象从apiserver中删除，并释放它们的名字。

1.8版本引入了一个alpha的特性：自动创建污点（[taints](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)）来表示conditions。可以通过给apiserver、controller manager和scheduler设置如下标识来开启该功能：  
`--feature-gates=...,TaintNodesByCondition=true`

当`TaintNodesByCondition`设为true，scheduler就会忽略节点的 conditions，取而代之的是，它会查看节点的污点以及pods的tolerations。

现在，用户可以在这两种新旧调度模型之间进行选择。没有任何tolerations的Pod将会按照旧的模式进行调度，而可以容忍特定节点污点的pod则可以调度在该节点上。

要注意，由于观察conditions和创建taint之间通常会有少于1秒的小小的延迟，开启该功能可能会轻微地增加一些成功调度但被 kubelet拒绝的pods。

### Capacity

描述节点上的可用资源：CPU，内存，该节点最大可调度的Pods数量。

### Info

节点的一般信息，如kernel版本，Kubernetes 版本 \(kubelet 和 kube-proxy 版本\)，Docker 版本 \(如果用到了Docker\)，OS 名字。这些信息由该节点上的Kubelet进行收集。

# Management

---

与[pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)和[services](https://kubernetes.io/docs/concepts/services-networking/service/)不同的是 ，节点在本质上并不是由Kubernetes创建的：它是由像Google Compute Engine这样的云提供商或你的物理机或VM在外部创建的。这意味着，当Kubernetes创建一个节点，它实际上只是创建了一个表示该节点的对象。在创建之后，Kubernetes将会检查该节点是否有效。比如，假如你通过如下内容来创建一个节点：

```
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

Kubernetes将会在内部创建一个节点对象， 并基于`metadata.name`字段（假设`metadata.name` 可以被解析）对该节点进行健康检查。假如该节点是有效的，例如所有必须的服务都在运行，则它就可以去运行pod；否则，该节点将会被任何集群活动所忽略，直到它变为有效的。要注意的是，Kubernetes会为该无效的节点一直维护一个结点对象，除非客户端显式地将其删除，在此期间，Kubernetes将会一直检查该节点是否是有效的。

目前，Kubernetes中有3个组件会与节点交互： node controller，kubelet以及kubectl。

### Node Controller {#node-controller}

node controller是Kubernetes的master组件，它会从不同维度对节点进行管理。

node controller在节点生命中扮演着不同的角色。首先，当节点注册时，会分配一个CIDR block到节点（假如允许CIDR block分配）。

其次是保持node controller内部的节点列表与云提供商提供的可用机器列表保持一致。当运行在云环境中时，一旦节点不健康，node controller将会向云提供商询问该节点的VM是否可用。如果不可用，则会将该节点从自身维护的节点列表中删除。

第三种角色就是监控节点的健康。node controller负责在节点不可达（比如由于某种原因node controller无法接收到该节点的心跳，再比如节点宕机）时，将节点的status从ready更新为unknown，如果该节点持续不可达（默认的超时时间是40秒，超过该时间，就会报告unknown，并在5分钟后开始删除pod），则会将该节点上的pods删除（使用优雅的终止方式）。node controller会以`--node-monitor-period`秒的间隔检查每个节点的状态。

在Kubernetes 1.4版本中，我们



, we updated the logic of the node controller to better handle cases when a large number of nodes have problems with reaching the master \(e.g. because the master has networking problem\). Starting with 1.4, the node controller will look at the state of all nodes in the cluster when making a decision about pod eviction.

In most cases, node controller limits the eviction rate to`--node-eviction-rate`\(default 0.1\) per second, meaning it won’t evict pods from more than 1 node per 10 seconds.

The node eviction behavior changes when a node in a given availability zone becomes unhealthy. The node controller checks what percentage of nodes in the zone are unhealthy \(NodeReady condition is ConditionUnknown or ConditionFalse\) at the same time. If the fraction of unhealthy nodes is at least`--unhealthy-zone-threshold`\(default 0.55\) then the eviction rate is reduced: if the cluster is small \(i.e. has less than or equal to`--large-cluster-size-threshold`nodes - default 50\) then evictions are stopped, otherwise the eviction rate is reduced to`--secondary-node-eviction-rate`\(default 0.01\) per second. The reason these policies are implemented per availability zone is because one availability zone might become partitioned from the master while the others remain connected. If your cluster does not span multiple cloud provider availability zones, then there is only one availability zone \(the whole cluster\).

A key reason for spreading your nodes across availability zones is so that the workload can be shifted to healthy zones when one entire zone goes down. Therefore, if all nodes in a zone are unhealthy then node controller evicts at the normal rate`--node-eviction-rate`. The corner case is when all zones are completely unhealthy \(i.e. there are no healthy nodes in the cluster\). In such case, the node controller assumes that there’s some problem with master connectivity and stops all evictions until some connectivity is restored.

Starting in Kubernetes 1.6, the NodeController is also responsible for evicting pods that are running on nodes with`NoExecute`taints, when the pods do not tolerate the taints. Additionally, as an alpha feature that is disabled by default, the NodeController is responsible for adding taints corresponding to node problems like node unreachable or not ready. See[this documentation](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)for details about`NoExecute`taints and the alpha feature.

Starting in version 1.8, the node controller can be made responsible for creating taints that represent Node conditions. This is an alpha feature of version 1.8.

### Self-Registration of Nodes {#self-registration-of-nodes}

When the kubelet flag`--register-node`is true \(the default\), the kubelet will attempt to register itself with the API server. This is the preferred pattern, used by most distros.

For self-registration, the kubelet is started with the following options:

* `--kubeconfig`
  * Path to credentials to authenticate itself to the apiserver.
* `--cloud-provider`
  * How to talk to a cloud provider to read metadata about itself.
* `--register-node`
  * Automatically register with the API server.
* `--register-with-taints`
  * Register the node with the given list of taints \(comma separated
    `<`
    `key`
    `>`
    `=`
    `<`
    `value`
    `>`
    `:`
    `<`
    `effect`
    `>`
    \). No-op if
    `register-node`
    is false.
* `--node-ip`
  * IP address of the node.
* `--node-labels`
  * Labels to add when registering the node in the cluster.
* `--node-status-update-frequency`
  * Specifies how often kubelet posts node status to master.

Currently, any kubelet is authorized to create/modify any node resource, but in practice it only creates/modifies its own. \(In the future, we plan to only allow a kubelet to modify its own node resource.\)

#### Manual Node Administration {#manual-node-administration}

A cluster administrator can create and modify node objects.

If the administrator wishes to create node objects manually, set the kubelet flag`--register-node=false`.

The administrator can modify node resources \(regardless of the setting of`--register-node`\). Modifications include setting labels on the node and marking it unschedulable.

Labels on nodes can be used in conjunction with node selectors on pods to control scheduling, e.g. to constrain a pod to only be eligible to run on a subset of the nodes.

Marking a node as unschedulable will prevent new pods from being scheduled to that node, but will not affect any existing pods on the node. This is useful as a preparatory step before a node reboot, etc. For example, to mark a node unschedulable, run this command:

```
kubectl cordon 
$NODENAME
```

Note that pods which are created by a DaemonSet controller bypass the Kubernetes scheduler, and do not respect the unschedulable attribute on a node. The assumption is that daemons belong on the machine even if it is being drained of applications in preparation for a reboot.

### Node capacity {#node-capacity}

The capacity of the node \(number of cpus and amount of memory\) is part of the node object. Normally, nodes register themselves and report their capacity when creating the node object. If you are doing[manual node administration](https://kubernetes.io/docs/concepts/architecture/nodes/#manual-node-administration), then you need to set node capacity when adding a node.

The Kubernetes scheduler ensures that there are enough resources for all the pods on a node. It checks that the sum of the requests of containers on the node is no greater than the node capacity. It includes all containers started by the kubelet, but not containers started directly by Docker nor processes not in containers.

If you want to explicitly reserve resources for non-pod processes, you can create a placeholder pod. Use the following template:

