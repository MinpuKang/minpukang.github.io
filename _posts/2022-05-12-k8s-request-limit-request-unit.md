---
layout:     post
title:      虚拟化｜聊聊K8s里的Request和Limit和资源单元
date:       2022-05-12
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 通信
    - Linux
    - 虚拟化
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

![就很牛][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

直入正题，最近经常被问到两个问题，这里做个总结。

### Request和Limit

一个问题是K8s里在定义容器资源时候的Request和Limit有啥联系和区别。

就是字面上的意思，request里定义的是k8s必须要保证的启动资源，limit是将来容器运行可能使用的资源上限。

Kube-scheduler通过request的定义来寻找一个可以满足需求的node，从而在node上启动对应的pod里所用的容器；但是容器运行之后因为业务的增长是可以使用超过request的资源的，但是最高只能用到limit里定义的资源，但是limit里定义的资源k8s是不能给确保提供的。

![][2]

如下的一段定义，启动pod的时候只要有node可以提供64Mi的内存和250m的cpu，这个pod就可以启动，但是业务增长后，250m的cpu已经不能满足了，如果node依然有足够的资源，pod里的容器就可以继续使用cpu一直到500m的limit截止。
```
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

这里就涉及到一个POD的Qos的概念，K8s定义了三种Qos类型：

- Guaranteed

- Burstable

- BestEffort

一张图看看这三种Qos类型：

![][3]

可以通过如下命令查看pod的qos类型：
```
#左右滑动
kubectl get pod <pod name> -n <namespace> -o yaml | grep -i qosclass

kubectl describe pod <pod name> -n <namespace> | grep -i "qos class"

Note: describe和以yaml格式输出的get结果中的qosclass参数是不一样的，有大小写和空格，所以要注意使用。
```

需要注意的一个问题是在生产环境中一个pod可能由一个或者多个容器构成的，Request和Limit的资源定义是针对一个一个容器的，但是Qos Class却是POD级别的，所以查看到一个Qos Class之后可以确定

- Guaranteed：POD中的所有容器都指定了Request和Limit，并且每个容器任何类型资源(cpu and memory)的request和limit是一样的。

- Burstable：POD中肯定有一个容器的某个资源类型(cpu or memory)的Limit是大于Request的设定。

- BestEffort：POD中所有容器都没有定义任何资源(cpu and memory)需求，也就是任何资源需求都是0。

以上是第一个问题。

### 资源单位

第二个被问到的就是在定义资源的时候经常看到100m、0.5等等的，这个都是什么鬼？

直接看K8s官网怎么说的：

###### CPU的资源单位

> CPU资源的约束和请求以 “cpu” 为单位。 在 Kubernetes 中，一个 CPU 等于1个物理CPU核或者一个虚拟CPU核， 取决于节点是一台物理主机还是运行在某物理主机上的虚拟机。
> 
> 很小的CPU的请求也是允许的。当你定义一个容器，将其 spec.containers[].resources.requests.cpu 设置为 0.5 时， 你所请求的 CPU 是你请求 1.0 CPU 时的一半。 对于 CPU 资源单位，数量 表达式 0.1 等价于表达式 100m，可以看作 “100 millicpu”。 有些人说成是“一百毫核”，其实说的是同样的事情。
> 
> CPU资源总是设置为资源的绝对数量而非相对数量值。 例如，无论容器运行在单核、双核或者48核的机器上，500m CPU 表示的是大约相同的计算能力。

总结来说就是一个cpu被”毫“分隔成了1000份，也就是1000m，那么在定义一个容器资源需求的时候就可以按照一定比例或者更详细的规划cpu的请求，比如200m、0.2等等的。

还有一个Note需要注意：

> Kubernetes 不允许设置精度小于 1m 的 CPU 资源。 因此，当 CPU 单位小于 1 或 1000m 时，使用毫核的形式是有用的； 例如 5m 而不是 0.005。

###### 内存资源单位

顺便就再说一下相对简单的Memory。

Memory 的约束和请求以字节为单位。 你可以使用普通的不带单位的字节，或者带有具体单位的数字来表示内存：E、P、T、G、M、k。 你也可以使用对应的2的幂数：Ei、Pi、Ti、Gi、Mi、Ki。 例如，以下表达式所代表的是大致相同的值：
```
128974848、129e6、129M、128974848000m、123Mi
```
请注意后缀的大小写。如果你请求 400m 内存，实际上请求的是 0.4 字节。 如果有人这样设定资源请求或限制，可能他的实际想法是申请 400 兆字节（400Mi） 或者 400M 字节。

什么是2的幂数？举个例子M和Mi，1M就是1\*1000\*1000的字节，1Mi就是1\*1024\*1024字节，所以1M < 1Mi，显然带这个小i的更准确。

以上就是K8s里关于Request和Limit，以及资源单位的简介。

更多详细的内容可以查看官网里如下的两篇内容：
- https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
- https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBYgTOZib66NxFde8aUVLf4Et8dLXrqibOrIchiaCtkBJFBvaBI7VuZeASqAftpibjtRia0C7MQLZh2hcw/0?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBYgTOZib66NxFde8aUVLf4EGLz7R6sxKEqqUku9CfndU3aPE2IyXD8zq1te8MARZj62xt2vAibszUw/0?wx_fmt=png


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBYgTOZib66NxFde8aUVLf4EpwXeTssMeVUrEZX34oIPLC4wOjoGyAUDiar4xYvTficYjQAwiaIpjy5Vg/0?wx_fmt=png

