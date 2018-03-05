# 概念

这一章节的内容可以帮助学习Kubernetes的组成部分以及Kubernetes中代表集群的所使用的概念，这些都可以帮助你对Kubernetes的工作原理有更深入的理解。

---

## 简述

为了让集群工作在kubernetes中，你应该使用Kubernetes API对象来描述你所期望的集群状态：你想要运行什么应用、这些应用使用什么容器镜像、集群的副本数量、你想使用什么网络和存储等等。你可以通过Kubernetes API创建对象来设定你想要的状态，通常是通过命令行的交互方式，即kubectl。当然，你也可以直接与Kubernetes API交互，设定或修改你想要的集群状态。

一旦你设定了你想要的状态，Kubernetes控制面板就会开始工作，来让集群的当前状态与所期望的状态保持一致。为了达到这种效果，Kubernetes会自动执行一系列的任务，比如启动或重启容器、为指定的应用扩容等等。Kubernetes控制面板是由运行在集群中的很多进程组成的。

* Kubernetes Master是指运行在集群中单一节点中的三个进程的集合，该节点被称为Master节点，这三个进程分别是：kube-apiserver，kube-controller，kube-scheduler。

* 集群中其他非Master节点都会运行下列两个进程：

                  kubelet：该进程会和Kubernetes 的Master结点进行交互。

                  kube-proxy：该进程是Kubernetes网络服务的网络代理。

---

## Kubernetes 对象

Kubernetes包含了一些代表你的整个系统的抽象概念：部署容器化的应用，关联的网络和存储资源以及其他一些描述集群工作性质的信息。这些抽象概念可以用Kubernetes API中的对象来表示（更多详细内容请参考[https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/ "Understanding Kubernetes Objects")），Kubernetes中的基础对象包括：



















