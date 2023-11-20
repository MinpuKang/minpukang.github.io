---
layout:     post
title:      云原生｜关于K8s中Ingress和Egress流量的那些事儿
date:       2023-11-20
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

为什么要写`Linux｜聊聊Linux系统中的路由策略`这篇文章呢，主要为了今天这篇做铺垫。

随着5G技术的迅猛发展，通信网络正经历着一场前所未有的变革。在这个数字化、互联的时代，Kubernetes作为容器编排和管理的领军者，正在发挥越来越重要的作用，为5G网络的构建和管理提供了全新的可能性。

5G网络的特性包括超高速率、低时延、大连接性和大容量，这为各种应用场景提供了广阔的发展空间，如物联网、智能工厂、自动驾驶等。然而，这些新兴的应用对网络的可靠性、灵活性和可扩展性提出了更高的要求。在这个背景下，Kubernetes的出现成为了一个重要的技术支持。

通信网络中有很严格的流量管控，会划分多个VRF(Virtual Routing And Forwarding)，不同业务的流量，必须要发给对应的VRF，比如无线到核心网的无线流量，5G核心网间的SBI流量，登录各个设备进行网络管理的管理流量，都要在各自的VRF下走动，不能串线。
![@七禾页话][2]

但是Kubernetes的基础系统就是一个Linux OS，默认来说所有的流量都从主路由表出去，但是一个路由表只能顶一个默认路由，如果想把流量分发到对应的VRF下，就需要在主路由表里定义很多的路由条目。

比如下图中，主路由表里的默认路由是到VRF1的，那么当ssh设备和Linux通信的时候，在只有默认路由的前提下，Linux的响应包是无法到达ssh设备的(前提是不允许VRF间路由)：
![@七禾页话][3]

那么要怎么办呢？两个方案，一个是在主路由表里定义明细路由；另外一个就是定义routing policy。
![@七禾页话][4]

哪个方案比较好呢？在明确知道目的地址并且数量比较少的前提下，两个方案其实都不错；但是如果目的地址数量比较多，并且分布在不同网段中，或者目的地址不明确的情况下，只要源地址是清晰的，显然routing policy就更胜一筹了，routing policy保证了每一张路由表中都有自己的默认路由，就可以涵盖所有的目的地址。

我们再回到Kubernetes，在Kubernetes下，一个POD对外暴漏业务有很多的技术手段，这里说一个Service使用MetalLB做LoadBalancer的方案，如下图，其中在MetalLB的Configmap中定义了批一个IP为3.3.3.3的地址并分配给了Service ssh使用，同时Service ssh的endpoint是POD ssh，POD ssh被K8s分配了一个192.168.1.1的内部地址：
![@七禾页话][5]

那么我们来看当外部的一个ssh client想要登录到K8s里ssh这个POD的消息流程：
1. 在ssh client输入`ssh 3.3.3.3`之后，ssh client发出了一个数据包，其中数据包IP层的源地址是自己的地址10.10.10.10，目的地址是3.3.3.3；
2. 数据包通过一系列路由后到达了K8s，K8s对数据包做DNAT后，数据包变成源地址依然是10.10.10.10，目的地址DNAT成为了ssh的POD地址192.168.1.1，然后发给ssh这个POD；
3. ssh POD收到ssh请求后，开始构建回应的数据包，源地址是自己的POD地址即192.168.1.1，目的地址是10.10.10.10；
4. POD将数据包发出去后，K8s再对其做SNAT，此时数据包变成源地址被SNAT成为了Service地址即3.3.3.3，目的地址是10.10.10.10，然后K8s匹配路由表，将数据包发出去。

那么来看看第四步，SNAT之后K8s匹配路由表，将数据包发出去，在上图中可以看到主路由表里只有一个默认路由发给VRF1，但是ssh的业务是通过VRF2来完成的，所以ssh client上就会出现connection timeout等类似的错误，表现的结果就是ssh登录失败。

解决方案依然是两个，一在主路由表里定义明细路由；二就是使用routing policy新加一张针对vrf2的路由表，并在vrf2这张路由表定义一个默认路由，并定义规则如果数据包的源地址是3.3.3.3，则使用vrf2这张路由表。
![@七禾页话][6]

对于Kubernetes下使用Routing Policy目前只针对响应Ingress流量起作用，怎么理解呢？

当外部服务器首先发起的流量我们叫做Ingress流量，K8s内部的POD被动响应这个Ingress流量的时候，才会用到Routing Policy，因为只有是响应外部流量的时候，数据包在从K8s集群发出去前源地址才会被SNAT成Service的LoadBalancer地址，因此才会匹配到`ip rule`，然后才会使用对应的路由表。

那么对于K8s集群内部主动发起的流量呢？这部分流量称为Egress流量，POD在发起Egress流量的时候，先封装一个数据包：源地址为POD的IP即192.168.1.1，目的地址是10.10.10.10；然后数据包发给K8s，K8s需要针对目的地址进行路由匹配，然后将源地址192.168.1.1 SNAT成一个eth的地址后，才可以将数据包发出去，发给路由器，因为POD的地址只是才K8s集群内部使用，不能出现在外部的流量交互中。
![@七禾页话][7]

问题来了，如果没有使用Solution1定义一个到10.10.10.10的明细路由，那么依然匹配的是主路由表里的默认，也就是会发给VRF1，业务失败，所以此时必须要定义明细路由，我们自定义的Routing Policy是不生效的。

那么如果能让Routing Policy生效呢？理论上就是在K8s差看路由表之前需要将数据包的源地址SNAT成Service地址也就是3.3.3.3之后，就可以匹配上自定义的Routing Policy，就可以使用vrf2这个路由表里的默认路由了，目前没有看到K8s官方有任何相关的资料可以实现这个想法。

为什么K8s不考虑这个问题呢？因为K8s本身是互联网的杰作，一是业务比较单一，没有那么多业务分离的需求；另外互联网中K8s集群主要是给外部提供服务的，很少有集群内部主动发起的业务，有也是某些公司内部做数据监控等的需求，那么这些公司内部的IP都比较少且固定，很容易在主路由表里添加明细路由。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG1RDuKlvWwjs0roW8CggmWVxelOQgMHNfV4has48vicA3E1AFbSib9Fl4A/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG16iaXicvNjL13t0Rhcv9mR7vXKyVBaHrBV6pd8WH1sVL4llKwmHXGNSibw/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG18GQjen2P1TwTyyPhlibL1kKrHIL1czJCfNwEGjgYjhSLshZV6w6eU9A/640?wx_fmt=png&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG18GQjen2P1TwTyyPhlibL1kKrHIL1czJCfNwEGjgYjhSLshZV6w6eU9A/640?wx_fmt=png&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG15A1Ik1LUiatT9IPL03aETZ4zM9V10gGI5YSiaGjIickUQlksQibbUAnnpQ/640?wx_fmt=png&amp;from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG19zcAB0w09RuwibMOkMIENJ4SScVcUicEDrsjM3tN75JoibXOs9ddzpjzg/640?wx_fmt=png&amp;from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG1iaE8VBJ17n7XwB8PRWHUbNmvhEfCls8sWCElibcsqwNeib8qnNFUTxgyw/640?wx_fmt=png&amp;from=appmsg

