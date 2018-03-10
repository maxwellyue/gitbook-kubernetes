英文地址：[https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

# Deployments

---

Deployments为[Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)和[ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)提供声明性的更新（declarative updates）。

你在Deployment对象中描述一个期望状态，Deployment controller就会以可以控制的速率将实际状态转换为期望状态。你可以通过定义Deployments来创建新的ReplicaSets，或者来删除已存在的Deployments并adopt all their resources with new Deployments。

> 注意：You你不应该去管理被一个Deployment所拥有的ReplicaSets。所有的操作都应该通过操作Deployment对象来完成。

## 用例

---

以下是Deployment的典型用例：

* 创建一个Deployment来rollout一个ReplicaSet。ReplicaSet会在后台创建Pods。通过检查rollout的状态来判断是否成功。

* 通过更新Deployment的PodTemplateSpec来声明Pods的新状态。这样，就会创建一个新的ReplicaSet，Deployment会以可控的速率将Pods从旧ReplicaSet转移到这个新的ReplicaSet中。每一次新的ReplicaSet都会更新Deployment的版本。

* 如果当前Deployment的状态不稳定，可以回滚到一个之前的Deployment版本中。每一次回滚都会更新Deployment的版本。

* 扩展（scale up）Deployment来应对更多负载。

* 暂停一个Deployment来应用多个PodTemplateSpec修复，并恢复Deployment来开启一个新的rollout。

* 使用Deployment的状态来作为判断rollout是否卡住的指示器。

* 清除你不再需要的旧的ReplicaSet。

## 创建Deployment

---

下面是Deployment的一个示例。它会创建一个ReplicaSet来运行3个nginx Pods。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

在这个示例中：

* 创建了一个名称为`nginx-deployment`的Deployment，通过`metadata: name`字段来指定。
* 这个Deployment会创建3个Pods副本，通过`replicas`字段来指定。
* `selector`字段定义了该Deployment如何找到其管理的Pods。在这个例子中，我们使用定义在Pod模板中的一个标签（`app:nginx`）。只要Pod模板自身符合规则，也可以使用更复杂的选择器规则。
* Pod模板的定义部分，即`template: spec`字段，指定Pod会运行一个`nginx`容器，该容器使用`nginx`1.7.9版本的镜像。
* 该Deployment会开放80端口供Pods使用。 

> Note: `matchLabels`is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of `matchExpressions`, whose key field is “key”, the operator is “In”, and the values array contains only “value”. The requirements are ANDed.

`template`字段包含以下说明：

* 这些Pods被打上`app: nginx`标签。
* 创建一个名为`nginx`的容器。
* `nginx`的镜像版本是`1.7.9`。
* 开放`80`端口，以便该容器进行发生和接收数据。

运行以下命令来创建该Deployment：

```
kubectl create -f https://raw.githubusercontent.com/kubernetes/website/master/docs/concepts/workloads/controllers/nginx-deployment.yaml
```

> **Note:**  
> 你可以在上述命令中使用`--record`将当前命令注释在创建或删除资源的注释中。这一点在之后的审查中非常有用，比如查看在每一个Deployment版本中执行了什么命令。

现在，运行`kubectl get deployments` 命令，将会输出类似如下的结果：

```
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         0         0            0           1s
```

当你查看集群中的Deployment时，会列出以下信息：

* `NAME`是指集群中的Deployments 的名字。
* `DESIRED`是指你期望的应用的_replicas_数量，你可以在创建Deployment的时候定义。 这是期望状态。
* `CURRENT`是指当前有多少个replicas在运行。
* `UP-TO-DATE`是指已经更新到期望状态的replicas的数量。
* `AVAILABLE`是指有多少replicas可用。
* `AGE`是指应用已经运行了多长时间。

你应该知道Deployment的定义中的值与上面这些值之间的对应关系：

* 期望的3个replicas与`spec: replicas`字段对应。
* 当前replicas数量0与`.status.replicas`字段对应。
* 已经更细到期望状态的replicas数量 0与`.status.updatedReplicas`字段对应。
* 可用replicas数量0与`.status.availableReplicas`字段对应。

运行`kubectl rollout status deployment/nginx-deployment`. 命令来查看该Deployment的rollout状态，该命令会返回如下结果：

```
Waiting  for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```

几秒之后再次运行`kubectl get deployments`命令：

```
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           18s
```

此时，该Deployment已经创建了3个replicas，并且所有的replicas都与期望状态一致且是可用的（Pod的状态态在`.spec.minReadySeconds`时间内都是Ready\)。

