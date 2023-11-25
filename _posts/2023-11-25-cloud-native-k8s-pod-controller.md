---
layout:     post
title:      云原生｜Hi，Pod，你被控制了！
date:       2023-11-25
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - CLoud Native
    - 云原生
    - K8s
    - Kubernetes
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

![][0]

![@七禾页话][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

了解了Pod的基础知识之后，对于实验来说可以通过kubectl run或者apply一个yaml来创建Pod，但是对于生产环境中构建一个CNF来说，有些Pod需要多个副本，有的运行完就不再需要了，有些需要定期执行某些任务，有些需要在不同的node上只创建一个Pod，这样通过一个一个的创建Pod是不仅费时费力且不便于维护，因此需要一个概念来根据不同需求创建对应的Pod并确保在任何时候都有对应要求的副本在运行，这个概念便是Pod的控制器。

Pod 控制器是 Kubernetes 引入的一种抽象概念，用于确保在集群中维护指定数量的 Pod 副本。它们负责处理 Pod 的create、delete、Scale等操作，以满足用户定义的状态。

Kubernetes 提供了多种类型的 Pod 控制器，其中包括 ReplicaSet、Deployment、StatefulSet、DaemonSet、Jobs和Conjob等。每种控制器都有其特定的应用场景和功能，使其适用于不同类型的应用程序。

## ReplicaSet

ReplicaSet 是 Kubernetes 中最基本的 Pod 控制器之一。它的主要任务是确保指定数量的 Pod 副本在任何时候都在运行。如果有太多的副本，ReplicaSet 会进行缩减；如果数量不足，它将启动新的 Pod 副本，最终保证副本是定义的数量。

**示例：**

```yaml
##左右滑动
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3

```

需要注意的是ReplicaSet控制的Pod可以创建到同一个node上的，前提是没有设定严格的anti-affinity策略：
```bash
##左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl get replicasets.apps 
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-66fb7f764c   2         2         2       42h
ubuntu@VM-16-3-ubuntu:~$ 
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
nginx-deployment-66fb7f764c-6qhbj   1/1     Running   0          42h   10.42.0.17   vm-16-3-ubuntu   <none>           <none>
nginx-deployment-66fb7f764c-nxqj7   1/1     Running   0          42h   10.42.0.16   vm-16-3-ubuntu   <none>           <none>
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod --no-headers | awk '{print $1}' | xargs -t -l kubectl describe pod | grep -iE "^Name:|^Namespace:|^Controlled By:|^Node:"
kubectl describe pod nginx-deployment-66fb7f764c-6qhbj
Name:             nginx-deployment-66fb7f764c-6qhbj
Namespace:        default
Node:             vm-16-3-ubuntu/10.0.16.3
Controlled By:  ReplicaSet/nginx-deployment-66fb7f764c
kubectl describe pod nginx-deployment-66fb7f764c-nxqj7
Name:             nginx-deployment-66fb7f764c-nxqj7
Namespace:        default
Node:             vm-16-3-ubuntu/10.0.16.3
Controlled By:  ReplicaSet/nginx-deployment-66fb7f764c
ubuntu@VM-16-3-ubuntu:~$ 

```

## Deployment

我们先直接打印一个ReplicaSet的describe结果，可以看到这个ReplicaSet是被Deployment控制的：
```bash
ubuntu@VM-16-3-ubuntu:~$ kubectl get replicasets.apps 
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-66fb7f764c   2         2         2       42h
ubuntu@VM-16-3-ubuntu:~$ kubectl describe replicasets.apps nginx-deployment-66fb7f764c | grep -iE "^Name:|^Namespace:|^Controlled By:"
Name:           nginx-deployment-66fb7f764c
Namespace:      default
Controlled By:  Deployment/nginx-deployment
ubuntu@VM-16-3-ubuntu:~$ 
ubuntu@VM-16-3-ubuntu:~$ kubectl get deployments.apps 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           42h
```

所以说Deployment 是建立在 ReplicaSet 之上的高级控制器，它提供了声明式定义应用程序的能力。Deployment 允许轻松进行应用程序的升级和回滚，而无需手动管理 ReplicaSet，总体来说Deployment在ReplicaSet的基础上提供了对Pod的更灵活的控制能力。

**示例：**

```yaml
##左右滑动
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```

## StatefulSet

StatefulSet 是一种用于维护有状态应用程序的稳定控制器。每个 Pod 在 StatefulSet 中都有唯一的标识，比如db-0、db-1、db-2，并且按照顺序进行部署和删除。适用于需要稳定网络标识和有序部署的应用程序，如数据库。

**示例：**

```yaml
##左右滑动
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: docker.io/library/nginx:latest
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "local-path"  #need check by kubectl get sc
      resources:
        requests:
          storage: 1Gi
```

一般来说StatefulSet控制的Pod都需要稳定的后端存储，也就是创建的时候会申请自己独立的存储空间，其次StatefulSet控制的Pod的存储是需要同步的，从而保证了数据的一致性。同时因为每个Pod都有自己唯一稳定的标识，那么如果删除Pod后，依然会以原名字创建，并且依然使用之前已经分配好的存储空间，也就是StatefulSet控制的Pod的存储具有永久性的特点。

比如我apply了上述示例中的yaml后，得到了如下的结果，创建的三个Pod是顺序启动的，并且分别关联了自己的PVC，每个PVC都有自己的PV来申请独立存储：
```bash
##左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          7m12s
web-1   1/1     Running   0          6m52s
web-2   1/1     Running   0          6m32s
ubuntu@VM-16-3-ubuntu:~$ kubectl get pvc
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
www-web-0   Bound    pvc-a20beb6d-1c02-4eef-9c22-b2efcc5bd6ac   1Gi        RWO            local-path     7m19s
www-web-1   Bound    pvc-99917fce-9ccf-4469-8ed1-9725b2155c1a   1Gi        RWO            local-path     6m59s
www-web-2   Bound    pvc-829dddb2-aa22-47a6-b2b4-ca200cbf3a19   1Gi        RWO            local-path     6m39s
ubuntu@VM-16-3-ubuntu:~$ for i in `kubectl get pod | awk '{print $1}'`;do echo $i;kubectl describe pod $i | grep -i "Volumes:" -A4;done
NAME
Error from server (NotFound): pods "NAME" not found
web-0
Volumes:
  www:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  www-web-0
    ReadOnly:   false
web-1
Volumes:
  www:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  www-web-1
    ReadOnly:   false
web-2
Volumes:
  www:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  www-web-2
    ReadOnly:   false
ubuntu@VM-16-3-ubuntu:~$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
pvc-a20beb6d-1c02-4eef-9c22-b2efcc5bd6ac   1Gi        RWO            Delete           Bound    default/www-web-0   local-path              9m37s
pvc-99917fce-9ccf-4469-8ed1-9725b2155c1a   1Gi        RWO            Delete           Bound    default/www-web-1   local-path              9m17s
pvc-829dddb2-aa22-47a6-b2b4-ca200cbf3a19   1Gi        RWO            Delete           Bound    default/www-web-2   local-path              8m57s
ubuntu@VM-16-3-ubuntu:~$ 
```

接下来我尝试删除web-0这个Pod，可以看到重新创建后的Pod依然是web-0，并且依然使用www-web-0这个PVC，PVC和PV都没有任何变动，也就是web-0这个Pod的存储内容并不跟随Pod的重建而修改：
```bash
##左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl delete pod web-0
pod "web-0" deleted
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
web-1   1/1     Running   0          10m
web-2   1/1     Running   0          10m
web-0   1/1     Running   0          5s
ubuntu@VM-16-3-ubuntu:~$ kubectl describe pod web-0 | grep -i "Volumes:" -A4
Volumes:
  www:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  www-web-0
    ReadOnly:   false
ubuntu@VM-16-3-ubuntu:~$ kubectl get pvc www-web-0
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
www-web-0   Bound    pvc-a20beb6d-1c02-4eef-9c22-b2efcc5bd6ac   1Gi        RWO            local-path     11m
ubuntu@VM-16-3-ubuntu:~$ kubectl get pv | grep www-web-0
pvc-a20beb6d-1c02-4eef-9c22-b2efcc5bd6ac   1Gi        RWO            Delete           Bound    default/www-web-0   local-path              11m
ubuntu@VM-16-3-ubuntu:~$ 
```

## DaemonSet

接下来再看一个有意思的控制器：DaemonSet。

DaemonSet 是用于确保在集群中的每个节点上运行一个副本的 Pod，不同于前三个控制器，它不受副本数控制，而是直接在可以调度的node上直接创建Pod，并且每个node上只创建一个，不能跨越多个节点。通常用于在每个节点上运行一些系统级别的任务，如日志收集、监控等。

**示例：**

```yaml
##左右滑动
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      # it may be desirable to set a high priority class to ensure that a DaemonSet Pod
      # preempts running Pods
      # priorityClassName: important
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

当我们查看DaemonSet的时候，可以看到有多个参数，其中Desired就是期望的Pod数量，它匹配的就是可以调度的node的数量；
```bash
##左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl get daemonsets.apps -n kube-system 
NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
svclb-traefik-1b6c5f6f   1         1         1       1            1           <none>          45h
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod -n kube-system svclb-traefik-1b6c5f6f-7rhcj 
NAME                           READY   STATUS    RESTARTS      AGE
svclb-traefik-1b6c5f6f-7rhcj   2/2     Running   2 (45h ago)   45h
ubuntu@VM-16-3-ubuntu:~$ 
```

查看可以调度的Node，比如是否被taint了：
```bash
##左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl get nodes -o wide --output=custom-columns=NAME:.metadata.name,STATUS:.status.conditions[0].status,SCHEDULABLE:.spec.taints
NAME             STATUS   SCHEDULABLE
vm-16-3-ubuntu   False    <none>
```

继续试验，将node打tain使其不可以调度：
```bash
##左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl taint node vm-16-3-ubuntu Test=true:NoSchedule
node/vm-16-3-ubuntu tainted
ubuntu@VM-16-3-ubuntu:~$ kubectl get nodes -o wide --output=custom-columns=NAME:.metadata.name,STATUS:.status.conditions[0].status,SCHEDULABLE:.spec.taints
NAME             STATUS   SCHEDULABLE
vm-16-3-ubuntu   False    [map[effect:NoSchedule key:Test value:true]]
ubuntu@VM-16-3-ubuntu:~$ 
```

再查看DaemonSet会看到检测不到Pod了，但是已经创建的Pod依然存在并不受影响：
```bash
##左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl get daemonsets.apps -n kube-system 
NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
svclb-traefik-1b6c5f6f   0         0         0       0            0           <none>          45h
ubuntu@VM-16-3-ubuntu:~/harbor$ kubectl get pod -n kube-system -o wide | grep -iE "svclb-tra|NAME"
NAME                                     READY   STATUS      RESTARTS        AGE     IP           NODE             NOMINATED NODE   READINESS GATES
svclb-traefik-1b6c5f6f-7rhcj             2/2     Running     2 (45h ago)     45h     10.42.0.35   vm-16-3-ubuntu   <none>           <none>
```

​此时如果我们删除当前taint的node上被DaemonSet控制的Pod后，就不会再有新的Pod创建在此node上了，因为此node不可被自动调度（需要搭配teloration才可以）：
```bash
##左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl delete pod -n kube-system svclb-traefik-1b6c5f6f-7rhcj 
pod "svclb-traefik-1b6c5f6f-7rhcj" deleted
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod -n kube-system | grep -iE "svclb-tra|NAME"
NAME                                     READY   STATUS      RESTARTS      AGE
ubuntu@VM-16-3-ubuntu:~$ 
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod -n kube-system | grep -iE "svclb-tra|NAME"
NAME                                     READY   STATUS      RESTARTS      AGE
ubuntu@VM-16-3-ubuntu:~$ 
```

删除taint使node可以被自动调度后，DaemonSet控制的Pod会立马在此node上创建一个新的出来：
```bash
##左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl taint node vm-16-3-ubuntu Test=true:NoSchedule-
node/vm-16-3-ubuntu untainted
ubuntu@VM-16-3-ubuntu:~/harbor$ kubectl get pod -n kube-system -o wide | grep -iE "svclb-tra|NAME"
NAME                                     READY   STATUS      RESTARTS        AGE     IP           NODE             NOMINATED NODE   READINESS GATES
svclb-traefik-1b6c5f6f-8rgf9             2/2     Running     0               2s      10.42.0.35   vm-16-3-ubuntu   <none>           <none>
ubuntu@VM-16-3-ubuntu:~$ kubectl get daemonsets.apps -n kube-system 
NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
svclb-traefik-1b6c5f6f   1         1         1       1            1           <none>          45h
ubuntu@VM-16-3-ubuntu:~$ 
```

## Job 和 CronJob

还有一些应用场景是我们需要Pod执行任务就结束了，有的需要定时执行任务，这就用到了Job和CronJob这两个Controller。

Job 用于一次性运行任务的 Pod，完成后基本不会再被创建。CronJob 是一个基于时间表的控制器，用于按计划运行 Job，每次到点儿都要创建一个Pod执行任务。

**示例：**

```yaml
##左右滑动
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

```yaml
##左右滑动
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

当我们使用`kubectl get pod`的命令后看到状态是Complete的Pod基本就是被Job或者CronJob控制的。
```bash
##左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod -n kube-system
NAME                                     READY   STATUS      RESTARTS      AGE
helm-install-traefik-bdqsn               0/1     Completed   2             46h
helm-install-traefik-crd-qn7wt           0/1     Completed   0             46h
traefik-9df5858df-nw96h                  1/1     Running     1 (45h ago)   46h
coredns-5cf45fb78d-lpnqx                 1/1     Running     1 (45h ago)   46h
local-path-provisioner-6f65f9b6d-4zxm5   1/1     Running     2 (45h ago)   46h
metrics-server-7dbd8c95c-w9b9j           1/1     Running     2 (45h ago)   46h
svclb-traefik-1b6c5f6f-8rgf9             2/2     Running     0             5m29s
ubuntu@VM-16-3-ubuntu:~$ kubectl get jobs -n kube-system 
NAME                       COMPLETIONS   DURATION   AGE
helm-install-traefik-crd   1/1           23s        46h
helm-install-traefik       1/1           39s        46h
ubuntu@VM-16-3-ubuntu:~$ kubectl describe pod -n kube-system helm-install-traefik-bdqsn | grep -iE "^Name:|^Controlled"
Name:             helm-install-traefik-bdqsn
Controlled By:    Job/helm-install-traefik
```

以上就是目前所定义的Pod控制器，在 Kubernetes 中，选择正确的 Pod 控制器是确保应用程序稳定运行的关键一步。每个控制器都有其独特的功能和适用场景，根据应用程序的要求进行选择。通过深入理解每个控制器的特性和示例，我们可以更好地利用 Kubernetes 强大的容器编排功能，确保应用程序在集群中高效稳定的运行。

## Pod的Stateless和Stateful

了解了这么多Pod的控制器，会注意到，有些控制器创建的Pod名字的后缀是随机的，而且重新创建后，随机数会变，有些控制器创建的Pod名字是固定有规律的，这就引申出Pod有无状态的概念。

我们可以简单粗暴的说那些随机数的Pod就是Stateless，比如ReplicaSet、Deployment、DaemonSet、Job、CronJob创建的Pod都是Stateless的，系统创建Pod的时候，各个副本没有先后顺序，随机创建，名字序列也随机，每个 Pod 实例都可以被视为独立的、无关紧要的单元，其创建和销毁不影响应用程序的正常运行。

但是对于StatefulSet创建的Pod的状态就是Stateful的，部署的时候每个Pod有序安装，并且要分配唯一不变的序列号，系统需要严格监控各个副本的状态，一旦某一个序列号的Pod出问题或者消失，系统需要重新创建一个同序列号的Pod。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eC9eFNicvUukSmrZbKibia3jicuEZ5zTrichh1lV4rmCJekrXp6Ftghfu8uuX0rs4sPh6sRpFRSc5Dzjhw/640?wx_fmt=jpeg

