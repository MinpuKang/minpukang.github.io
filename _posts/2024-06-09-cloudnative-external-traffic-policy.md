---
layout:     post
title:      云原生｜K8s中external-traffic-policy导致的业务问题
date:       2024-06-09
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - CLoud
    - 云原生
    - K8s
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

最近有同事问我为什么有的SVC的External IP可以从k8s的control节点ping或者curl通，有的却不能。

经过查看发现是跟service中的external-traffic-policy有关系，那么external-traffic-policy是什么？为什么会有业务影响呢？

external-traffic-policy直白翻译就是“外部流量策略”，是控制外部访问service地址之后如何分发给Endpoint也就是相关业务POD的策略，是将流量仅仅转给node-local的POD还是可以转给cluster级别的也就是其他node上的POD。

external-traffic-policy有两个值：
- Local：流量只转给收到此流量的Node上的业务POD。
- Cluster：流量可以转给集群中任何Node的业务POD（可以是本Node上的，也可以是其他Node上的）。

读起来比较绕口，请看图：

![@七禾页话][2]

这两种模式的存在是因为kube-proxy在转发报文的时候是否需要保留原访问者的IP也就是数据包里的源地址。那么Local和Cluster两种模式有什么区别呢？

#### Cluster模式

Cluster模式是K8s中默认的external-traffic-policy模式，此模式中kube-proxy不用管POD到底在哪个node上，都会公平的转发。

因为其公平转发，当外部流量进入到集群中后，即使接收外部流量的node上存在业务POD，kube-proxy也可能将流量转给其他node上的POD，从而会看到node之间的不必要的传输，即增加了node之间的东西流量；与此同时，当kube-proxy在给pod转发外部流量的时候，还会做一次SNAT，将数据包中的源地址SNAT成接收此流量的Node的IP。看着也比较拗口，请看图：

![@七禾页话][3]

使用此模式后，数据包的IP如下：

![@七禾页话][4]

#### Local模式

Local模式下，kube-proxy只会将外部流量转发给同节点上的业务POD，不会出现跨Node的流量，从而减少了集群中东西向流量的压力，如下图：

![@七禾页话][5]

需要注意的是Local模式下，service的Type只能是NodePort或者LoadBalancer：

```
$ kubectl apply -f mysvc.yml
The Service "mysvc" is invalid: spec.externalTrafficPolicy: Invalid value: "Local": ExternalTrafficPolicy can only be set on NodePort and LoadBalancer service
```

在Local的模式下，由于不会出现跨Node的消息转发，所以需要一个LoadBalancer做到负载均衡，如下图：

![@七禾页话][6]

但是这样依然会造成POD之间的流量不均衡，因为LoadBalancer只会看到两个Node，是无法知道各个Node上有多少POD的，所以此时需要POD之间使用反亲和（anti-affinity）的设置从而保证每个Node上的POD是平均分布的：

![@七禾页话][7]

另外对于Local模式，如果某种原因导致外部流量发给了一个没有响应业务POD的Node，数据就会被丢弃，给大家的感知就是业务不通：

![@七禾页话][8]

造成这样的原因比如：
- 使用静态路由协议，将svc的ip的下一跳包含了集群所有Node，但是业务POD只在集群中某几个Node上，这种解决的方法就是使用动态路由协议，通过K8s自己来判断POD的分配后自动给路由器发布svc IP的路由。

- 再就是在集群上做业务测试的时候，如果随便选择了一个Node做ping或者curl的业务尝试，如果此Node上没有业务POD，就有会出现没有响应的结果。

### 总结

总的来说Cluster和Local两种模式各有千秋，存在即合理：

- 如果想要性能好、时延小，选择Local，少一次SNAT，也少了跨Node的转发，性能就会好很多，但是需要注意负载和路由的问题。

- 如果在没有很好的LoadBalancer的情况下，Cluster可以保证业务的均衡以及在逻辑上绝对不丢包的保证。

所以具体怎么使用，要结合集群所使用的相关技术来择优选择。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBGYDy2fxoHGcuvLEMRByib2MTnQcbHT00q8hpGoWrBZvXEjVnDepGymmaNazlLoUGicXRBZpBoW0Xg/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBGYDy2fxoHGcuvLEMRByib2gejW6mMzNibjCibjOqejaohPMm3HXH1Ep6ybpb2PsCbfgicskiccIiczH5A/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBGYDy2fxoHGcuvLEMRByib2PZT1lWIqEANPmjMfZgf52h7cPicEbmTf2at37bIw8beo2zCaEl0L2mQ/640?wx_fmt=png&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBGYDy2fxoHGcuvLEMRByib2fu7kfwzurQ7iaYU7YykR9oyy7zv7COzg9QCKHrCKL9LfDVxmCopocgQ/640?wx_fmt=png&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBGYDy2fxoHGcuvLEMRByib24aOY1YyQwNy05CLGZcjib6QIcmk49iaIE9BF5hngygyUPxz1Ngw9wB0Q/640?wx_fmt=png&amp;from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBGYDy2fxoHGcuvLEMRByib2clM5lO6RbafsP1n4qhDLRbLxSXQkV71qSNUqp8t5uf0Hje4nua5ljw/640?wx_fmt=png&amp;from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBGYDy2fxoHGcuvLEMRByib2b1WrociciatHelqykyNhJoynvibcAKJrIDC9LwGYhDeEVP6HHboSyQKcg/640?wx_fmt=png&amp;from=appmsg


[8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBGYDy2fxoHGcuvLEMRByib2gDy86Kibu4dlW4pkuGb1mpOv2JThEciatYwENmKZ3KAMJ2cTRgG5CVicQ/640?wx_fmt=png&amp;from=appmsg

