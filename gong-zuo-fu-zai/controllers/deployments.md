英文地址：[https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

# Deployments

---

Deployments为[Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)和[ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)提供声明性的更新（declarative updates）。

你在Deployment对象中描述一个期望状态，Deployment controller就会以可以控制的速率将实际状态转换为期望状态。你可以通过定义Deployments来创建新的ReplicaSets，或者来删除已存在的Deployments并adopt all their resources with new Deployments。

> 注意：You你不应该去管理被一个Deployment所拥有的ReplicaSets。所有的操作都应该通过操作Deployment对象来完成。

## 用例

---

以下是Deployment的典型用例：

* 创建一个Deployment来rollout一个ReplicaSet。ReplicaSet会在后台创建Pods。通过检查rollout的状态来判断是否成功。

* 通过更新Deployment的PodTemplateSpec来声明Pods的新状态。这样，就会创建一个新的ReplicaSet，Deployment会以可控的速率将Pods从旧ReplicaSet转移到这个新的ReplicaSet中。每一次新的ReplicaSet都会更新Deployment的版本。

* 如果当前Deployment的状态不稳定，可以回滚到一个之前的Deployment版本中。每一次回滚都会更新Deployment的版本。

* 扩展（scale up）Deployment来应对更多负载。

* 暂停一个Deployment来应用多个PodTemplateSpec修复，并恢复Deployment来开启一个新的rollout。

* 使用Deployment的状态来作为判断rollout是否卡住的指示器。

* 清除你不再需要的旧的ReplicaSet。

## 创建一个Deployment

---

下面是Deployment的一个示例。它会创建一个ReplicaSet来运行3个nginx Pods。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

在这个示例中：

* 创建了一个名称为`nginx-deployment`的Deployment，通过`metadata: name`字段来指定。
* 这个Deployment会创建3个Pods副本，通过`replicas`字段来指定。













