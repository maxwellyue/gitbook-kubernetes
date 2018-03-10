英文地址：[https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

# StatefulSets

---

StatefulSet是一个工作负载API对象（the workload API object），被用来管理有状态应用。 

**Note:**StatefulSets从1.9版本开始稳定。 

Manages the deployment and scaling of a set of[Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/),_and provides guarantees about the ordering and uniqueness_of these Pods.

Like a[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

A StatefulSet operates under the same pattern as any other Controller. You define your desired state in a StatefulSet_object_, and the StatefulSet_controller_makes any necessary updates to get there from the current state.

* 



