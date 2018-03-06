英文地址：[https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)

# 理解Kubernetes对象

---

Kubernetes中的对象在Kubernetes系统中是持久化了的实体。Kubernetes使用这些实体来表示你的集群的状态。

它们可以描述为：

* 什么样的容器化的应用在运行，运行在哪个节点上
* 应用的可用资源有哪些
* 应用行为的规则，比如重启策略、升级、容错

一个Kubernetes对象是一个意图的记录（record of intent）：一旦你创建了一个对象，Kubernetes系统就会持续工作来保证该对象存在。通过创建一个对象，你可以很清楚地告诉Kubernetes系统你想要你的集群如何工作，即你的集群的期望状态。

为了操作kubernetes对象，无论是创建、修改或者删除，你必须使用Kubernetes API。比如，当你使用命令行工具kubectl时，kubectl其实是在调用Kubernetes API。你也可以直接在你自己的程序中使用客户端依赖包来直接调用Kubernetes API。

### 对象定义和状态（Object Spec and Status）

每一个Kubernetes对象都有两个嵌套的属性来管理该对象的配置：spec和status。spec属性，必须由用户来指定，描述该对象的一些特征。status属性是该对象的实际状态，是由Kubernetes系统来提供和更新的。在任意一个时间点，Kubernetes控制面板都在致力于让一个对象的实际状态与你指定的期望状态保持一致。

举个例子，kubernetes中，Deployment对象可以用来表示你的集群中应用程序正在运行。当你创建了一个Deployment，你可能会设置Deployment的spec属性来定义你想要运行3个程序副本。Kubernetes系统会读取Deployment的spec属性，并启动3个应用的实例——即更新该对象的status，使其与你期望的一致。如果这些实例中的任何一个失败了（表示状态发生了变化），Kubernetes系统就会对spec和当前状态之间的差别做出响应，在这个例子中，就是启动一个新的应用实例来替代失败的那个应用实例。

更多关于对象的spec，status，metadata，见[Kubernetes API Conventions](https://git.k8s.io/community/contributors/devel/api-conventions.md)。

### 描述一个Kubernetes对象

当你在Kubernetes中创建一个对象，你必须提供对象的spec来描述你期望的状态以及一些该对象的基本信息（ 比如名字）。当你使用Kubernetes API来创建对象时（无论是直接调用还是通过kubectl），API请求中必须在请求体中以JSON的格式来提供这些信息。最常见的方式是，你通过yaml文件的形式为kubectl提供信息。kubectl在向Kubernetes API发送请求的时候，会将yaml文件中的信息转换为JSON格式。

下面是一个yaml文件（`nginx-depolyment.yaml`）的示例，它演示了定义Deployment对象时必须的字段和spec属性。

```
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
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

使用上面的yaml文件来创建Deployment时，可以在`kubectl create`命令中将yaml文件当做一个参数传入，如：

```
$ kubectl create -f https://k8s.io/docs/user-guide/nginx-deployment.yaml --record
```

该命令的执行结果大概如下：

```
deployment "nginx-deployment" created
```

### 必须的字段（Required Fields）

在yaml文件中，你必须为下列字段设置值：

* apiVersion：你使用Kubernetes API的哪个版本来创建该对象
* kind：对象的类型
* metadata：帮你对对象进行唯一标识的数据，包括name、UID以及命令空间namespace。

同时，你也必须为对象设置spec属性。对于不同的Kubernetes对象，spec的格式不同，并包含不同的嵌套字段来描述对象。[Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/)可以帮助你找到你可以在Kubernetes中创建的所有类型的对象的spec格式。比如，你可以在[这里](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#podspec-v1-core)找到Pod的spec格式，在[这里](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#deploymentspec-v1-apps)找到Deployment对象的spec格式。

