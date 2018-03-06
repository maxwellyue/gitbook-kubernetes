英文地址：[https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

# Namespaces

---

Kubernetes支持在相同物理机集群中创建多个虚拟集群。这些虚拟集群被称为namespaces。

# 什么时候使用Multiple Namespaces

---

Namespaces被用来在一个拥有多个团队或多个项目的很多用户的环境中使用。对于只有几十个用户的集群来讲，你根本就无需创建或考虑使用namespace。当你需要使用namespaces提供的特性的时候，再开始使用namespaces。

命名空间为names提供了作用域（scope）。同一命名空间内的资源的名字必须唯一，但在多个命名空间内则不必如此。

命名空间是一种在多用户之间将集群资源进行隔离的手段。

在Kubernetes的未来版本中，同一命名空间下的对象将默认使用相同的访问控制策略。

仅仅为了隔离不同的资源，如同一应用的不同版本，这是不必要的。此时，你可以使用labels来区分这些同一命名空间内的资源。

# 使用Namespaces

---

关于命名空间的创建和删除，见[Admin Guide documentation for namespaces](https://kubernetes.io/docs/admin/namespaces)。

### 查看命名空间

你可以使用如下命令来列出集群中的当前的命名空间：

```
$ kubectl get namespaces
NAME          STATUS    AGE
default       Active    1d
kube-system   Active    1d
kube-public   Active    1d
```

Kubernetes在启动时，会初始化下面的3个命名空间：

* default：没有指定命名空间的对象的默认命名空间
* kube-system：kubernetes系统创建的对象
* kube-public：该命名空间会自动创建，且对所有用户可见（包括没有认证的用户）。该命名空间主要为在整个集群中都可以使用的资源而保留的（考虑到在某些情况下，一些资源应该被整个集群可见）。这仅仅是一种惯例，而非约束。

### 为请求设置命名空间

暂时地设定一个请求的命名空间，使用`--namespace`参数。

例如：

```
$ kubectl --namespace=<insert-namespace-name-here> run nginx --image=nginx
$ kubectl --namespace=<insert-namespace-name-here> get pods
```

### 为请求设置默认命名空间

你可以为同一个Context下，设置接下来的kubelet命令全部都使用某一种命名空间：

```
$ kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name-here>
# Validate it
$ kubectl config view | grep namespace:
```

# Namespaces和DNS

---

当你创建了一个[Service](https://kubernetes.io/docs/user-guide/services)，它会创建一个与之相应的[DNS entry](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)。该entry的格式如`<service-name>.<namespace-name>.svc.cluster.local`，这表示：如果一个容器仅仅使用&lt;service-name&gt;，它会解析到一个namespace中的service。当你在多命名空间中（如开发、Staging、生产）使用相同的配置的时候就非常有用。如果你想跨命名空间进行访问，你必须使用fully qualified domain name \(FQDN\)。

# 并不是所有的对象都在Namespace中

---

大多数Kubernetes资源（如pods、services、replication controller等）都在命名空间的管理范畴。但命名空间资源自身并不在命名空间中。还有一些低级别的资源，如Nodes和PV等，并不属于任何命名空间。事件是一种例外：它们可能有命名空间，也可能没有，取决于事件是什么样的事件。













