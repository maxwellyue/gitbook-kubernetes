英文原文：[https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)

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

* Does not provide nor mandate a configuration language/system \(e.g.,[jsonnet](https://github.com/google/jsonnet)\). It provides a declarative API that may be targeted by arbitrary forms of declarative specifications.

* Does not provide nor adopt any comprehensive machine configuration, maintenance, management, or self-healing systems.

另外，Kubernetes不仅仅是一个编排系统。实际上，它会消除编排的需求。“编排”这个技术的定义是一个定义好的工作流执行顺序：先A，后B，再C。作为比较，Kubernetes是由一系列独立的、可组合的控制程序组成的，这些控制程序会持续地将（用户的应用）现在的状态转变为期望的状态。你如何从A到C，kubernetes并不关心。Kubernetes也不要求中心控制。这样就可以得到一个更容易去使用、强健、可伸缩、可扩展的系统。

# 为什么要使用容器

还在寻找应该使用容器的理由？请看下图：![](/assets/屏幕快照 2018-03-05 下午10.07.25.png)The Old Way：使用这方式部署应用其实就是在主机上使用操作系统的包管理工具安装一个程序。这样做的缺点是应用程序的执行、配置、依赖库、生命周期耦合在一起，并紧紧与主机的操作系统绑定。你也许可以会建立一个固定的VM镜像来进行滚动升级和回滚，但VM是重量级的，且并非是便利的。

The New Way：这种方式其实是部署一个基于操作系统而非硬件的虚拟化容器。这些容器彼此隔离，并与主机隔离：它们有自己的文件系统，它们看不到其他的进程，它们的计算资源的使用可以进行限定。容器比VM更容易比构建，并且由于容器与底层基础设及文件系统是解耦的，它们可以跨云、跨OS。

由于容器小、快，应用可以打包在容器的镜像中。这种“一应用一容器”的关系可以最大化容器的功效。使用容器，容器镜像可以在构建/发布的时候进行创建，而不是在部署的时候，因为应用不必与和应用相关的其他栈相组合，也不必和生产环境相组合。在构建/发布阶段进行创建容器镜像可以在开发环境和生产环境中保持一致的环境。同样，容器也比VM更容易传输，这给有助于监控和管理。当容器进程的生命周期由基础设施而非隐藏在容器中的监控进程管理时，这一点尤其突出。最后，使用“一应用一容器”的方式的话，管理容器就是在管理应用的部署。

容器的优点可总结如下：

* 敏捷的应用创建和部署：相比VM，容器镜像的创建更加高效。
* 持续开发、集成和部署：提供可靠的、更快的容器镜像构建和部署，以及快速、简单的回滚。
* Dev and Ops separation of concerns: Create application container images at build/release time rather than deployment time, thereby decoupling applications from infrastructure.
* Observability Not only surfaces OS-level information and metrics, but also application health and other signals.
* Environmental consistency across development, testing, and production: Runs the same on a laptop as it does in the cloud.
* Cloud and OS distribution portability: Runs on Ubuntu, RHEL, CoreOS, on-prem, Google Kubernetes Engine, and anywhere else.
* Application-centric management: Raises the level of abstraction from running an OS on virtual hardware to run an application on an OS using logical resources.
* Loosely coupled, distributed, elastic, liberated micro-services: Applications are broken into smaller, independent pieces and can be deployed and managed dynamically – not a fat monolithic stack running on one big single-purpose machine.
* Resource isolation: Predictable application performance.
* Resource utilization: High efficiency and density.

# Kubernetes的字面意思是啥？K8s又是啥？

Kubernetes这个名字起源于希腊语, 是“舵手”或者“领航员”的意思，同时也是“管理者”和“控制论”这两个词的的词根。

K8s是把Kubernetes中间的8个字符“ubernete”使用数字而代替，是Kubernetes的缩写形式。

