英文地址：[https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

# Labels 和 Selectors

---

标签Labels是指一些附加到对象（如pods）上的键值对。标签是用来对对象定义一些对用户有用或相关的属性。标签可用来组织和选择对象的子集。标签可以在创建的时候附加到对象上，也可以在之后的任何时间进行添加和修改。每一个对象都可以有一组Labels键值对。对于给定的一个对象，每一个标签中的key不能重复。

```
"labels": {
  "key1" : "value1",
  "key2" : "value2"
}
```

我们最终会对标签进行索引，以便高效查询和监控对象，并在客户端、UI界面中等使用它们进行排序、分组。在标签定义中，我们不会使用非标识、很大的数据。一些非标识性的信息，应该使用[annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)来进行记录。

# 为啥要使用Labels

---

标签可以让用户以松散耦合的方式来按照自己的组织架构去映射对象。

服务部署和批处理管道通常是多维度的实体（e.g., multiple partitions or deployments, multiple release tracks, multiple tiers, multiple micro-services per tier）。对这些实体的管理往往需要跨领域的操作，这打破了严格分层表示的封装，特别是由基础设施而不是用户确定的严格层级。

标签示例：

* `"release" : "stable"`，`"release" : "canary"`

* `"environment" : "dev"`，`"environment" : "qa"`，`"environment" : "production"`

* `"tier" : "frontend"`，`"tier" : "backend"`，`"tier" : "cache"`

* `"partition" : "customerA"`，`"partition" : "customerB"`

* `"track" : "daily"`，`"track" : "weekly"`

上面的标签都是最常用的，你可以根据自身的惯例来设置标签。但请记住对于同一个对象，标签的key不可重复！

# 语法和字符集

---

Label其实是一对 key/value。有效的Label keys必须包含两部分：一个可选前缀+名称，通过斜线/来区分，名称部分是必须的，并且最多63个字符，开始和结束的字符必须是字母或者数字，中间是字母数字和”\_”，”-“，”.”，前缀是可有可无的，如果指定了，那么前缀必须是一个DNS子域，一系列的DNS label通过”.”来划分，长度不超过253个字符，并以“/”来结尾。如果前缀被省略了，这个Label的key被假定为对用户私有的。系统组件（比如kube-scheduler, kube-controller-manager, kube-apiserver, kubectl）,这些为最终用户添加标签的组件必须要指定一个前缀。`Kuberentes.io/` 前缀是为Kubernetes 核心组件的保留前缀。

合法的label values必须是63个或者更短的字符。要么是空，要么首位字符必须为字母数字字符，中间必须是横线，下划线，点或者数字字母。

# 标签选择器Label selectors

---

与names和UID不同，标签不必是唯一的，实际上，我们期望多个对象拥有相同的标签。

通过标签选择器，客户端/用户可以选择一组对象。标签选择器是Kubernetes中的核心分组原语。

Kubernetes API目前支持两种类型的选择器：_equality-based_和_set-based_。一个标签选择器可以由多个必须条件组成，并以逗号分隔。在多个必须条件指定的情况下，所有的条件都必须满足，因而逗号起着AND逻辑运算符的作用。

一个empty的label选择器（即有0个必须条件的选择器）会选择集合中的每一个对象。

一个null的label选择器（仅对于可选的选择器字段才可能）不会返回任何对象。

**注意**：两个controllers的标签选择器在同一命名空间内不能重叠，否则会相互冲突。

### 基于相等的条件

基于相等性或者不相等性的条件允许用label的键或者值进行过滤。匹配的对象必须满足所有指定的label约束，尽管他们可能也有额外的label。有三种运算符是允许的，“=”，“==”和“!=”。前两种代表相等性（他们是同义运算符），后一种代表非相等性。例如：

```
environment = production
tier != frontend
```

第一个会选择所有键等于 environment 值为 production 的资源；后一种会选择所有“键为 tier 值不等于 frontend ”以及“没有键为 tier “的label的资源。你可以通过下面的条件来过滤所有处于 production 但不是 frontend 的资源： `environment=production , tier!=frontend` 。

”基于相等的条件“这种方式的一种用途是为Pods设置node选择标准。比如，下面的Pod会选择拥有`accelerator=nvidia-tesla-p100`标签的node。

```
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

### 基于集合的条件

基于集合的label条件允许用一组值来对key进行过滤。支持三种操作符: in ， notin和 exists\(仅针对于key符号\) 。例如：

```
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

第一个例子，选择所有键等于 environment ，且value等于 production 或者 qa 的资源。

第二个例子，选择所有键等于 tier 且值是除了 frontend 和 backend 之外的资源，和那些没有label的键是 tier 的资源。

第三个例子，选择所有所有有一个label的键为partition的资源（无论值是什么）。

第四个例子，选择所有的没有lable的键名为 partition 的资源。

类似地，逗号操作符相当于一个AND操作符。因而如果要选择一个`partition`键（不管值是什么），并且 `environment`不是`qa`资源，可以用 `partition, environment notin (qa)` 。

基于集合的选择器是一个相等性的宽泛的形式，因为 `environment=production` 相当于`environment in (production)`。

基于集合的条件可以与基于相等性的条件混合。例如， `partition in (customerA, customerB),environment!=qa` 。

# API

---

#### LIST 和WATCH过滤

LIST和WATCH操作，可以使用query参数来指定label选择器来过滤返回对象的集合。上面介绍的两种条件类型都可以使用。

基于相等条件，比如：  
`$ kubectl get pods -l 'environment=production,tier=frontend'`

基于集合条件，比如：  
`$ kubectl get pods -l 'environment in (production),tier in (frontend)'`

#### Set references in API objects

一些Kubernetes对象，比如[`services`](https://kubernetes.io/docs/user-guide/services)和[`replicationcontrollers`](https://kubernetes.io/docs/user-guide/replication-controller)，也会使用标签选择器来定义其他资源，比如[pods](https://kubernetes.io/docs/user-guide/pods)。

### Service 和 ReplicationController

一个service所对应的pods集合是通过标签选择器来定义的。类似地，一个ReplicationController所管理的pods集合也是通过标签选择器来定义的。

这两个对象所使用的标签选择器使用maps的方式被定义在json或yaml文件中，并且**只支持相等条件**的选择器，比如

```
"selector":{
    "component":"redis",
}
```

或者

```
selector:
    component: redis
```

上面定义的选择器的条件是：`component=redis`或者`component in (redis)`。

### 支持集合条件的资源

比较新的资源（即Kubernetes中较高版本中出现的），如[`Job`](https://kubernetes.io/docs/concepts/jobs/run-to-completion-finite-workloads/)，[`Deployment`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)，[`Replica Set`](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)，和[`Daemon Set`](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)，都同时支持集合条件。

```
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

matchLabels是一个键值对的map。一个单独的 {key,value} 相当于 matchExpressions 的一个元素，它的键字段是”key”,操作符是 In ，并且值数组值包含”value”。 matchExpressions 是一个pod的选择器条件的列表。合法的操作符包含In, NotIn, Exists, 和 DoesNotExist。在In和NotIn的情况下，值的组必须不能为空。所有的条件，包含 matchLabels 和matchExpressions 中的，会用AND符号连接，他们必须都被满足以完成匹配。

#### Selecting sets of nodes {#selecting-sets-of-nodes}

还有一种用例是：在调度pod到node时，使用选择器来限制nodes。见[node selection](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)。





