英文地址：[https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

# StatefulSets

---

StatefulSet是一个工作负载API对象（the workload API object），被用来管理有状态应用。

> **Note:**StatefulSets从1.9版本开始稳定。

管理deployment，对Pods进行伸缩，提供Pods的有序性和唯一性。

就像[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)，一个StatefulSet管理那些基于相同容器定义的Pods。与Deployment不同的是，一个StatefulSet为每个它管理的Pods维护一个粘性身份（sticky identity）。这些pods通过相同的spec被创建，但并不通用：无论经过多少次重新调度，每一个Pod都有一个持久的识别码。

StatefulSet与其他Controller的工作模式相同。你在一个StatefulSet对象中定义你期望的状态，StatefulSet Controller就会将它从当前状态更新到你期望的状态。

## 使用StatefulSets

---

如果应用需要满足下列的要求，StatefulSets就会非常有价值：

* 稳定，唯一的网络身份识别。
* 稳定，持久的存储。
* 有序的，优雅的部署和伸缩。
* 有序的，优雅的删除和终止。
* 有序的，自动的滚动升级。

上面说到的稳定，是指在Pod调度中可以保持一致。如果一个应用不需要任何的稳定的身份或有序的部署、删除，或者伸缩，那么你应该使用可以提供无状态副本集的controller。如[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)或者 [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)这样的Controllers会更适合无状态应用。 

## 一些限制 {#limitations}

---

* StatefulSet was a beta resource prior to 1.9 and not available in any Kubernetes release prior to 1.5.
* As with all alpha/beta resources, you can disable StatefulSet through the
  `--runtime-config`
  option passed to the apiserver.
* The storage for a given Pod must either be provisioned by a
  [PersistentVolume Provisioner](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/README.md)
  based on the requested
  `storage class`
  , or pre-provisioned by an admin.
* Deleting and/or scaling a StatefulSet down will
  _not_
  delete the volumes associated with the StatefulSet. This is done to ensure data safety, which is generally more valuable than an automatic purge of all related StatefulSet resources.
* StatefulSets currently require a
  [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)
  to be responsible for the network identity of the Pods. You are responsible for creating this Service.

  


  


