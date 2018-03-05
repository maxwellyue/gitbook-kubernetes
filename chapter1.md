# 概念

这一章节的内容可以帮助学习Kubernetes的组成部分以及Kubernetes中代表集群的所使用的概念，这些都可以帮助你对Kubernetes的工作原理有更深入的理解。

---

## 简述

为了让集群工作在kubernetes中，你应该使用Kubernetes API对象来描述你所期望的集群状态：你想要运行什么应用、这些应用使用什么容器镜像、集群的副本数量、你想使用什么网络和存储等等。你可以通过Kubernetes API创建对象来设定你想要的状态，通常是通过命令行的交互方式，即kubectl。当然，你也可以直接与Kubernetes API交互，设定或修改你想要的集群状态。

一旦你设定了你想要的状态，Kubernetes控制面板就会开始工作，来让集群的当前状态与所期望的状态保持一致。为了达到这种效果，Kubernetes会自动执行一系列的任务，比如启动或重启容器、为指定的应用扩容等等。Kubernetes控制面板是由运行在集群中的很多进程组成的。

* Kubernetes Master是指运行在集群中单一节点中的三个进程的集合，该节点被称为Master节点，这三个进程分别是：kube-apiserver，kube-controller，kube-scheduler。

* 集群中其他非Master节点都会运行下列两个进程：

  ```
              kubelet：该进程会和Kubernetes 的Master结点进行交互。

              kube-proxy：该进程是Kubernetes网络服务的网络代理。
  ```

---

## Kubernetes 对象

Kubernetes包含了一些代表你的整个系统的抽象概念：部署容器化的应用，关联的网络和存储资源以及其他一些描述集群工作性质的信息。这些抽象概念可以用Kubernetes API中的对象来表示（更多详细内容请参考[https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/ "Understanding Kubernetes Objects")）。

Kubernetes中的基础对象包括：

* Pod
* Service
* Volume
* Namespace

另外，Kubernetes还包含了更高层级的抽象概念，被称为控制器（Controllers）。Controllers是基于基础对象建立的，这些Controllers提供了额外的功能，使得操作更加便利，主要包括：

* ReplicaSet
* Deployment
* StatefulSet
* DaemonSet
* Job

---

## Kubernetes 控制面板 {#kubernetes-control-plane}

Kubernetes的控制面板是由一系列组件构成的，比如Kubernetes Master和kubelet，它负责管理Kubernetes与你的集群的交互。Kubernetes控制面板维护系统中所有的Kubernetes对象，并通过不断轮询来负责维护这些对象的状态。在任意时间点，控制面板会对集群中的任何改变做出响应，来让系统中的Kubernetes对象的状态与你所期望的状态保持一致。

举个例子，当你使用Kubernetes API 创建了一个Deployment对象后，实际上你就为系统提供了一种新的期望状态。Kubernetes控制面板就会记录这些对象创建的指令，遵照你的描述来启动你要求的应用，并将它们调度到集群节点中，通过这种方式，你的集群实际状态就和你期望的状态达成了一致。

### Kubernetes Master

Kubernetes负责维护你期望的集群状态。当你和Kubernetes进行交互（通常是通过kubectl命令行），实际上你是在和Kubernetes的Master进行交互。

这里的Master意指管理集群状态的进程集合。通常情况下，这些进程运行在集群中的单一节点中，该节点被称为Master节点或Master。当然，Master可以通过复制来达到高可用和冗余（即Master的高可用）。

### Kubernetes Nodes

集群中的Nodes是指运行application或cloud workflows的机器（如VM，物理机等等）。kubernetes Master控制每一个节点，通常会很少直接与Nodes进行交互。









