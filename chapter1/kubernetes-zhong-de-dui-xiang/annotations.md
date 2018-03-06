英文地址：[https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)

# Annotations

---

用户可以在使用kubernetes的Annotations将任意的非标识性的元数据附加到对象上。工具或依赖库等客户端可以获取到这些元数据。

## Attaching metadata to objects {#attaching-metadata-to-objects}

---

你可以使用Labels或者Annotations将一些元数据附加到Kubernetes中的对象上。Labels可以用来根据特定条件来筛选对象。相反，Annotations并不是用来定义和选择对象的。Annotations中的元数据可以很小或很大，可以是结构化数据也可以是非结构化数据，可以包含labels中不允许的字符。

与labels类似，annotations也是键值对：

```
"annotations": {
  "key1" : "value1",
  "key2" : "value2"
}
```

下面是一些可以在annotations中记录的信息样例：

* Fields managed by a declarative configuration layer. Attaching these fields as annotations distinguishes them from default values set by clients or servers, and from auto-generated fields and fields set by auto-sizing or auto-scaling systems.

* Build, release, or image information like timestamps, release IDs, git branch, PR numbers, image hashes, and registry address.

* Pointers to logging, monitoring, analytics, or audit repositories.

* Client library or tool information that can be used for debugging purposes: for example, name, version, and build information.

* User or tool/system provenance information, such as URLs of related objects from other ecosystem components.

* Lightweight rollout tool metadata: for example, config or checkpoints.

* Phone or pager numbers of persons responsible, or directory entries that specify where that information can be found, such as a team web site.

你也可以不使用annotations，而将这种类型的数据存储在外部数据库或目录中。但是这么做的话，会使得生成用于部署，管理，反思等的共享客户端库和工具变得困难。



