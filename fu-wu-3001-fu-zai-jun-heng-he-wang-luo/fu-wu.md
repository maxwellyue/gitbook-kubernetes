# Service

---

Kubernetes中Pods是会死亡的。它们出生，然后死亡，不会被复活。[`ReplicationControllers`](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)主要用来动态地创建和销毁Pods（比如，当做一些滚动升级的时候，会对Pods进行伸缩）。尽管每个Pod都有自己的IP地址，但是Pods的IP地址在长时间内并不可靠。这就导致了一个问题：假如在Kubernetes集群中，一组Pods（假设为backends）为另一组Pods（假设为frontends）提供功能，这些frontends如何发现并跟踪哪些Pods在backends中？

这时候，Service就闪亮登场了。

Kubernetes的一个Service是一种抽象，它定义了某些Pods的一个逻辑集合，以及如何访问它们（有时被称为一个微服务）的策略。Service中的Pods集合通常由标签选择器来设定。

作为示例，考虑一个运行了3个副本的图片处理的backend。这些副本都是可替代的--frontends并不关心它们在使用哪个副本。当Service中的Pods发生变化时，frontend客户端并不需要关注这些变化，也不必追踪backends列表。此时，Service这个抽象就实现了这种解耦。

对于原生Kubernetes应用，Kubernetes提供了简单的Endpoints API，它们会在Service中Pods发生改变时而改变。对于非原生Kubernetes应用，Kubernetes提供了为Service提供了一个基于网桥的虚拟IP，可以通过这个IP来连接到backend的Pods。

## 定义一个service {#defining-a-service}

---

Kubernetes中的一个A`Service`是一个REST对象，与Pod相似。与其他REST对象一样，你可以将一个`Service`的定义传给apiserver来创建一个Service实例。比如，假设你有一组Pods，每个Pod都暴露9376端口，并设置了一个`"app=MyApp"`标签。

```
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

上面的定义就会创建一个新的Service对象，名为“my-service”，目标是有`"app=MyApp"`标签的所有Pods，且监听这些Pods的9376端口。这个Service也会分配一个IP地址（有时被称为集群IP），服务代理会使用该IP。该Service的标签选择器将会被持续地计算，计算结果会传给一个名字也是“my-service”的`Endpoints`。

要注意，一个Service可以将传入端口映射到任意的`targetPort`。默认情况下，`targetPort`会被设置成与`port`字段相同的值。更有趣的是，`targetPort`可以是一个字符串，该字符串指向Pods的端口的名字。 在不同的Pods中，分配到这个名字的实际端口号可以不同。这为你部署和升级你的Service提供了很多灵活性。比如，你可以在你的后端软件的下一个版本中改变端口号，而不必与客户端断开。

Kubernetes中`Services`支持`TCP`和`UDP`协议。默认是`TCP`协议。

#### 没有选择器的Service

Services通常会抽象Pods的访问，但是Services也可以抽象其他类型backends。比如：

Services generally abstract access to Kubernetes`Pods`, but they can also abstract other kinds of backends. For example:

* 你想在生产环境中使用一个外部数据库集群，但在测试环境中使用你自己的数据库。
* 你想为在另一个命名空间或另一个集群中的服务提供接入。
* 你正将你的工作负载迁移到Kubernetes中，但一些backends运行在Kubernetes之外。

In any of these scenarios you can define a service without a selector:

