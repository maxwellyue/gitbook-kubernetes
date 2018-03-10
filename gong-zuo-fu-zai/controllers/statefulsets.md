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