运行`kubectl get rs`命令来查看该Deployment创建的ReplicaSet \(`rs`\)：

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-2035384211   3         3         3       18s
```

可以看出，ReplicaSet的名字的格式是：`[DEPLOYMENT-NAME]-[POD-TEMPLATE-HASH-VALUE]`。该哈希值是Deployment创建时自动生成的。

运行`kubectl get pods --show-labels`命令来查看为每个Pod自动生成的标签：

```
NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-2035384211-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
nginx-deployment-2035384211-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
nginx-deployment-2035384211-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
```

被创建的ReplicaSet用来确保在任意时刻都有3个`nginx`Pods在运行。

> **Note:**You你必须在Deployment中定义一个恰当的选择器和标签\(在这个例子中是`app: nginx`\)。不要使用其controllers（或其他Deployments和StatefulSets）去覆盖这些labels 或者 selectors。Kubernetes不会阻止你去干这样的事情，如果多个controllers覆盖了这些选择器，那么这些controllers可能会彼此冲突，并出现不可预料的行为。

##### Pod-template-hash 标签

> 不要修改这个标签。

这个`pod-template-hash`标签是由Deployment controller自动添加到每一个ReplicaSet中的。

该标签会确保一个Deployment的ReplicaSets不会重复。它是通过对ReplicaSet中的`PodTemplate`进行哈希计算得到的，并将计算结果添加到ReplicaSet的选择器、Pod模板的标签以及该ReplicaSet中存在的所有Pods中。

## 更新Deployment

---

一个Deployment的rollout只会在它的pod模板被改动的时候触发。比如，模板的标签或者容器镜像被更新。其他更新，比如scaling一个Deployment，不会触发一次rollout。

假如现在我们想将nginx Pods 的镜像用`nginx:1.9.1`镜像替代原来的`nginx:1.7.9`镜像。

```
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
deployment "nginx-deployment" image updated
```

此外，我们可以 `edit`该Deployment 并将`.spec.template.spec.containers[0].image`

的值从`nginx:1.7.9`修改为`nginx:1.9.1`：

```
$ kubectl edit deployment/nginx-deployment
deployment "nginx-deployment" edited
```

运行以下命令来查看rollout的状态：

```
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```

在成功rollout后，可以运行以下命令获取该Deployment：

```
$ kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           36s
```

可以看出UP-TO-DATA的replicas数量为3，说明Deployment已经将replicas更新到最新配置。CURRENT的replicas的数量为3，说明该Deployment管理的replicas的总数为3，AVAILABLE说明那当前可用replicas的数量为3。

我们可以运行`kubectl get rs`命令来查看此次Deployment更新Pods的方式：创建了一个新的ReplicaSet且将其扩展到3个replicas，并将旧的ReplicaSet中的replicas数量缩小到0个。

```
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         3       6s
nginx-deployment-2035384211   0         0         0       36s
```

运行`get pods`会列出新的Pods：

```
$ kubectl get pods
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-1564180365-khku8   1/1       Running   0          14s
nginx-deployment-1564180365-nacti   1/1       Running   0          14s
nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
```

下次我们想更新这些Pods的时候，只需要再次更新该Deployment的pod模板即可。

Deployment可以确保在升级过程中，只会有一定数量的Pods会关闭。By default, it ensures that at least 25% less than the desired number of Pods are up \(25% max unavailable\).

Deployment can also ensure that only a certain number of Pods may be created above the desired number of Pods. By default, it ensures that at most 25% more than the desired number of Pods are up \(25% max surge\).

比如，当你仔细观察上面的Deployment的时候，你会发现，它先会创建一个新的Pod，然后删除一些旧的Pods并创建新的Pods。它不会将旧的Pods杀死，直到足够数量的新Pods被启动；它也不会创建新的Pods，直到一定数量的旧的Pods被杀死。在整个过程中，它会确保可用Pods至少是2，Pods总数最多是4。

```
$ kubectl describe deployments
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=2
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.9.1
    Port:         80/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
  Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
  Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
  Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
  Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
  Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
  Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
