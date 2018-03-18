# Service

---

Kubernetes中Pods是会死亡的。它们出生，然后死亡，不会被复活。[`ReplicationControllers`](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)主要用来动态地创建和销毁Pods（比如，当做一些滚动升级的时候，会对Pods进行伸缩）。尽管每个Pod都有自己的IP地址，但是Pods的IP地址在长时间内并不可靠。这就导致了一个问题：假如在Kubernetes集群中，一组Pods（假设为backends）为另一组Pods（假设为frontends）提供功能，这些frontends如何发现并跟踪哪些Pods在backends中？

这时候，Service就闪亮登场了。

Kubernetes的一个Service是一种抽象，它定义了某些Pods的一个逻辑集合，以及如何访问它们（有时被称为一个微服务）的策略。Service中的Pods集合通常由标签选择器来设定。

作为示例，考虑一个运行了3个副本的图片处理的backend。这些副本都是可替代的--frontends并不关心它们在使用哪个副本。当Service中的Pods发生变化时，frontend客户端并不需要关注这些变化，也不必追踪backends列表。此时，Service这个抽象就实现了这种解耦。

对于原生Kubernetes应用，Kubernetes提供了简单的Endpoints API，它们会在Service中Pods发生改变时而改变。对于非原生Kubernetes应用，Kubernetes提供了为Service提供了一个基于网桥的虚拟IP，可以通过这个IP来连接到backend的Pods。



