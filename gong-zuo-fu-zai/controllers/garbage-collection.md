英文地址：[https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/)

# 垃圾回收 

---

The role of the Kubernetes garbage collector is to delete certain objects that once had an owner, but no longer have an owner.

**Note**: Garbage collection is a beta feature and is enabled by default in Kubernetes version 1.4 and later.



## Owners and dependents {#owners-and-dependents}

Some Kubernetes objects are owners of other objects. For example, a ReplicaSet is the owner of a set of Pods. The owned objects are called_dependents_of the owner object. Every dependent object has a`metadata.ownerReferences`field that points to the owning object.

Sometimes, Kubernetes sets the value of`ownerReference`automatically. For example, when you create a ReplicaSet, Kubernetes automatically sets the`ownerReference`field of each Pod in the ReplicaSet. In 1.8, Kubernetes automatically sets the value of`ownerReference`for objects created or adopted by ReplicationController, ReplicaSet, StatefulSet, DaemonSet, Deployment, Job and CronJob.

You can also specify relationships between owners and dependents by manually setting the`ownerReference`field.

Here’s a configuration file for a ReplicaSet that has three Pods:

| [`my-repset.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/docs/concepts/workloads/controllers/my-repset.yaml) | ![](https://d33wubrfki0l68.cloudfront.net/951ae1fcc65e28202164b32c13fa7ae04fab4a0b/b77dc/images/copycode.svg) |
| :--- | :--- |
|  | apiVersion:extensions/v1beta1kind:ReplicaSetmetadata:name:my-repsetspec:replicas:3selector:matchLabels:pod-is-for:garbage-collection-exampletemplate:metadata:labels:pod-is-for:garbage-collection-examplespec:containers:-name:nginximage:nginx |

If you create the ReplicaSet and then view the Pod metadata, you can see OwnerReferences field:

```
kubectl create 
-f
 https://k8s.io/docs/concepts/controllers/my-repset.yaml
kubectl get pods 
--output
=
yaml

```

The output shows that the Pod owner is a ReplicaSet named my-repset:

```
apiVersion: v1
kind: Pod
metadata:
  ...
  ownerReferences:
  - apiVersion: extensions/v1beta1
    controller: 
true
    
blockOwnerDeletion: 
true
    
kind: ReplicaSet
    name: my-repset
    uid: d9607e19-f88f-11e6-a518-42010a800195
  ...

```

## Controlling how the garbage collector deletes dependents {#controlling-how-the-garbage-collector-deletes-dependents}

When you delete an object, you can specify whether the object’s dependents are also deleted automatically. Deleting dependents automatically is called_cascading deletion_. There are two modes of_cascading deletion_:_background_and_foreground_.

If you delete an object without deleting its dependents automatically, the dependents are said to be_orphaned_.

### Foreground cascading deletion {#foreground-cascading-deletion}

In_foreground cascading deletion_, the root object first enters a “deletion in progress” state. In the “deletion in progress” state, the following things are true:

* The object is still visible via the REST API
* The object’s
  `deletionTimestamp`
  is set
* The object’s
  `metadata.finalizers`
  contains the value “foregroundDeletion”.

Once the “deletion in progress” state is set, the garbage collector deletes the object’s dependents. Once the garbage collector has deleted all “blocking” dependents \(objects with`ownerReference.blockOwnerDeletion=true`\), it delete the owner object.

Note that in the “foregroundDeletion”, only dependents with`ownerReference.blockOwnerDeletion`block the deletion of the owner object. Kubernetes version 1.7 added an[admission controller](https://kubernetes.io/docs/admin/admission-controllers/#ownerreferencespermissionenforcement)that controls user access to set`blockOwnerDeletion`to true based on delete permissions on the owner object, so that unauthorized dependents cannot delay deletion of an owner object.

If an object’s`ownerReferences`field is set by a controller \(such as Deployment or ReplicaSet\), blockOwnerDeletion is set automatically and you do not need to manually modify this field.

### Background cascading deletion {#background-cascading-deletion}

In_background cascading deletion_, Kubernetes deletes the owner object immediately and the garbage collector then deletes the dependents in the background.

### Setting the cascading deletion policy {#setting-the-cascading-deletion-policy}

To control the cascading deletion policy, set the`propagationPolicy`field on the`deleteOptions`argument when deleting an Object. Possible values include “Orphan”, “Foreground”, or “Background”.

Prior to Kubernetes 1.9, the default garbage collection policy for many controller resources was`orphan`. This included ReplicationController, ReplicaSet, StatefulSet, DaemonSet, and Deployment. For kinds in the extensions/v1beta1, apps/v1beta1, and apps/v1beta2 group versions, unless you specify otherwise, dependent objects are orphaned by default. In Kubernetes 1.9, for all kinds in the apps/v1 group version, dependent objects are deleted by default.

Here’s an example that deletes dependents in background:

```
kubectl proxy 
--port
=
8080
curl 
-X
 DELETE localhost:8080/apis/extensions/v1beta1
amespaces/default/replicasets/my-repset 
\
-d
'{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Background"}'
\
-H
"Content-Type: application/json"
```

Here’s an example that deletes dependents in foreground:

```
kubectl proxy 
--port
=
8080
curl 
-X
 DELETE localhost:8080/apis/extensions/v1beta1
amespaces/default/replicasets/my-repset 
\
-d
'{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}'
\
-H
"Content-Type: application/json"
```

Here’s an example that orphans dependents:

```
kubectl proxy 
--port
=
8080
curl 
-X
 DELETE localhost:8080/apis/extensions/v1beta1
amespaces/default/replicasets/my-repset 
\
-d
'{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}'
\
-H
"Content-Type: application/json"
```

kubectl also supports cascading deletion. To delete dependents automatically using kubectl, set`--cascade`to true. To orphan dependents, set`--cascade`to false. The default value for`--cascade`is true.

Here’s an example that orphans the dependents of a ReplicaSet:

```
kubectl delete replicaset my-repset 
--cascade
=
false
```

### Additional note on Deployments {#additional-note-on-deployments}

When using cascading deletes with Deployments you_must_use`propagationPolicy: Foreground`to delete not only the ReplicaSets created, but also their Pods. If this type of_propagationPolicy_is not used, only the ReplicaSets will be deleted, and the Pods will be orphaned. See[kubeadm/\#149](https://github.com/kubernetes/kubeadm/issues/149#issuecomment-284766613)for more information.

  


