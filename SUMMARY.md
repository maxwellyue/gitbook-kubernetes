# Summary

* [介绍](README.md)
* [Kubernetes概述](chapter1.md)
  * [Kubernetes是什么](chapter1/kubernetesshi-shi-yao.md)
  * [Kubernetes组件](chapter1/kuberneteszu-jian.md)
  * [Kubernetes API](chapter1/kubernetes-api.md)
  * [Kubernetes 中的对象](chapter1/kubernetes-zhong-de-dui-xiang.md)
    * [理解Kubernetes对象](chapter1/kubernetes-zhong-de-dui-xiang/li-jiekubernetes-dui-xiang.md)
    * [命名Names](chapter1/kubernetes-zhong-de-dui-xiang/ming-ming-names.md)
    * [命名空间NameSpaces](chapter1/kubernetes-zhong-de-dui-xiang/ming-ming-kong-jian-namespaces.md)
    * [Labels和Selectors](chapter1/kubernetes-zhong-de-dui-xiang/labelshe-selectors.md)
    * [Annotations](chapter1/kubernetes-zhong-de-dui-xiang/annotations.md)
  * [使用kubectl进行对象管理](chapter1/shi-yong-kubectl-jin-xing-dui-xiang-guan-li.md)
    * [Kubernetes对象管理](chapter1/shi-yong-kubectl-jin-xing-dui-xiang-guan-li/kubernetesdui-xiang-guan-li.md)
    * [使用Imperative Commands管理Kubernetes对象](chapter1/shi-yong-kubectl-jin-xing-dui-xiang-guan-li/shi-yong-ming-ling-xing-guan-li-kubernetes-dui-xiang.md)
    * [使用Imperative object configuration管理Kubernetes对象](chapter1/shi-yong-kubectl-jin-xing-dui-xiang-guan-li/shi-yong-pei-zhi-wen-jian-guan-li-kubernetes-dui-xiang.md)
    * [使用Declarative object configuration管理Kubernetes对象](chapter1/shi-yong-kubectl-jin-xing-dui-xiang-guan-li/shi-yong-pei-zhi-wen-jian-dui-kubernetes-dui-xiang-jin-xing-sheng-ming-shi-guan-li.md)
* [Kubernetes架构](jia-gou-fen-jie.md)
  * [节点Nodes](jia-gou-fen-jie/nodes.md)
  * [Master节点通信](jia-gou-fen-jie/master-node-communication.md)
  * [Concepts Underlying the Cloud Controller Manager](jia-gou-fen-jie/concepts-underlying-the-cloud-controller-manager.md)
* [扩展Kubernetes](kuo-zhankubernetes.md)
* 容器
* [工作负载](gong-zuo-fu-zai.md)
  * [Pods](gong-zuo-fu-zai/pods.md)
    * [Pods综述](gong-zuo-fu-zai/pods/podszong-shu.md)
    * [Pods](gong-zuo-fu-zai/pods/pods.md)
    * Pods生命周期
    * Pods Preset
    * Disruptions
  * [Controllers](gong-zuo-fu-zai/controllers.md)
    * ReplicaSet
    * [ReplicationConroller](gong-zuo-fu-zai/controllers/replicationconroller.md)
    * [Deployments](gong-zuo-fu-zai/controllers/deployments.md)
    * [StatefulSets](gong-zuo-fu-zai/controllers/statefulsets.md)
    * [DaemonSet](gong-zuo-fu-zai/controllers/daemonset.md)
    * [Garbage Collection](gong-zuo-fu-zai/controllers/garbage-collection.md)
    * Jobs-Run to Completion
    * CronJob
* [配置](pei-zhi.md)
  * [Configuration最佳实践](pei-zhi/configurationzui-jia-shi-jian.md)
  * [管理容器的计算资源](pei-zhi/guan-li-rong-qi-de-ji-suan-zi-yuan.md)
  * 分配Pods到节点
  * [污点和容忍度](pei-zhi/wu-dian-he-rong-ren-du.md)
  * Secrets
* [服务、负载均衡和网络](fu-wu-3001-fu-zai-jun-heng-he-wang-luo.md)
  * [服务](fu-wu-3001-fu-zai-jun-heng-he-wang-luo/fu-wu.md)
  * [服务和Pods的DNS](fu-wu-3001-fu-zai-jun-heng-he-wang-luo/fu-wu-he-pods-de-dns.md)
  * 连接应用和服务
  * Ingress
  * 网络策略
  * 通过HostAliases为Pod/etc/hosts等添加entries
* 存储
* 集群管理
* [实践](shi-jian.md)

