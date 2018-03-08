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
* 
* Fetching logs for pods.
* Attaching \(through kubectl\) to running pods.
* Providing the kubelet’s port-forwarding functionality.

These connections terminate at the kubelet’s HTTPS endpoint. By default, the apiserver does not verify the kubelet’s serving certificate, which makes the connection subject to man-in-the-middle attacks, and**unsafe**to run over untrusted and/or public networks.

To verify this connection, use the`--kubelet-certificate-authority`flag to provide the apiserver with a root certificate bundle to use to verify the kubelet’s serving certificate.

If that is not possible, use[SSH tunneling](https://kubernetes.io/docs/concepts/architecture/master-node-communication/#ssh-tunnels)between the apiserver and kubelet if required to avoid connecting over an untrusted or public network.

Finally,[Kubelet authentication and/or authorization](https://kubernetes.io/docs/admin/kubelet-authentication-authorization/)should be enabled to secure the kubelet API.

### apiserver -&gt; nodes, pods, and services {#apiserver---nodes-pods-and-services}

The connections from the apiserver to a node, pod, or service default to plain HTTP connections and are therefore neither authenticated nor encrypted. They can be run over a secure HTTPS connection by prefixing`https:`to the node, pod, or service name in the API URL, but they will not validate the certificate provided by the HTTPS endpoint nor provide client credentials so while the connection will be encrypted, it will not provide any guarantees of integrity. These connections**are not currently safe**to run over untrusted and/or public networks.

### SSH Tunnels {#ssh-tunnels}

---

[Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/)uses SSH tunnels to protect the Master -&gt; Cluster communication paths. In this configuration, the apiserver initiates an SSH tunnel to each node in the cluster \(connecting to the ssh server listening on port 22\) and passes all traffic destined for a kubelet, node, pod, or service through the tunnel. This tunnel ensures that the traffic is not exposed outside of the private GCE network in which the cluster is running.

