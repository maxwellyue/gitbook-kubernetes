英文地址：[https://kubernetes.io/docs/concepts/overview/working-with-objects/names/](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/)

所有的Kubernetes对象都会使用name和UID来清楚地标识。

对于非唯一性的用户提供的属性，Kubernetes提供了[labels](https://kubernetes.io/docs/user-guide/labels)和[annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)。

关于Name和UID的确切的语法规则，见[identifiers design doc](https://git.k8s.io/community/contributors/design-proposals/architecture/identifiers.md)。

# Names

---

Names是资源URL中客户端提供的一个字符串，指向一个对象，如：`/api/v1/pods/some-name`。

同一种kind下，对于特定的name，只能存在一个对象与之对应。但是，如果你删除了这个对象，你就可以创建一个与该对象同名的对象了。

按照惯例，Kubernetes对象的名字最多有253个字符，由小写字母、数字、`-`、`.`，但一些特殊的资源会有更多定义的限制。

# UID

---

UID是Kubernetes中系统自动生成用来唯一标识对象的字符串。每一个被创建的对象在Kubernetes集群中的整个生命周期中都有一个唯一的UID。UID被用来区分相似实体的历史版本。



