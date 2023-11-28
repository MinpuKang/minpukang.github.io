---
layout:     post
title:      云原生｜什么是HPA和PDB?
date:       2023-11-28
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

最近的项目，Kubernetes的人员要做系统维护的时候，需要修改我容器化应用的PDB，一直问我是否可以改动。

然后我就懵了，好奇的想知道PDB是一个什么东西？简单了解了之后，感觉跟HPA有点儿类似，确又有本质区别，相近的两个概念，容易引起迷糊，就一起简单聊一聊。

在Kubernetes中，HPA和PDB是两个非常关键的概念，用于自动化地调整应用程序的规模和管理维护期间的Pod容忍性，那么他们具体是干什么的，都在什么场景下使用呢？

## Horizontal Pod Autoscaler（HPA）

### 概述

HPA是Horizontal Pod Autoscaler的缩写，它在Kubernetes中允许根据应用程序的负载动态调整Pod的副本数量，从而使得应用程序能够自动扩展或收缩，以适应变化的工作负载，进而提高资源利用率和应用程序的性能。

### 原理

HPA 的原理基于两个核心概念：**指标（Metrics）**和**目标值（Target Value）**。

1. **指标：** HPA 使用预定义的或自定义的指标（例如 CPU 使用率、内存使用率）来监控应用程序的负载。这些指标反映了应用程序当前的性能状况。

2. **目标值：** HPA 需要设定一个目标值，表示期望的指标水平。当监测到应用程序的实际指标超过或低于这个目标值时，HPA 将触发相应的伸缩操作。

### 使用场景

HPA 在以下场景中特别有用：

- **流量波动：** 当应用程序面临流量波动的时候，需要在使用高峰期动态扩展副本，低谷的时候再缩减相应副本时，HPA是能够自动完成这一过程的，比如每天中午的忙时扩展，午夜闲时自动收编。

- **成本优化：** HPA还可以根据实际负载调整规模，以提高资源利用率，从而降低云服务成本，比如双十一期间因订单量激增，自动扩容申请，但是双十一过后自动回复平常的使用水平。

### 示例

接下来让我们用一个简单的小例子来看看HPA的使用。假设有一个Web服务，我们希望在CPU使用率达到80%时自动扩展，下降到20%时自动缩容。

首先，需要创建一个HPA对象，如下所示：

```yaml
@@左右滑动
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-deployment-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

apply一下看看结果：
```bash
@@左右滑动
ubuntu@VM-16-3-ubuntu:~$ cat hpa.yaml 
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-deployment-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
ubuntu@VM-16-3-ubuntu:~$ 
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-89f9774c6-vrzs7   1/1     Running   0          11s
ubuntu@VM-16-3-ubuntu:~$ kubectl get deployments.apps 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           14s
ubuntu@VM-16-3-ubuntu:~$ kubectl apply -f hpa.yaml 
horizontalpodautoscaler.autoscaling/nginx-deployment-hpa created
ubuntu@VM-16-3-ubuntu:~$ kubectl get hpa
NAME                   REFERENCE                     TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment-hpa   Deployment/nginx-deployment   <unknown>/80%   1         10        1          25s
ubuntu@VM-16-3-ubuntu:~$ 
```

可以看到HPA已经成功检测到Deployment的当前副本，同时已经设定了Targets是80%，但是由于没有Metrix，所以当前的使用率无法获取。

这个小例子中定义了一个叫做为`nginx-deployment-hpa`的HPF并关联了名字叫做`nginx-deployment`的Deployment。当CPU使用率平均值达到80%时，HPA将触发自动扩展，确保Pod的数量在1到10之间。

通过应用这个HPA对象，Kubernetes将根据CPU使用率的变化自动调整Pod的数量，以确保nginx服务的性能和可用性。

## Pod Disruption Budget（PDB）

### 概述

PDB的全称叫做Pod Disruption Budget，是用于控制维护期间Pod中断的策略。它确保在进行计划性维护或者升级时不会导致应用程序的过度中断，从而提高应用程序的可靠性。

### 原理

PDB使用两个关键概念：**最小可用副本数（Min Available）**和**最大不可用副本数（Max Unavailable）**，分别用于定义在维护期间需要保持的最小可用Pod数量和允许的最大不可用Pod数量：

1. **`minAvailable`（最小可用 Pod 数量）：**

   - `minAvailable` 用于指定在维护期间必须保持运行的Pod的最小数量。这确保了即使在维护期间，也有足够数量的Pod在运行，以保持服务的可用性。

   - 我们可以将 `minAvailable` 设置为一个整数值，也可以是一个百分比。例如，如果设置为2，表示在维护期间至少要保持2个Pod在运行。如果设置为百分比，它表示相对于ReplicaSet（或其他控制器）中当前运行的Pod数量的百分比。

2. **`maxUnavailable`（最大不可用 Pod 数量）：**

   - `maxUnavailable` 用于指定在维护期间允许的最大不可用 Pod 数量。这是一个相对值，通常与 `minAvailable` 一起使用，提供了更灵活的控制。

   - 跟`minAvailable`，我们也可以将 `maxUnavailable` 设置为一个整数值或百分比。例如，设置为 `maxUnavailable: 1` 表示在维护期间允许最多1个Pod不可用；设置为 `maxUnavailable: 10%` 表示在维护期间允许最多10%的Pod不可用。

简单来说就是在Kubernetes维护升级期间，有最多有多少POD可以被删掉，至少要有多少POD必须Ready来提供服务。

### 使用场景

PDB一般适用于以下场景：

- **保障服务可用性：** 在进行计划性维护时，PDB可以确保至少保持指定数量的Pod在运行，从而不影响服务的可用性。

- **避免突发中断：** PDB可以防止在维护期间出现过多的Pod中断，确保应用程序的稳定运行。

### 示例

继续做实验，比如，比如针对Nginx的POD定义了如下的PDB策略：

```yaml
@@左右滑动
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  minAvailable: 2  # 至少保持 2 个 Pod 在运行
  maxUnavailable: 1  # 允许最多 1 个 Pod 不可用
  selector:
    matchLabels:
      app: nginx  # 应用程序标签
