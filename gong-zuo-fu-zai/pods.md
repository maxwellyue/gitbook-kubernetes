英文地址：[https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)

## 理解Pods {#understanding-pods}

---

Pod是Kubernetes的\]基础构件，是你在Kubernetes对象模型中所能创建或部署的最小、最简单的单元。一个Pod表示集群中运行的一个程序。

Pod中包含了应用容器的封装（或者，在某些情况下，多种容器）、存储资源、唯一的网络IP和控制容器运行的选项。一个Pod表示了一个部署单元：Kubernetes中单一的应用实例，其中可能包含一个或多个仅仅耦合并共享资源的容器。

> [Docker](https://www.docker.com/) 是Kubernetes Pod中最常见的容器运行时，但是Pods也支持其他容器运行时。

kubernetes集群中，Pods主要用在两个方面：

* 运行单一容器

  “一容器一Pod”的模型在Kubernetes中是最常见的。此时，你可以将Pod看成是单一容器的包装，Kubernetes对Pods进行管理，而不会直接管理容器。

* 运行需要工作在一起的多个容器

一个Pod中也可以包含一个由多个需要紧密耦合、共享资源的co-located的容器组成。这些co-located的容器可能会形成一个统一的服务单元--一个容器负责从共享卷中向外提供文件服务，而另一个独立的“sidecar”容器负责刷新或更新这些文件。Pod会将这些容器和存储资源封装成为一个单一的可管理实体。

[Kubernetes Blog](http://blog.kubernetes.io/) 有更多关于Pod使用的信息，见：

* [The Distributed System Toolkit: Patterns for Composite Containers](http://blog.kubernetes.io/2015/06/the-distributed-system-toolkit-patterns.html)
* [Container Design Patterns](http://blog.kubernetes.io/2016/06/container-design-patterns.html)

每一个Pod意味着运行一个给定的应用的实例。如果你想水平伸缩你的应用（比如，运行多个实例），你应该使用多个Pod，每一个实例对应一个Pod。在Kubernetes中，这些实例被简称为副本（_replication_）。这些被复制的Pods通常被一个叫做Controller的抽象概念进行当做一个分组来创建和管理。更多信息见[Pods and Controllers](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pods-and-controllers)。

### Pods如何管理多个容器 {#how-pods-manage-multiple-containers}

Pods被设计成支持多个共生的进程（即容器），并将这些进程当做一个服务单元。Pods中的容器会一起被自动调度到集群中的同一物理机或虚拟机上。这些容器会共享资源和依赖，相互通信，并在何时和如何终止这一点上进行协调。

注意，在一个Pod中运行多个共生的容器是相对高级的用例。你仅仅需要在你的容器是仅仅耦合的情况下才需要使用这种方式。比如，你可能有一个容器来充当web文件服务器，而另一个“sidecar”容器负责更新这些文件，如下图所示：

插图：

#### 网络

Pods为这些容器提供两种共享的资源：网络和存储。这些同一Pod中的容器之间可以使用`localhost`进行这种方式进行通信。当这些容器需要与外部的其他Pods进行通信时，它们必须在如何使用网络资源上进行协调。

#### 存储 {#storage}

一个Pod可以定义一个共享存储卷集。该Pod中的所有容器都可以访问这些共享卷，这样容器之间就可以共享数据。Volumes也可以用来持久化Pod中的数据，以便其中的容器重启后可继续使用。Kubernetes如何在Pod中实现共享存储的内容见[Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)。

## Working with Pods {#working-with-pods}

---

在Kubernetes中，很少去直接创建独立的Pods，即使是一个Pods，这是因为Pods被设计为相对短暂的、一次性的实体。当一个Pod被创建（你可以创建，或者Controller间接创建），它会被调度到集群中的一个节点上去运行。Pod会一直运行在该节点，除非出现以下几种情形之一：进程终止、Pod对象被删除、Pod因缺少资源被evicte、节点失败。

注意：不要将Pod中容器的重启与Pod的重启混淆。Pod本身不会运行，它只是容器运行的环境，并一直存在，直到被删除。

Pods没有自愈的能力。如果一个Pod被调度到一个失败的节点上，或者调度操作本身失败了，该Pod就会删除；同样地，Pods不会在由于资源缺少或节点维护时发生eviction时存活。Kubernetes使用更高级别的抽象，叫做Controller，来管理这些相对短暂的Pod实例。因此，即使可以直接创建Pod，但Kubernetes中，更为常见的是通过Controller来管理你的Pods。关于Kubernetes如何使用Controllers实现Pod的伸缩和复原的内容见[Pods and Controllers](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pods-and-controllers)。

### Pods and Controllers {#pods-and-controllers}

Controller可以帮你来创建和管理多个Pods、处理副本、回滚、提供自愈能力。比如，如果一个节点失败，Controller会自动调度一个相同的Pod到另一个节点上来完成Pod的替代。

包含一个或多个Pods的Controllers包括但不限于：

* [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
* [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

通常来说，Controllers可以使用你提供的Pod模板来创建它负责的Pods。

## Pod 模板 {#pod-templates}

---

Pod模板是包含在其他对象（如[Replication Controllers](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)，[Jobs](https://kubernetes.io/docs/concepts/jobs/run-to-completion-finite-workloads/)和[DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)）中的Pod定义。Controllers使用Pod模板来制作实际的Pods。下面的示例是包含一个会打印信息的容器的Pod的清单。

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```

Pod模板相当于一个饼干模具，而不是去定义所有副本的期望状态。一旦饼干从模具中取出，这个饼干就与模具没有任何关系了。这里没有“量子纠缠”。该模板后续的修改或者切换到一个新的模板并不会对已经创建的Pods产生影响。同样地，被replication controller创建的Pods可以直接进行后续的更新操作。这与Pods形成鲜明对比，Pod指明了属于该Pod的所有容器的当前期望状态。这种方法从根本上简化了系统语义并增加了原语的灵活性。



  






