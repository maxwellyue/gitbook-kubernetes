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

* 在1.9版本之前，StatefulSet 是一个测试资源；在1.5之前的版本，它是不可用的。
* 与其他alpha/beta资源一样，你可以通过设置`--runtime-config`选项给apiserver来关闭StatefulSet。
* 一个给定Pod的存储应该由基于请求的`storage class`的[PersistentVolume Provisioner](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/README.md)提供或者由管理员预先配置。
* 删除或缩减一个StatefulSet不会删除与之关联的存储卷。这是为了确保数据的安全，这种做法比自动清除与之关联的资源更有价值。
* 目前，StatefulSets 需要一个[Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)来负责Pods的网络标识。你要负责创建该服务。

## 组成部分（Components） {#components}

---

下面的示例展示了StatefulSet的组成部分：

* 一个名为nginx的Headless Service，用来控制网络域。
* 一个名为web的StatefulSet，它的Spec部分定义了3个nginx容器副本将会运行在不同的Pods中。
* volumeClaimTemplates将会使用PersistentVolume Provisioner提供的[PersistentVolumes](https://www.gitbook.com/book/maxwellyue/kubernetes-learning/edit#)来提供稳定的存储。

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

## Pod 选择器 {#pod-selector}

---

You must set the`spec.selector`field of a StatefulSet to match the labels of its`.spec.template.metadata.labels`. Prior to Kubernetes 1.8, the`spec.selector`field was defaulted when omitted. In 1.8 and later versions, failing to specify a matching Pod Selector will result in a validation error during StatefulSet creation.

  