```

在这个例子中，PDB `nginx-pdb` 将确保在维护期间至少有2个Pod在运行，并且允许最多1个Pod不可用。请注意，`selector` 部分根据部署nginx所打的label来定义。

结果在apply的时候遇到了问题，提示min和max不能同时定义，没办法，注释掉一个apply成功：
```bash
@@左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-89f9774c6-vrzs7   1/1     Running   0          30m
nginx-deployment-89f9774c6-fnbsx   1/1     Running   0          16s
nginx-deployment-89f9774c6-9bw6m   1/1     Running   0          16s
ubuntu@VM-16-3-ubuntu:~$ kubectl apply -f pdb.yaml 
The PodDisruptionBudget "nginx-pdb" is invalid: spec: Invalid value: policy.PodDisruptionBudgetSpec{MinAvailable:(*intstr.IntOrString)(0xc01a234760), Selector:(*v1.LabelSelector)(0xc01a234780), MaxUnavailable:(*intstr.IntOrString)(0xc01a234740), UnhealthyPodEvictionPolicy:(*policy.UnhealthyPodEvictionPolicyType)(nil)}: minAvailable and maxUnavailable cannot be both set
ubuntu@VM-16-3-ubuntu:~$ 
ubuntu@VM-16-3-ubuntu:~$ vi pdb.yaml 
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  minAvailable: 2  # 至少保持 2 个 Pod 在运行
  #maxUnavailable: 1  # 允许最多 1 个 Pod 不可用，提示冲突后，注释掉maxUnavailable
  selector:
    matchLabels:
      app: nginx  # 应用程序标签
ubuntu@VM-16-3-ubuntu:~$ 
ubuntu@VM-16-3-ubuntu:~$ kubectl apply -f pdb.yaml 
poddisruptionbudget.policy/nginx-pdb created
ubuntu@VM-16-3-ubuntu:~$ kubectl get pdb
NAME        MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
nginx-pdb   2               N/A               1                     5s
ubuntu@VM-16-3-ubuntu:~$ 
```

所以在真正使用的时候需要特别注意PDB，我们就因为PDB的问题，导致Kubernetes更新失败并无法修复，最后重装了Kubernetes。

HPA和PDB作为Kubernetes中两个关键的概念，用于应对不同的场景和挑战。HPA使得应用程序能够根据实时负载动态调整规模，提高资源利用率；而PDB则确保在计划性维护期间最小化Pod中断，提高应用程序的可靠性。通过合理使用这两个功能，我们可以更好地管理和优化Kubernetes集群中的应用程序。

在具体使用中，需要针对业务的需求来决定是否使用，使用后如何定义具体的参数。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eC9eFNicvUukSmrZbKibia3jicullYPxwWYEU3l646sphEGxBeU7ibCJ0GCzMo4gu0OlJHEDTuxxwYurhA/640?wx_fmt=jpeg

