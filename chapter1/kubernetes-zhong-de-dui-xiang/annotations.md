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

* 构建、发布或镜像的信息，如时间戳，发行ID，git分支，PR编号，镜像hashes和Registry地址等。

* Pointers to logging, monitoring, analytics, or audit repositories.

* 用来debug的客户依赖库或工具的信息，比如名称、 版本和构建信息。

* User or tool/system provenance information, such as URLs of related objects from other ecosystem components.

* 轻量级回滚工具的元数据，比如配置或检查点。

* 对该对象负责的人员的手机电话/寻呼机号码或者可以找到此类信息的地址信息，比如团队的web站点。

你也可以不使用annotations，而将这种类型的数据存储在外部数据库或目录中。但是这么做的话，会使得生成用于部署，管理，反思等的共享客户端库和工具变得困难。

