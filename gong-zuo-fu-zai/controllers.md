英文地址：[https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

# ReplicaSet

---

ReplicaSet是下一代的Replication Controller。目前，ReplicaSet和Replication Controller的区别只是选择器的支持。ReplicaSet 支持set-based条件的选择器，而Replication Controller仅仅支持equality-based条件的选择器。

## 如何使用ReplicaSet

---

大多数支持Replication Controllers的[`kubectl`](https://kubernetes.io/docs/user-guide/kubectl/)commands 指令也同时支持ReplicaSets。其中的一个例外就是[`rolling-update`](https://kubernetes.io/docs/user-guide/kubectl/v1.9/#rolling-update)指令。如果你想使用滚动升级功能，请使用Deployments来代替。同样地，[`rolling-update`](https://kubernetes.io/docs/user-guide/kubectl/v1.9/#rolling-update)command指令是imperative的，而Deployments是declarative的，所以我们推荐通过[`rollout`](https://kubernetes.io/docs/user-guide/kubectl/v1.9/#rollout)指令使用Deployments。

尽管ReplicaSets可以单独使用，但现在它主要被[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)当做一种pod创建、删除和更新的编排机制。当使用Deployments的时候，你不必担心如何管理他们创建的的ReplicaSets。Deployments拥有且管理他们的ReplicaSets。

## 什么时候使用ReplicaSet

---

ReplicaSet会确保在任意时刻，都有指定数量的Pod副本在运行。但是，Deployment是一个更高级别的概念，用来管理ReplicaSets并为Pod提供声明性更新（declarative updates），同时还有一些其他拥有的功能。因此，我们推荐使用Deployments，而不是直接使用ReplicaSets，除非你想要定制更新编排或根本不需要更新。

这其实是说，你可能永远不必手动操作ReplicaSet对象：而是使用Deployment，并在它的spec部分定义你的应用。

## 例子

---

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # this replicas value is default
  # modify it according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: tier, operator: In, values: [frontend]}
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access environment variables to find service host
          # info, comment out the 'value: dns' line above, and uncomment the
          # line below.
          # value: env
        ports:
        - containerPort: 80
```

将这份清单保存到`frontend.yaml`文件中，并将其提交给Kubernetes集群，就会创建定义好的ReplicaSet和它管理的Pods。

```
$ kubectl create -f frontend.yaml
replicaset "frontend" created
$ kubectl describe rs/frontend
Name:		frontend
Namespace:	default
Selector:	tier=frontend,tier in (frontend)
Labels:		app=guestbook
		tier=frontend
Annotations:	<none>
Replicas:	3 current / 3 desired
Pods Status:	3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:       app=guestbook
                tier=frontend
  Containers:
   php-redis:
    Image:      gcr.io/google_samples/gb-frontend:v3
    Port:       80/TCP
    Requests:
      cpu:      100m
      memory:   100Mi
    Environment:
      GET_HOSTS_FROM:   dns
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen    LastSeen    Count    From                SubobjectPath    Type        Reason            Message
  ---------    --------    -----    ----                -------------    --------    ------            -------
  1m           1m          1        {replicaset-controller }             Normal      SuccessfulCreate  Created pod: frontend-qhloh
  1m           1m          1        {replicaset-controller }             Normal      SuccessfulCreate  Created pod: frontend-dnjpy
  1m           1m          1        {replicaset-controller }             Normal      SuccessfulCreate  Created pod: frontend-9si5l
$ kubectl get pods
NAME             READY     STATUS    RESTARTS   AGE
frontend-9si5l   1/1       Running   0          1m
frontend-dnjpy   1/1       Running   0          1m
frontend-qhloh   1/1       Running   0          1m
```

## 编写ReplicaSet Spec

---

todo  




