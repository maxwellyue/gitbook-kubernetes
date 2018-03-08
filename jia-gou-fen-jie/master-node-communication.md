英文地址：[https://kubernetes.io/docs/concepts/architecture/master-node-communication/](https://kubernetes.io/docs/concepts/architecture/master-node-communication/)

## 简述 {#overview}

---

本小节主要介绍master（实际是apiserver）与Kubernetes集群的通信方式，目的是为了让用户定制自己的安装方式来增强网络配置，从而使集群可以运行在不可信的网络中（or on fully public IPs on a cloud provider\)。

## Cluster -&gt; Master {#cluster---master}

---

所有从集群中向Master的路径最终都会到达apiserver（master的其他组件不会向外暴露远程服务）。在一个典型的部署中，apiserver被配置成在安全端口（443）监听一个或更多不同形式的已认证的客户端的连接。应该开启不同的认证形式，尤其是在允许匿名请求或者[service account tokens](https://kubernetes.io/docs/admin/authentication/#service-account-tokens)的时候。

节点应该配备集群的公共的公共根证书，以便可以通过有效的客户端证书安全地连接到apiserver。例如，在默认的GCE部署中，kubelet会用所在主机的证书来当做凭证。关于自动提供kubelet客户端证书的部分见[kubelet TLS bootstrapping](https://kubernetes.io/docs/admin/kubelet-tls-bootstrapping/)。

如果Pods想与apiserver进行安全连接，那么Kubernetes就可以在实例化该Pod的时候，将公共根证书和一个有效的bearer token自动注入到该Pod，这样Pods就可以借助该服务账号来与apiserver进行安全连接了。The`kubernetes`service \(in all namespaces\) is configured with a virtual IP address that is redirected \(via kube-proxy\) to the HTTPS endpoint on the apiserver.

master的其他组件也通过安全端口与集群的apiserver进行通信。

这样，集群（节点以及运行在其中的Pods）与master通信的默认操作模式就是默认安全，并建立在不受信任和/或公共网络中。

## Master -&gt; Cluster {#master---cluster}

---

从Master（指apiserver）到集群中的通信路径主要有两种。第一种是apiserver到集群中任一节点中进行的kubelet进程。第二种是通过apiserver的代理功能从apiserver到任意节点、Pod或Service。

### apiserver -&gt; kubelet {#apiserver---kubelet}

从apiserver到kubelet的通信主要用来：

* 获取Pods的日志
* 为运行中的Pods设置一些额外信息（通过kubectl）
* 让kubelet’s拥有端口转发功能

这些连接最终到达kubelet终端。默认情况下，apiserver不会验证kubelet的服务证书，这会导致连接可能会遭受中间人攻击，并使在不受信任或公共网络运行时不安全。

如果想要验证这种连接，可以使用`--kubelet-certificate-authority`标识附带根证书，以此让apiserver来验证kubelet的服务证书。

如果不能使用这种方式，并且需要避免通过不可信或公共网络连接，那么可以在apiserver和kubelet之间来使用SSH隧道的方式。

最后，应该开启[Kubelet authentication and/or authorization](https://kubernetes.io/docs/admin/kubelet-authentication-authorization/)来确保kubelet API的安全。

### apiserver -&gt; nodes, pods, and services {#apiserver---nodes-pods-and-services}

从apiserver到node，pod或者 service的连接默认是普通的HTTP连接，因此这种连接没有经过认证也没有经过加密。这些连接可以通过在在API URL前面添加`https:`使用安全的HTTPS的方式进行，但是即使这些连接经过了加密，但是不会去校验HPPTS终端所提供的证书，也不会提供客户端证书，因此它不会对连接的正确性（integrity）提供任何保证。目前，这些连接在不受信任或公共网络中的运行是不安全的。

### SSH Tunnels {#ssh-tunnels}

---

[Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) 使用 SSH 隧道来保护Master到 集群的通信路径。在这种配置方式下，apiserver会与节点中的所有节点分别初始化一个SSH隧道，并通过该隧道来向kubelet、node、 pod 或者 service 来传输数据。这种隧道会确保数据不会暴露到保证运行着集群的私有GCE网络中。

