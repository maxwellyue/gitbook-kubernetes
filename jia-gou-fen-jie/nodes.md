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

第三种角色就是监控节点的健康。node controller负责在节点不可达（比如由于某种原因node controller无法接收到该节点的心跳，再比如节点宕机）时，将节点的status从ready更新为unknown，如果该节点持续不可达（默认的超时时间是40秒，超过该时间，就会报告unknown，并在5分钟后开始evict pod），则会将该节点上的pods删除（使用优雅的终止方式）。node controller会以`--node-monitor-period`秒的间隔检查每个节点的状态。

在Kubernetes 1.4版本中，我们对node controller的逻辑进行了升级，以便可以更好地处理当大量节点不可达master的情况（比如，master出现了网络问题）。从Kubernetes 1.4版本开始，node controller在做出evict pod的决定时会检查所有节点的状态。

在大多数情况下，node controller将eviction rate限制在每秒`--node-eviction-rate`\(默认 0.1\) ，这意味着，它不会以超过每10秒一个节点的速率从节点中evict pods 。

当一个给定的可用的zone变得不健康时，节点的eviction行为会发生改变。node controller会检查不健康（节点condition中ready的值为false或unknown）的nodes在该zone中所占的百分比。如果这个比例达到了`--unhealthy-zone-threshold`\(默认 0.55\)，那么eviction rate 就会减小；如果这个集群很小（比如小于或等于`--large-cluster-size-threshold`个节点 ，默认50\)，那么就会停止 evictions ,，否则eviction rate就会持续减小到 `--secondary-node-eviction-rate`\(默认0.01\) 每秒。可用zone之所以实现这些规则，是因为一个可用zone可能与master断开，但此时其他zones仍然保持连接。如果你的集群没有跨越多个云提供商可用性区域，那么就仅仅存在一个可用zone（整个集群）。

将你的节点分布到不同的zones中的一个主要原因是，你的工作负载可以在一个zone不可用时，将其迁移到另一个可用zone中。因此，如果一个zone中的所有节点都不健康（例如集群中不存在健康节点了），node controller 就会以`--node-eviction-rate`的正常速率进行evicts。在这种情况下，node controller 会认为master的连通性出现了问题，就会停止所有的evictions，直到恢复了一些连接。

从Kubernetes 1.6开始，node controller 也会对那些运行在出现`NoExecute`污点，但却不容忍任何污点的pods进行evicting。此外，作为alpha特性，该功能是默认关闭的，node controller还负责对出现问题（如不可达或not ready）的结点进行添加污点。更多该alpha特性的细节见[this documentation](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)。

从Kubernetes 1.8开始，node controller可以创建污点来表示结点的conditions。这是1.8版本的alpha特性。

### Self-Registration of Nodes {#self-registration-of-nodes}

当kubelet的`--register-node`标志为true \(默认值\)的时候，kubelet 会尝试将自身节点（运行该kubelet的节点）注册到API server上。这是大多数发行版使用的首选模式。

对于自注册功能，kubelet会附带以下可选参数启动：

* `--kubeconfig`
  * 向apiserver进行身份认证时的使用的凭证的路径。
* `--cloud-provider`
  * 读取自身元数据时如何与云提供商进行交互。
* `--register-node`
  * 自动向API server注册。
* `--register-with-taints`
  * 注册时携带的taints（逗号分隔的`<key>=<value>:<effect>`)。当`register-node`为false时，该参数无效。
* `--node-ip`
  * 节点的IP地址。
* `--node-labels`
  * 注册时携带的标签。
* `--node-status-update-frequency`
  * 定义kubelet向master发送节点状态的频率。 

目前，任意一个kubelet都被授权可以创建/修改任意一个节点资源。但在实践中，它应当仅仅创建/修改自身。(未来，我们计划仅仅允许kubelet去修改自身节点资源)

#### 手动管理节点

集群的管理员可以创建和修改节点对象。

如果管理员想手动创建节点对象，需要将kubelet设置为`--register-node=false`。

管理员可以修改节点资源（无论`--register-node`是true还是false）。修改操作包括为节点设置标签以及将其标记为不可调度。

节点上的标签可以与选择器一起来控制pods的调度，比如限制一个pod只可以在一个特定的节点集合中运行。

将节点标记为不可调度，则会组织新的pods调度到该节点，但不会影响已经在该节点运行的pods。这个功能可以用在节点重启的时候。比如，可以通过以下命令来禁止对该节点调度：

```
kubectl cordon $NODENAME
```
要注意的是，DaemonSet controller创建的pods会绕过调度器，所以会忽略节点的不可调度属性。The assumption is that daemons belong on the machine even if it is being drained of applications in preparation for a reboot.

### Node capacity

节点的容量（如CPU数量、内存大小等）是节点对象的一部分。正常情况下，节点在注册它们自己并创建节点对象的时候，会报告自身的容量信息。如果你是手动进行节点管理，那么你需要在添加节点的时候设置节点容量。

Kubernetes调度器会确保某节点上所有的pods可以拥有足够的资源。它会检查容器请求总和不会超过节点容量。它包括所有由kubelet启动的容器，但不会包括由Docker直接启动的容器或者非容器进程。

如果你想显式地为非pod进行保留一些资源，你可以使用如下模板
创建一个placeholder pod：

```
apiVersion: v1
kind: Pod
metadata:
  name: resource-reserver
spec:
  containers:
  - name: sleep-forever
    image: k8s.gcr.io/pause:0.8.0
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
```

设置你想保留的cpu和内存值。将该文件放置在清单目录（由kublet的`--config=DIR`参数指定）中。


API Object