```

可以看到，当我们首次创建Deployment的时候，它创建了一个ReplicaSet \(nginx-deployment-2035384211\)，然后直接将replicas数量增大到3个。当我们更新该Deployment的时候，它会创建一个新的ReplicaSet \(nginx-deployment-1564180365\)，并增大副本到1，然后将旧的副本集中的副本的数量减小到2，因此至少2个Pods可用，同一时刻最多4个Pods。然后，它会以相同策略持续增大新副本集中副本数量，减小旧副本集中副本数量。最终，我们会的得到拥有3个副本的副本集，旧的副本集中的副本数量会减小到0。

### Rollover \(又叫 multiple updates in-flight\) {#rollover-aka-multiple-updates-in-flight}

每当Deployment controller观察到有新的Deployment对象，就会创建一个新的ReplicaSet来启动期望的Pods。已经存在的 ReplicaSet会控制那些标签匹配`.spec.selector`的Pods，那些模板不匹配`.spec.template`的Pods就被scaled down。最终买新的ReplicaSet 将会scaled to`.spec.replicas`，所有旧的ReplicaSets 将会scaled to 0.

如果你在一次rollout正在执行的过程中对Deployment进行更新操作，该Deployment将会为每次更新创建一个新的ReplicaSet，将其 scaling，并会roll over之前的ReplicaSet。它会将旧的ReplicaSet添加到旧ReplicaSets列表，并将其scaling down。

比如，假如你创建了一个Deployment 来创建5 个`nginx:1.7.9`副本，但是当只有3个`nginx:1.7.9`副本被创建时，你更新了该Deployment 来让它创建5个`nginx:1.9.1`, 副本。在这种情况下，Deployment 会立即开始杀死这3个已经创建的`nginx:1.7.9`Pods，并开始创建`nginx:1.9.1`Pods。它不会等到这5个`nginx:1.7.9`Pods都创建好之后再去改变。

### 更新标签选择器 {#label-selector-updates}

通常来说，并不建议对标签选择器进行更新，应该在一开始就规划好标签选择器。在任何情况下，如果你需要进行标签选择器的更新，都要谨慎行事，并确保你掌握了所有的含义。

> **Note:**在 API `apps/v1`版本中，一个Deployment的标签选择器一旦创建就不可改变。

* Selector additions require the pod template labels in the Deployment spec to be updated with the new label too, otherwise a validation error is returned. This change is a non-overlapping one, meaning that the new selector does not select ReplicaSets and Pods created with the old selector, resulting in orphaning all old ReplicaSets and creating a new ReplicaSet.
* Selector updates – that is, changing the existing value in a selector key – result in the same behavior as additions.
* Selector removals – that is, removing an existing key from the Deployment selector – do not require any changes in the pod template labels. No existing ReplicaSet is orphaned, and a new ReplicaSet is not created, but note that the removed label still exists in any existing Pods and ReplicaSets.

## 回滚Deployment {#rolling-back-a-deployment}

---

有时，你需要回滚一个Deployment；比如，当一个Deployment不稳定时，比如一直崩溃。默认情况下，所有的Deployment的rollout历史都被保存在系统中，所以你在可以在任何时间进行回滚 \(你可以通过修改历史版本限制来改变保存多少历史版本\)。

> **Note:**A 当触发Deployment的rollout时，Deployment的revision就会被创建。这意味着，新的revision当且仅当在该Deployment的 pod模板\(`.spec.template`\) 被修改的时候才会被创建。比如你修改了模板中的标签或容器镜像。其他修改，比如scaling该Deployment，不会创建一个Deployment revision，所以我们可以facilitate simultaneous manual- or auto-scaling。这意味着，当你回滚到较早版本的时候，只有Deployment的pod模板部分被回滚。

假如你在更新Deployment的时候，镜像名字应该是`nginx:1.9.1`，你却写成了`nginx:1.91`：

```
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.91
deployment "nginx-deployment" image updated
```

那么这次rollout就会被卡住：

```
$ kubectl rollout status deployments nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
```

按Ctrl-C来停止对rollout状态的观察。更多rollouts卡住的信息请[read more here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#deployment-status)。

你可以看到新（nginx-deployment-3066724191）、旧（nginx-deployment-1564180365 和 nginx-deployment-2035384211）副本集的数量都是2。

```
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   2         2         0       25s
nginx-deployment-2035384211   0         0         0       36s
nginx-deployment-3066724191   2         2         2       6s
```

查看被创建的Pods，你会发现被新ReplicaSet创建的2个pods被卡在拉取镜像这一步。

```
$ kubectl get pods
NAME                                READY     STATUS             RESTARTS   AGE
nginx-deployment-1564180365-70iae   1/1       Running            0          25s
nginx-deployment-1564180365-jbqqo   1/1       Running            0          25s
nginx-deployment-3066724191-08mng   0/1       ImagePullBackOff   0          6s
nginx-deployment-3066724191-eocby   0/1       ImagePullBackOff   0          6s
```

**Note:**Deployment controller将会自动停止bad rollout，并会停止scaling up新的ReplicaSet。这取决于你指定的rollingUpdate 参数\(`maxUnavailable`\) . Kubernetes默认将其设置为1，并将`spec.replicas`设置为1，所以如果你不关心这些参数，你的Deployment默认会100%不可用！在未来版本中，Kubernetes 将会修复这点。

```
$ kubectl describe deployment
Name:           nginx-deployment
Namespace:      default
CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
Labels:         app=nginx
Selector:       app=nginx
Replicas:       2 updated | 3 total | 2 available | 2 unavailable
StrategyType:       RollingUpdate
MinReadySeconds:    0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
OldReplicaSets:     nginx-deployment-1564180365 (2/2 replicas created)
NewReplicaSet:      nginx-deployment-3066724191 (2/2 replicas created)
Events:
  FirstSeen LastSeen    Count   From                    SubobjectPath   Type        Reason              Message
  --------- --------    -----   ----                    -------------   --------    ------              -------
  1m        1m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
  21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
  21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-1564180365 to 2
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 2
```

为了修复这种rollout失败，我们需要将回滚到一个稳定的Deployment的revision。

##### 检查Deployment的Rollout历史

首先，查看这个deployment的修改历史：

```
$ kubectl rollout history deployment/nginx-deployment
deployments "nginx-deployment"
REVISION    CHANGE-CAUSE
1           kubectl create -f docs/user-guide/nginx-deployment.yaml --record
2           kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
3           kubectl set image deployment/nginx-deployment nginx=nginx:1.91
```

因为我们在创建该deployment的时候添加了`--record`参数，所以我们可以很容易地看到每次的修改。

如果要查看某个REVISION的具体信息，可以运行：

```
$ kubectl rollout history deployment/nginx-deployment --revision=2
deployments "nginx-deployment" revision 2
  Labels:       app=nginx
          pod-template-hash=1159050644
  Annotations:  kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
  Containers:
   nginx:
    Image:      nginx:1.9.1
    Port:       80/TCP
     QoS Tier:
        cpu:      BestEffort
        memory:   BestEffort
    Environment Variables:      <none>
  No volumes.
