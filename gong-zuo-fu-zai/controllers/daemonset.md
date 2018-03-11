英文地址：[https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

## DaemonSet是什么？ {#what-is-a-daemonset}

---

_DaemonSet_用来确保所有（或部分）节点都会运行一个Pod的拷贝。每当有新节点添加到集群中，那么Pods也会添加到其中。当节点从集群中删除，这些Pods就会被回收。删除一个DaemonSet将会清除它创建的Pods。

一些DaemonSet典型的使用场景为：

* 在每个节点运行集群存储守护进程，比如`glusterd`，`ceph`。

* 在每个节点上运行日志收集守护进程，比如`fluentd`或者`logstash`。

* 在每个节点运行监控守护进程，比如[Prometheus Node Exporter](https://github.com/prometheus/node_exporter)，`collectd`，Datadog agent，New Relic agent，或者 Ganglia`gmond`。

在一个简单的情况下，一个DaemonSet，覆盖所有节点，会包含所有类型的守护进程。更复杂的设置的话，可以为每一种类型的守护进程设置多个DaemonSets，这些DaemonSets针对不同的硬件类型具有不同的标志和/或不同的内存和CPU请求。

##  编写DaemonSet Spec

---



