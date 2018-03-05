英文原文：[https://kubernetes.io/docs/concepts/overview/components/](https://kubernetes.io/docs/concepts/overview/components/)

# Master组件

---

Master组件是集群的控制面板（control plane）。Master组件为集群做出全局指令（如调度），并检测和响应集群事件（如当replication控制器发现replicas数量不满足时，就会启动一个新的pod）。

Master组件可以运行在集群中的任意一个节点上。但是，简单起见，启动脚本通常会在同一个机器上启动所有master组件，并且不在该机器上运行用户的容器。可以参考[Building High-Availability Clusters](https://kubernetes.io/docs/admin/high-availability/)来看一个multi-master-VM的设置方式。

### kube-apiserver

kube-apiserver是Master中暴露Kubernetes API的组件，相当于Kubernetes控制面板的前端。

它被设计成可水平伸缩的---通过部署多个实例。参考[Building High-Availability Clusters](https://kubernetes.io/docs/admin/high-availability/)。

### etcd

持久化、高可用的键值对存储系统，用来存储所有节点数据。

时刻为你的Kubernetes集群的etcd数据做备份！关于etcd的更深入的内容，见[etcd documentation](https://github.com/coreos/etcd/blob/master/Documentation/docs.md)。

### kube-scheduler

负责监控新创建还没有分配到节点的pod，并选择一个结点来运行该pod。

调度策略包括资源需求、硬件/软件/规则限定、affinity and anti-affinity specifications, data locality, inter-workload interference and deadlines。

### kube-controller-manager {#kube-controller-manager}

负责运行控制器（controllers）。

从逻辑上来讲，每一个controller是一独立的进程，但为了减少复杂性，它们都被编译在同一个二进制包并运行在同一个进程中。

这些controllers包括：

* Node Controller：负责监听和响应节点宕机。
* Replication Controller：负责为系统中的每一个replication controller objedct保持正确的pod数量。
* Endpoints Controller：填充Endpoints对象，如joins Services & Pods。
* Service Account & Token Controllers：为新的命名空间创建默认的账户和API接入tokens。

### cloud-controller-manager {#cloud-controller-manager}

[cloud-controller-manager](https://kubernetes.io/docs/tasks/administer-cluster/running-cloud-controller/)runs controllers that interact with the underlying cloud providers. The cloud-controller-manager binary is an alpha feature introduced in Kubernetes release 1.6.

cloud-controller-manager runs cloud-provider-specific controller loops only. You must disable these controller loops in the kube-controller-manager. You can disable the controller loops by setting the`--cloud-provider`flag to`external`when starting the kube-controller-manager.

cloud-controller-manager allows cloud vendors code and the Kubernetes core to evolve independent of each other. In prior releases, the core Kubernetes code was dependent upon cloud-provider-specific code for functionality. In future releases, code specific to cloud vendors should be maintained by the cloud vendor themselves, and linked to cloud-controller-manager while running Kubernetes.

The following controllers have cloud provider dependencies:

* Node Controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
* Route Controller: For setting up routes in the underlying cloud infrastructure
* Service Controller: For creating, updating and deleting cloud provider load balancers
* Volume Controller: For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes

  


# Node组件

---

Node组件运行在每一个节点上，维护运行的pods并提供Kubernetes运行时环境。

### kubelet

运行在集群中每个节点上的agent，它负责保证容器在pod中的运行。

kubelet维护了多种机制提供的PodSpecs，它会保证PodSpecs中描述的容器正常健康地运行。kubelet不会管理那些不是由Kubernetes创建的容器。

### kube-proxy

kube-proxy是Kubernetes中服务这个抽象概念的保证，它负责维护主机上的网络规则并执行连接转发（负责为Service提供cluster内部的服务发现和负载均衡）。

### Container Runtime {#container-runtime}

容器运行时是一种负责运行容器的软件，Kubernetes目前支持两种运行时：[Docker](http://www.docker.com/)和[rkt](https://coreos.com/rkt/)。

## 插件（Addons） {#addons}

---











  
  


# 



