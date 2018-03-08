英文地址：[https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)

## 理解Pods {#understanding-pods}

---

Pod是Kubernetes的\]基础构件，是你在Kubernetes对象模型中所能创建或部署的最小、最简单的单元。一个Pod表示集群中运行的一个程序。

Pod中包含了应用容器的封装（或者，在某些情况下，多种容器）、存储资源、唯一的网络IP和控制容器运行的选项。一个Pod表示了一个部署单元：Kubernetes中单一的应用实例，其中可能包含一个或多个仅仅耦合并共享资源的容器。

> [Docker](https://www.docker.com/) 是Kubernetes Pod中最常见的容器运行时，但是Pods也支持其他容器运行时。

kubernetes集群中，Pods主要用在两个方面：

* 运行单一容器  

 “一容器一Pod”的模型在Kubernetes中是最常见的。此时，你可以将Pod看成是单一容器的包装，Kubernetes对Pods进行管理，而不会直接管理容器。

* 运行需要工作在一起的多个容器

Pods in a Kubernetes cluster can be used in two main ways:

* **Pods that run a single container**
  . The “one-container-per-Pod” model is the most common Kubernetes use case; in this case, you can think of a Pod as a wrapper around a single container, and Kubernetes manages the Pods rather than the containers directly.
* **Pods that run multiple containers that need to work together**
  . A Pod might encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. These co-located containers might form a single cohesive unit of service–one container serving files from a shared volume to the public, while a separate “sidecar” container refreshes or updates those files. The Pod wraps these containers and storage resources together as a single manageable entity.

The[Kubernetes Blog](http://blog.kubernetes.io/)has some additional information on Pod use cases. For more information, see:

* [The Distributed System Toolkit: Patterns for Composite Containers](http://blog.kubernetes.io/2015/06/the-distributed-system-toolkit-patterns.html)
* [Container Design Patterns](http://blog.kubernetes.io/2016/06/container-design-patterns.html)

Each Pod is meant to run a single instance of a given application. If you want to scale your application horizontally \(e.g., run multiple instances\), you should use multiple Pods, one for each instance. In Kubernetes, this is generally referred to as_replication_. Replicated Pods are usually created and managed as a group by an abstraction called a Controller. See[Pods and Controllers](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pods-and-controllers)for more information.

### How Pods manage multiple Containers {#how-pods-manage-multiple-containers}

Pods are designed to support multiple cooperating processes \(as containers\) that form a cohesive unit of service. The containers in a Pod are automatically co-located and co-scheduled on the same physical or virtual machine in the cluster. The containers can share resources and dependencies, communicate with one another, and coordinate when and how they are terminated.

Note that grouping multiple co-located and co-managed containers in a single Pod is a relatively advanced use case. You should use this pattern only in specific instances in which your containers are tightly coupled. For example, you might have a container that acts as a web server for files in a shared volume, and a separate “sidecar” container that updates those files from a remote source, as in the following diagram:

![](https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg "pod diagram")

Pods provide two kinds of shared resources for their constituent containers:_networking\_and\_storage_.

#### Networking {#networking}

Each Pod is assigned a unique IP address. Every container in a Pod shares the network namespace, including the IP address and network ports. Containers_inside a Pod\_can communicate with one another using_`localhost`_. When containers in a Pod communicate with entities\_outside the Pod_, they must coordinate how they use the shared network resources \(such as ports\).

#### Storage {#storage}

A Pod can specify a set of shared storage_volumes_. All containers in the Pod can access the shared volumes, allowing those containers to share data. Volumes also allow persistent data in a Pod to survive in case one of the containers within needs to be restarted. See[Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)for more information on how Kubernetes implements shared storage in a Pod.

## Working with Pods {#working-with-pods}

You’ll rarely create individual Pods directly in Kubernetes–even singleton Pods. This is because Pods are designed as relatively ephemeral, disposable entities. When a Pod gets created \(directly by you, or indirectly by a Controller\), it is scheduled to run on a Node in your cluster. The Pod remains on that Node until the process is terminated, the pod object is deleted, the pod is\_evicted\_for lack of resources, or the Node fails.

**Note:**Restarting a container in a Pod should not be confused with restarting the Pod. The Pod itself does not run, but is an environment the containers run in and persists until it is deleted.

Pods do not, by themselves, self-heal. If a Pod is scheduled to a Node that fails, or if the scheduling operation itself fails, the Pod is deleted; likewise, a Pod won’t survive an eviction due to a lack of resources or Node maintenance. Kubernetes uses a higher-level abstraction, called a_Controller_, that handles the work of managing the relatively disposable Pod instances. Thus, while it is possible to use Pod directly, it’s far more common in Kubernetes to manage your pods using a Controller. See[Pods and Controllers](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pods-and-controllers)for more information on how Kubernetes uses Controllers to implement Pod scaling and healing.

### Pods and Controllers {#pods-and-controllers}

A Controller can create and manage multiple Pods for you, handling replication and rollout and providing self-healing capabilities at cluster scope. For example, if a Node fails, the Controller might automatically replace the Pod by scheduling an identical replacement on a different Node.

Some examples of Controllers that contain one or more pods include:

* [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
* [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

In general, Controllers use a Pod Template that you provide to create the Pods for which it is responsible.

## Pod Templates {#pod-templates}

Pod templates are pod specifications which are included in other objects, such as[Replication Controllers](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/),[Jobs](https://kubernetes.io/docs/concepts/jobs/run-to-completion-finite-workloads/), and[DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/). Controllers use Pod Templates to make actual pods. The sample below is a simple manifest for a Pod which contains a container that prints a message.

