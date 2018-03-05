Kubernete是一个便利的、可扩展的开源平台，用于管理容器化的工作负载和服务，并为声明式配置和自动化提供便利。Kubernetes拥有庞大的、快速成长的生态系统。Kubernetes的相关服务、支持和工具非常方便。

谷歌在2014年将Kubernetes开源。Kubernetes是基于谷歌近15年的生产环境中大规模的工作负载运维经验之上，并融合了社区中最好的思想和实践。

# 为什么需要Kubernetes，它能做什么？

Kubernetes拥有很多特性，它可以认为是一个：

* 容器平台

* 微服务平台

* 便利的云平台等

Kubernetes提供了一套以容器为中心的管理环境。基于用户的工作负载情况，它可以对计算、网络、存储等进行编排。这就简化了PaaS平台的构建，并为IaaS平台的构建提供了伸缩性，同时也为跨基础设置服务商提供了便利。

# Kubernetes如何成为一个平台？

尽管Kubernetes提供了很多功能，但仍然有很多场景会从新特性中获益。特定应用的工作流程可以通过精简来加快开发速度。Ad hoc orchestration that is acceptable initially通常需要大规模地自动化。这就是为什么Kubernetes被设计为服务平台，它的组件和工具可以使应用程序更容易地进行部署、伸缩、管理。

Labels 可以让用户以他们想要的方式来组织他们的资源。Annotations 可以让用户为他们的资源设置一些通用信息，以此来改善工作流程并为检查点的状态管理工具提供了一种容易的方式。

另外，Kubernetes控制面板所基于的APIs，对开发者和用户都是一样的。用户可以编写自己的控制器，比如调度器（Schedulers）。

这样的设计也让很多其它系统可以构建于Kubernetes之上。

# Kubernetes不是什么

Kubernetes不是一个传统的，什么都包含的PaaS系统。由于Kubernetes是在容器层面而非硬件层面进行操作，它提供了一些与PaaS相同的功能，比如部署、伸缩、负载均衡、日志以及监控。但是，Kubernetes不是monolithic，这些默认的解决方案都是可选和插件化的。Kubernetes为提供了建立开发平台的基本组件，更重要的是，它给予用户选择的空间以及灵活性。

Kubernetes：

* 不限制应用的类型。Kubernetes致力于支持广泛的工作负载，包括有状态、无状态和数据处理工作负载。如果一个应用可以在容器中运行，那么它也可以在Kubernetes中运行良好。
* 不部署源代码，也不构建你的应用。CI/CD的工作流程取决于组织的文化和偏好以及技术要求。
* 不提供应用级别的服务，比如中间件（如消息总线等）、数据处理框架（比如Spark）、数据库（比如MySQL）、缓存以及集群存储系统（如Ceph），这些不会被当做内建的服务。这些组件可以运行在kubernetes中，也可以被运行在kubernetes中的应用通过向去访问。
* 不会提出日志、监控以及报警的解决方案，它会提供一些integrations as proof of concept, and mechanisms to collect and export metrics。
* 