```

##### 回滚到之前的Revision

现在，我们决定撤销当前的rollout，并回滚到之前的revision：

```
$ kubectl rollout undo deployment/nginx-deployment
deployment "nginx-deployment" rolled back
```

另外，你也可以通过定义来`--to-revision`来指定回滚的版本：

```
$ kubectl rollout undo deployment/nginx-deployment --to-revision=2
deployment "nginx-deployment" rolled back
```

更多rollout相关的命令，请阅读[`kubectl rollout`](https://kubernetes.io/docs/user-guide/kubectl/v1.9/#rollout)。

现在，该Deployment就回滚到了之前的稳定一个revision。Deployment controller会生成一个回滚到之前revision 2的`DeploymentRollback`事件。

```
$ kubectl get deployment
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           30m

$ kubectl describe deployment
Name:           nginx-deployment
Namespace:      default
CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
Labels:         app=nginx
Selector:       app=nginx
Replicas:       3 updated | 3 total | 3 available | 0 unavailable
StrategyType:       RollingUpdate
MinReadySeconds:    0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
OldReplicaSets:     <none>
NewReplicaSet:      nginx-deployment-1564180365 (3/3 replicas created)
Events:
  FirstSeen LastSeen    Count   From                    SubobjectPath   Type        Reason              Message
  --------- --------    -----   ----                    -------------   --------    ------              -------
  30m       30m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 2
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-1564180365 to 2
  2m        2m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-3066724191 to 0
  2m        2m          1       {deployment-controller }                Normal      DeploymentRollback  Rolled back deployment "nginx-deployment" to revision 2
  29m       2m          2       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
