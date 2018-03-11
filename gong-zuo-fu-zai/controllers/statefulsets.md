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

你必须将StatefulSet的`spec.selector`字段设置成与它的标签`.spec.template.metadata.labels`相匹配。Kubernetes 1.8版本之前，`spec.selector`字段如果省略了，会设置默认值。在1.8及之后的版本，如果不能定义一个相匹配的Pod选择器，那么会在创建StatefulSet时发生校验错误。

## Pod 身份标识 {#pod-identity}

---

StatefulSet的Pods有一个由一个序号、一个稳定的网络标识和稳定的存储组成的唯一身份标识。无论该Pod被调度到哪个节点，这个身份标识都会与该Pod紧密关联。

### Ordinal Index {#ordinal-index}

对于一个拥有N个副本的StatefulSet来说，每个Pod都会分配一个整数序号，范围是0~N-1，在该StatefulSet内，每个Pod的序号都是唯一的。

### Stable Network ID {#stable-network-id}

StatefulSet中的每个Pod从StatefulSet的名称和Pod的序号派生出其主机名，这种结构的主机名为`$(statefulset name)-$(ordinal)`。上面的例子中，将会创建三个名称为`web-0,web-1,web-2`的Pods。StatefulSet可以使用[Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)来控制Pods的域。该服务会通过`$(service name).$(namespace).svc.cluster.local`的形式管理域，其中“cluster.local”是指集群的域。当每个Pod被创建后，Pod会被分配形如`$(podname).$(governing service domain)`的相匹配的DNS子域，这个governing service 是通过StatefulSet中的`serviceName`属性中定义的。

下面是集群域，服务名称，StatefulSet名称的一些示例选项，并说明了它们是如何影响StatefulSet中Pods的DNS名称的。

| Cluster Domain | Service \(ns/name\) | StatefulSet \(ns/name\) | StatefulSet Domain | Pod DNS | Pod Hostname |
| :--- | :--- | :--- | :--- | :--- | :--- |
| cluster.local | default/nginx | default/web | nginx.default.svc.cluster.local | web-{0..N-1}.nginx.default.svc.cluster.local | web-{0..N-1} |
| cluster.local | foo/nginx | foo/web | nginx.foo.svc.cluster.local | web-{0..N-1}.nginx.foo.svc.cluster.local | web-{0..N-1} |
| kube.local | foo/nginx | foo/web | nginx.foo.svc.kube.local | web-{0..N-1}.nginx.foo.svc.kube.local | web-{0..N-1} |

要注意，集群域会被设置为`cluster.local`，除非[otherwise configured](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#how-it-works)。

### Stable Storage

Kubernetes会为每个VolumeClaimTemplate创建一个[PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)。在上面的例子中，每个Pod都会获得StorageClass为`my-storage-class` ，大小为1Gib的PersistentVolume。如果不指定StorageClass，就会使用默认的StorageClass。当Pod被调度到一个节点是，它的`volumeMounts`会挂载到PersistentVolumes中。注意，与该Pod持久卷声明关联的PersistentVolumes不会在Pods或StatefulSet删除时被删除。它只能通过手动进行删除。

### Pod Name Label {#pod-name-label}

StatefulSet controller创建Pod的时候，会为它添加一个形如`statefulset.kubernetes.io/pod-name`的标签，并将其设置为该Pod的名字。该标签可以允许你为StatefulSet中的Pods附加一个服务。

## Deployment and Scaling Guarantees {#deployment-and-scaling-guarantees}

---

* For a StatefulSet with N replicas, when Pods are being deployed, they are created sequentially, in order from {0..N-1}.
* When Pods are being deleted, they are terminated in reverse order, from {N-1..0}.
* Before a scaling operation is applied to a Pod, all of its predecessors must be Running and Ready.
* Before a Pod is terminated, all of its successors must be completely shutdown.

The StatefulSet should not specify a`pod.Spec.TerminationGracePeriodSeconds`of 0. This practice is unsafe and strongly discouraged. For further explanation, please refer to[force deleting StatefulSet Pods](https://kubernetes.io/docs/tasks/run-application/force-delete-stateful-set-pod/).

When the nginx example above is created, three Pods will be deployed in the order web-0, web-1, web-2. web-1 will not be deployed before web-0 is[Running and Ready](https://kubernetes.io/docs/user-guide/pod-states/), and web-2 will not be deployed until web-1 is Running and Ready. If web-0 should fail, after web-1 is Running and Ready, but before web-2 is launched, web-2 will not be launched until web-0 is successfully relaunched and becomes Running and Ready.

If a user were to scale the deployed example by patching the StatefulSet such that`replicas=1`, web-2 would be terminated first. web-1 would not be terminated until web-2 is fully shutdown and deleted. If web-0 were to fail after web-2 has been terminated and is completely shutdown, but prior to web-1’s termination, web-1 would not be terminated until web-0 is Running and Ready.

### Pod Management Policies {#pod-management-policies}

In Kubernetes 1.7 and later, StatefulSet allows you to relax its ordering guarantees while preserving its uniqueness and identity guarantees via its`.spec.podManagementPolicy`field.

#### OrderedReady Pod Management {#orderedready-pod-management}

`OrderedReady`pod management is the default for StatefulSets. It implements the behavior described[above](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#deployment-and-scaling-guarantees).

#### Parallel Pod Management {#parallel-pod-management}

`Parallel`pod management tells the StatefulSet controller to launch or terminate all Pods in parallel, and to not wait for Pods to become Running and Ready or completely terminated prior to launching or terminating another Pod.

  


  
  


