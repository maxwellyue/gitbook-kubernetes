英文地址：[https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

# ReplicaSet

---

ReplicaSet是下一代的Replication Controller。目前，ReplicaSet和Replication Controller的区别只是选择器的支持。ReplicaSet 支持set-based条件的选择器，而Replication Controller仅仅支持equality-based条件的选择器。

## 如何使用ReplicaSet

---

大多数支持Replication Controllers的[`kubectl`](https://kubernetes.io/docs/user-guide/kubectl/)commands 指令也同时支持ReplicaSets。其中的一个例外就是[`rolling-update`](https://kubernetes.io/docs/user-guide/kubectl/v1.9/#rolling-update)指令。如果你想使用滚动升级功能，请使用Deployments来代替。同样地，[`rolling-update`](https://kubernetes.io/docs/user-guide/kubectl/v1.9/#rolling-update)command指令是imperative的，而Deployments是declarative的，所以我们推荐通过[`rollout`](https://kubernetes.io/docs/user-guide/kubectl/v1.9/#rollout)指令使用Deployments。

尽管ReplicaSets可以单独使用，但现在它主要被[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)当做一种pod创建、删除和更新的编排机制。当使用Deployments的时候，你不必担心如何管理他们创建的的ReplicaSets。Deployments拥有且管理他们的ReplicaSets。

## 什么时候使用ReplicaSet

---



  
  


  
  


