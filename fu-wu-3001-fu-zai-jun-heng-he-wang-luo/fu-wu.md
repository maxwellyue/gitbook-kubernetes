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

* 你想在生产环境中使用一个外部数据库集群，但在测试环境中使用你自己的数据库。

* 你想为在另一个命名空间或另一个集群中的服务提供接入。

* 你正将你的工作负载迁移到Kubernetes中，但一些backends运行在Kubernetes之外。

在这些场景，你可以定义一个没有选择器的Service：

```
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

因为该服务没有选择器，所以不会创建相应的`Endpoints`对象。你需要手动将该服务与你自己定义的endpoints对应上：

```
kind: Endpoints
apiVersion: v1
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 1.2.3.4
    ports:
      - port: 9376
```

注意：Endpoint的IPs may not be loopback \(127.0.0.0/8\), link-local \(169.254.0.0/16\), or link-local multicast \(224.0.0.0/24\).

访问没有选择器的Service，与访问有选择器的Service是一样的。流量将被路由到用户定义的endpoints上（ 上面的例子中就是`1.2.3.4:9376`\)。

一个ExternalName类型的service是一种特殊的没有选择器的服务。它不会定义任何的端口或者endpoints。Rather, it serves as a way to return an alias to an external service residing outside the cluster.

```
kind: Service
apiVersion: v1
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

当寻找`my-service.prod.svc.CLUSTER`主机时，集群DNS服务将返回一个CNAME记录，其值为`my.database.example.com`。

访问这样的服务，与访问其他服务是一样的，唯一的不同是，重定向发生在DNS层，不会发生代理或转发。之后你如果决定将你的数据库迁移到集群中，你可以启动它的Pods，添加适当的标签或者endpoints，并修改Service的类型。

## 虚拟IP和服务代理 {#virtual-ips-and-service-proxies}

---

Kubernetes集群中的每一个节点上都会运行一个`kube-proxy`。`kube-proxy`负责实现除`ExternalName`类型之外的Service的虚拟IP功能。在Kubernetes v1.0中，Service是四层结构的（TCP/UDP over IP\) ，代理完全是在用户空间中。在Kubernetes v1.1中，`Ingress`API 被添加进来，代表7层（HTTP）结构的Service，同时也添加了iptables，并且自Kubernetes v1.2起就成为了默认的操作模型。在Kubernetes v1.8.0-beta.0中， 又添加了ipvs代理。

### 代理模式：userspace

在这种模式下，kube-proxy会监听Kubernetes Master中的Service和Endpoints的新增和移除。它会为每一个Service在本地节点上打开一个端口（随机选择）。任何该代理端口的连接都会被代理到该Service的一个Pod上（被记录在Endpoints中）。使用哪个Pod是由Service的`SessionAffinity`决定的。Lastly, it installs iptables rules which capture traffic to the`Service`’s`clusterIP`\(which is virtual\) and`Port`and redirects that traffic to the proxy port which proxies the backend`Pod`. By default, the choice of backend is round robin.![](/assets/屏幕快照 2018-03-18 下午6.48.08.png)

上图中的ServiceIP就是clusterIP。

### 代理模式: iptables {#proxy-mode-iptables}

在这种模式下，kube-proxy会监听Kubernetes Master中的Service和Endpoints的新增和移除。对每一个Service，it installs iptables rules which capture traffic to the`Service`’s`clusterIP`\(which is virtual\) and`Port`and redirects that traffic to one of the`Service`’s backend sets. For each`Endpoints`object, it installs iptables rules which select a backend`Pod`. By default, the choice of backend is random.

显然，iptables不需要在userspace和kernelspace之间切换，它比userspace代理更快，更可靠。However, unlike the userspace proxier, the iptables proxier cannot automatically retry another`Pod`if the one it initially selects does not respond, so it depends on having working[readiness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#defining-readiness-probes).![](/assets/屏幕快照 2018-03-18 下午6.55.27.png)上图中的ServiceIP就是clusterIP。

### 代理模式: ipvs {#proxy-mode-ipvs}

**FEATURE STATE:**`Kubernetes v1.9`[beta](https://kubernetes.io/docs/concepts/services-networking/service/#)

In this mode, kube-proxy watches Kubernetes Services and Endpoints, calls`netlink`interface to create ipvs rules accordingly and syncs ipvs rules with Kubernetes Services and Endpoints periodically, to make sure ipvs status is consistent with the expectation. When Service is accessed, traffic will be redirected to one of the backend Pods.

Similar to iptables, Ipvs is based on netfilter hook function, but uses hash table as the underlying data structure and works in the kernel space. That means ipvs redirects traffic much faster, and has much better performance when syncing proxy rules. Furthermore, ipvs provides more options for load balancing algorithm, such as:

* `rr`: round-robin
* `lc`: least connection
* `dh`: destination hashing
* `sh`: source hashing
* `sed`: shortest expected delay
* `nq`: never queue

**Note:**ipvs mode assumes IPVS kernel modules are installed on the node before running kube-proxy. When kube-proxy starts with ipvs proxy mode, kube-proxy would validate if IPVS modules are installed on the node, if it’s not installed kube-proxy will fall back to iptables proxy mode.![](/assets/屏幕快照 2018-03-18 下午7.02.33.png)在这些代理模式中，任何绑定到该服务IP:PORT的流量都会被代理到一个适当的后端中，而客户端不会知道任何关于Kubernetes、Service或者Pods的信息。Client-IP based session affinity can be selected by setting`service.spec.sessionAffinity`to “ClientIP” \(the default is “None”\), and you can set the max session sticky time by setting the field`service.spec.sessionAffinityConfig.clientIP.timeoutSeconds`if you have already set`service.spec.sessionAffinity`to “ClientIP” \(the default is “10800”\).

## 多端口 Services {#multi-port-services}

---

很多Service需要暴露不止一个端口。在这种情况下，Kubernetes支持对同一个Service对象定义多个端口。当使用多个端口时，你必须给定所有的端口名字，这样endpoints才不会有歧义。比如：

```
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 9376
  - name: https
    protocol: TCP
    port: 443
    targetPort: 9377
```



