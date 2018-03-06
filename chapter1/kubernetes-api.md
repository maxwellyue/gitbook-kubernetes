英文原文：[https://kubernetes.io/docs/concepts/overview/kubernetes-api/](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)

# The Kubernetes API

总体的API约定描述见[API conventions doc](https://git.k8s.io/community/contributors/devel/api-conventions.md)。

关于API中终端、资源类型和样例的描述见[API Reference](https://kubernetes.io/docs/reference)。

关于远程接入API的讨论见[access doc](https://kubernetes.io/docs/admin/accessing-the-api)。

Kubernetes API是系统中声明式配置模式的基础。命令行工具kubelet可以被用来创建、更新、删除、获取API对象。

Kubernetes 同时也将API的资源以序列化的方式进行了存储，目前存储在etcd中。

Kubernetes本身是由多个组件构成的，这些组件之间的交互都是通过它的API。

# API 变化

---

从我们的经验来看，任何一个成功的系统都需要不断成长，不断改进，因为总会有新的用例出现或者旧的用例发生改变。因此，我们期望Kubernetes API 可以持续改进、不断成长。但我们会在很长的一段时间内保持现有客户端与Kubernetes API 之间的兼容性。总体来看，新的API资源和新的资源领域可以很频繁地添加进来。而减少API资源或领域则需要遵循[API deprecation policy](https://kubernetes.io/docs/reference/deprecation-policy/)。

关于“什么是一个兼容的改进”和“如何改进API”的细节见[API change document](https://git.k8s.io/community/contributors/devel/api_changes.md)。

## OpenAPI 和 Swagger 定义 {#openapi-and-swagger-definitions}

---

完整的API使用了[Swagger v1.2](http://swagger.io/)和[OpenAPI](https://www.openapis.org/)进行了文档化。Kubernetes 的apiserver向外暴露了用来获取Swagger v1.2版本的 Kubernetes API的接口，位置在`/swaggerapi`。你也可以通过给apiserver设置`--enable-swagger-ui=true`来通过`/swagger-ui`这个路径在UI界面中浏览API。

从Kubernetes 1.4版本开始，开始提供了OpenAPI 规范，路径在[`/swagger.json`](https://git.k8s.io/kubernetes/api/openapi-spec/swagger.json)。 While we are transitioning from Swagger v1.2 to OpenAPI \(aka Swagger v2.0\), some of the tools such as kubectl and swagger-ui are still using v1.2 spec. OpenAPI spec is in Beta as of Kubernetes 1.5.

Kubernetes implements an alternative Protobuf based serialization format for the API that is primarily intended for intra-cluster communication, documented in the[design proposal](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/protobuf.md)and the IDL files for each schema are located in the Go packages that define the API objects.

## API versioning {#api-versioning}

To make it easier to eliminate fields or restructure resource representations, Kubernetes supports multiple API versions, each at a different API path, such as`/api/v1`or`/apis/extensions/v1beta1`.

We chose to version at the API level rather than at the resource or field level to ensure that the API presents a clear, consistent view of system resources and behavior, and to enable controlling access to end-of-lifed and/or experimental APIs. The JSON and Protobuf serialization schemas follow the same guidelines for schema changes - all descriptions below cover both formats.

Note that API versioning and Software versioning are only indirectly related. The[API and release versioning proposal](https://git.k8s.io/community/contributors/design-proposals/release/versioning.md)describes the relationship between API versioning and software versioning.

Different API versions imply different levels of stability and support. The criteria for each level are described in more detail in the[API Changes documentation](https://git.k8s.io/community/contributors/devel/api_changes.md#alpha-beta-and-stable-versions). They are summarized here:

* Alpha level:
  * The version names contain
    `alpha`
    \(e.g.
    `v1alpha1`
    \).
  * May be buggy. Enabling the feature may expose bugs. Disabled by default.
  * Support for feature may be dropped at any time without notice.
  * The API may change in incompatible ways in a later software release without notice.
  * Recommended for use only in short-lived testing clusters, due to increased risk of bugs and lack of long-term support.
* Beta level:
  * The version names contain
    `beta`
    \(e.g.
    `v2beta3`
    \).
  * Code is well tested. Enabling the feature is considered safe. Enabled by default.
  * Support for the overall feature will not be dropped, though details may change.
  * The schema and/or semantics of objects may change in incompatible ways in a subsequent beta or stable release. When this happens, we will provide instructions for migrating to the next version. This may require deleting, editing, and re-creating API objects. The editing process may require some thought. This may require downtime for applications that rely on the feature.
  * Recommended for only non-business-critical uses because of potential for incompatible changes in subsequent releases. If you have multiple clusters which can be upgraded independently, you may be able to relax this restriction.
  * **Please do try our beta features and give feedback on them! Once they exit beta, it may not be practical for us to make more changes.**
* Stable level:
  * The version name is
    `vX`
    where
    `X`
    is an integer.
  * Stable versions of features will appear in released software for many subsequent versions.

## API groups {#api-groups}

To make it easier to extend the Kubernetes API, we implemented[_API groups_](https://git.k8s.io/community/contributors/design-proposals/api-machinery/api-group.md). The API group is specified in a REST path and in the`apiVersion`field of a serialized object.

Currently there are several API groups in use:

1. The_core_group, often referred to as the_legacy group_, is at the REST path`/api/v1`and uses`apiVersion: v1`.

2. The named groups are at REST path`/apis/$GROUP_NAME/$VERSION`, and use`apiVersion: $GROUP_NAME/$VERSION`\(e.g.`apiVersion: batch/v1`\). Full list of supported API groups can be seen in[Kubernetes API reference](https://kubernetes.io/docs/reference/).

There are two supported paths to extending the API with[custom resources](https://kubernetes.io/docs/concepts/api-extension/custom-resources/):

1. [CustomResourceDefinition](https://kubernetes.io/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/)
   is for users with very basic CRUD needs.
2. Coming soon: users needing the full set of Kubernetes API semantics can implement their own apiserver and use the
   [aggregator](https://git.k8s.io/community/contributors/design-proposals/api-machinery/aggregated-api-servers.md)
   to make it seamless for clients.

## Enabling API groups {#enabling-api-groups}

Certain resources and API groups are enabled by default. They can be enabled or disabled by setting`--runtime-config`on apiserver.`--runtime-config`accepts comma separated values. For ex: to disable batch/v1, set`--runtime-config=batch/v1=false`, to enable batch/v2alpha1, set`--runtime-config=batch/v2alpha1`. The flag accepts comma separated set of key=value pairs describing runtime configuration of the apiserver.

IMPORTANT: Enabling or disabling groups or resources requires restarting apiserver and controller-manager to pick up the`--runtime-config`changes.

## Enabling resources in the groups {#enabling-resources-in-the-groups}

DaemonSets, Deployments, HorizontalPodAutoscalers, Ingress, Jobs and ReplicaSets are enabled by default. Other extensions resources can be enabled by setting`--runtime-config`on apiserver.`--runtime-config`accepts comma separated values. For example: to disable deployments and ingress, set`--runtime-config=extensions/v1beta1/deployments=false,extensions/v1beta1/ingress=false`

  


  


