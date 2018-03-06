英文地址：[https://kubernetes.io/docs/concepts/overview/object-management-kubectl/overview/](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/overview/)

命令行工具kubectl提供了多种创建和管理Kubernetes对象的方式。本节对就会对围绕这几种不同的方式展开。

## Management techniques {#management-techniques}

---

**Warning：**一个Kubernetes对象必须使用同一种管理方式来进行管理。如果混用不同的方式对同一对象进行管理，将会导致不可预料的行为。

| 管理方式 | 操作对象 | 推荐环境 | 并发写 | 学习曲线 |
| :--- | :--- | :--- | :--- | :--- |
| imperative commands | 活着的对象 | 开发环境项目 | 1+ | 最低 |
| imperative object configuration | 单独的文件 | 生产环境项目 | 1 | 中等 |
| declarative object configuration  | 文件的目录 | 生产环境项目 | 1+ | 最高 |

## Imperative commands {#imperative-commands}

---

使用Imperative commands的时候，用户会直接操作集群中存活的对象。用户将操作指令当做一个参数或标识传给kubectl。

这是在集群中启动或运行一个一次性任务的最简单的方式。因为这种方式直接在存活的对象上进行操作，因此它不会提供之前配置的历史记录。

##### 例子

通过创建一个Deployment对象来运行一个nginx容器实例：

```
kubectl run nginx --image nginx
```

也可以使用如下语法，可以达到同样的效果：

```
kubectl create deployment nginx --image nginx
```

##### 权衡（Trade-offs）

相比对象配置的优势：

* 使用命令很简单，容易学习，也容易记住。
* 使用命令只需要简单的一步操作就可以对集群做出改变。

相比对象配置的缺点：

* 使用命令与修改审计流程不匹配。
* 使用命令不会提供与修改相关的审计跟踪记录。
* 使用命令没有记录。
* 使用命令无法为创建新对象提供模板。

## Imperative object configuration

---

imperative object configuration这种方式是指：kubectl命令中定义操作类型（创建，替换等）、可选参数以及至少一个文件名称。这个文件必须包含以YAML或JSON的格式完整地定义一个对象。

可以在[API reference](https://kubernetes.io/docs/api-reference/v1.9/)中查阅对象定义的更多细节。

注意**:**`replace`命令会用新提供的spec来替换掉之前存在的spec，且配置文件中缺失的对对象的改变都将被丢弃。这种方式不适用于那些specs与配置文件分离的资源类型。比如，`LoadBalancer`类型的服务，它们的, `externalIPs`字段更新是与集群中的配置分离的。

##### 例子

创建定义在配置文件中的对象：

```
kubectl create -f nginx.yaml
```

删除在两个配置文件定义的对象：

```
kubectl delete -f nginx.yaml -f redis.yaml
```

通过覆盖配置的方式来更新定义在配置文件中的对象：

```
kubectl replace -f nginx.yaml
```

##### 权衡

相比imperative commands的优势：

* 对象的配置文件可以存储在资源控制系统中，如git。
* 对象的配置文件可以与一些流程整合，如在推送和审计跟踪之前审查更改。
* 对象的配置文件为创建对象提供了模板。

相比imperative commands的劣势：

* 对象配置需要对对象的schema有一些基本的理解。
* 对象配置多了一步操作：编写YAML文件。

相比declarative object configuration的优势：

* imperative object configuration的行为简单，更易理解。

* kubernetes 1.5版本中，imperative object configuration更成熟。

相比declarative object configuration的劣势：

* Imperative object configuration可以很好地操作文件，但不能是目录。

* 对存活对象的更新必须反映在配置文件中，否则这些更新将在下次更新的时候丢失。

## Declarative object configuration {#declarative-object-configuration}

---

使用declarative object configuration这种方式，用户会对本地的对象配置文件进行修改，但用户无需定义对文件的操作。每个对象的创建、更新或删除操作都是由kubectl自动检测的。

**注意**：Declarative object configuration这种方式将会保留任何其他用户做出的修改，即使这些修改没有合并回对象配置文件。可以使用patch操作，而不是replace操作来write观察到的区别。

##### 例子

处理config目录下的所有对象配置文件，创建对象或patch已存活的对象：

```
kubectl apply -f configs/
```

递归处理：

```
kubectl apply -R -f configs/
```

##### 权衡

相比imperative object configuration的优势：

* 对存活对象做出的直接更改可以得到保留，即使这些修改没有合并回对象配置文件。
* 对目录的操作支持的更好，并且会自动检测每个对象的操作类型（create，patch,，delete）。

相比imperative object configuration的劣势：

* 难以调试，很难理解未预期结果的出现。
* 使用差异的部分更新会创建复杂的合并和修补操作。