```

## 伸缩Deployment {#scaling-a-deployment}

---

你可以通过如下命令来对一个Deployment进行伸缩：

```
$ kubectl scale deployment nginx-deployment --replicas=10
deployment "nginx-deployment" scaled
```

假如你的集群中开启了[horizontal pod autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)，那么你可以为Deployment设置自动伸缩器，并基于CPU利用率来选择最小和最大的Pods数。

```
$ kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80
deployment "nginx-deployment" autoscaled
```

### Proportional scaling {#proportional-scaling}

RollingUpdate Deployments support running multiple versions of an application at the same time. When you or an autoscaler scales a RollingUpdate Deployment that is in the middle of a rollout \(either in progress or paused\), then the Deployment controller will balance the additional replicas in the existing active ReplicaSets \(ReplicaSets with Pods\) in order to mitigate risk. This is called_proportional scaling_.

For example, you are running a Deployment with 10 replicas,[maxSurge](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge)=3, and[maxUnavailable](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable)=2.

```
$ 
kubectl get deploy
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     10        10        10           10          50s
```

You update to a new image which happens to be unresolvable from inside the cluster.

```
$ kubectl set image deploy nginx-deployment 
nginx
=
nginx:sometag
deployment 
"nginx-deployment"
 image updated
```

The image update starts a new rollout with ReplicaSet nginx-deployment-1989198191, but it’s blocked due to the`maxUnavailable`requirement that we mentioned above.

```
$ 
kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   5         5         0         9s
nginx-deployment-618515232    8         8         8         1m
```

Then a new scaling request for the Deployment comes along. The autoscaler increments the Deployment replicas to 15. The Deployment controller needs to decide where to add these new 5 replicas. If we weren’t using proportional scaling, all 5 of them would be added in the new ReplicaSet. With proportional scaling, we spread the additional replicas across all ReplicaSets. Bigger proportions go to the ReplicaSets with the most replicas and lower proportions go to ReplicaSets with less replicas. Any leftovers are added to the ReplicaSet with the most replicas. ReplicaSets with zero replicas are not scaled up.

In our example above, 3 replicas will be added to the old ReplicaSet and 2 replicas will be added to the new ReplicaSet. The rollout process should eventually move all replicas to the new ReplicaSet, assuming the new replicas become healthy.

```
$ 
kubectl get deploy
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m

$ 
kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```



