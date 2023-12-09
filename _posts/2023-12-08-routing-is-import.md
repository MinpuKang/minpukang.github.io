---
layout:     post
title:      通信｜通信三要素，路由，路由，还是XXX路由
date:       2023-12-08
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 通信
    - 路由
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

最近做项目的时候，经常被路由的问题困扰，不是被路由困扰，而是被大家理解不到路由而困扰。

所以很多时候都是在无休止的跟人讲路由，感觉怎么来理解并梳理路由是一个问题。

技术上的东西真的是仁者见仁智者见智，每个人都有自己的一套学习方法和对其理解的逻辑，但是基本有大概几个层次：
- 我不知道这个东西
- 我知道这个东西，但是不会运用
- 我知道这个东西，基本会使用
- 我熟悉这个东西，可以讲给别人听
- 我非常熟悉这个东西，规则是我定的

对于做通信运维或者Linux系统运维的并且大概有五年左右的工作经验的朋友来说，路由至少应该达到“我熟悉这个东西，可以讲给别人听”的程度。

比如有两个概念，在我跟一些同事聊天的过程中就感觉是混淆的，一个是静态路由，一个是默认路由。

是不是有人开始嘀咕了，这两有啥可混淆的，我心里也是这么想的，但是事实就是如此。

静态路由对标的是动态路由，针对的是路由发布定义的协议。

默认路由是在所有路由都无法匹配的时候而使用的路由条目，它是静态路由或者动态路由里的一条路由。

比如Linux下通过`ip route`匹配的default或者`route -n`匹配的0.0.0.0就是使用静态路由协议的默认路由条目。
```bash
##左右滑动
ubuntu@VM-16-3-ubuntu:~$ ip route | grep default
default via 10.0.16.1 dev eth0 proto dhcp src 10.0.16.3 metric 100 
ubuntu@VM-16-3-ubuntu:~$ route -n |grep -E ^0.0.0.0
0.0.0.0         10.0.16.1       0.0.0.0         UG    100    0        0 eth0
ubuntu@VM-16-3-ubuntu:~$ 
```

所以默认路由的目的地址是0.0.0.0/0可以匹配所有IP的一个路由条目。

那么一条路由到底都包含什么东西呢？很简单的两个重要的点：目的地址和下一跳。只要把握好了这两个，路由就没有问题了。
- 目的地址就是你要跟谁通信？谁就是目的地址，这个目的地址在网络传输中几乎是永远不变的(变，可能有NAT)。
- 下一跳就是你要把这个包发给谁？这个谁就是下一跳，下一跳在网络传输中是实时变化的。

举个现实里的例子，你网上买了一个东西，商家要给你发货，那么对商家来说，你就是目的地址，那么商家把包裹封装好了之后要扔给谁呢？快递接收点就是商家的下一跳，而快递小哥就是那条负责把包裹扔到接收点儿的路由；然后快递小哥把包裹扔到了快递接收点后，接收点有通过卡车运到集散地，集散地就是快递接收点儿的下一跳，卡车就是路由，以此类推，一直到将包裹交到你这个目的地址的手上。

继续对应到网络，我们来梳理一个下图中从H1到H2通信中各个地点的路由，梳理路由，我的建议是模拟发包：
![@七禾页话][2]

首先H1肯定要知道H2的地址也就是4.4.4.2，在封装IP层的时候不用考虑掩码，只需要知道你要给谁发包的IP地址即可，如果非得纠结掩码，可以使用32，那么来看分装包的三层是什么样的：
```bash
##左右滑动
源地址：1.1.1.1，目的地址：4.4.4.2 
就这么简单的模拟即可
```

然后H1开始要把这个数据包从自己身上发出去到路由器R1也就是H1最近的邮局，网络中我们叫做网关（gateway），那么此时H1一定是要检查的是自己的路由表，比如路由表如下：
```bash 
##左右滑动
default via 5.5.5.2 dev eth0
```

那么问题来了：H1是否可以将包发出去呢？最终包是否可以到达H2呢？这是两个完全不一样的问题。
- H1是否可以将包发出去呢？答案是可以。
- 最终包是否可以到达H2呢？答案是不可以。

H1可以通过路由表将数据包发给路由器R3，但是R3之后没有到H2的路由了，所以包最终到不了H2。

怎么解决？修改H1的路由表，使其可以将到H2 4.4.4.2的包发给正确的路由器：
```bash 
##左右滑动
default via 5.5.5.2 dev eth0
4.4.4.2/32 via 1.1.1.2 dev eth1
```
或者
```bash 
##左右滑动
default via 5.5.5.2 dev eth0
4.4.4.0/30 via 1.1.1.2 dev eth1
```
或者
```bash 
##左右滑动
default via 1.1.1.2 dev eth0
```

有什么不同？
- 方案一是唯一匹配4.4.4.2这个主机地址的明细路由；
- 方案二是通过匹配4.4.4.0/30这个网段的明细路由来选择的；
- 方案三是修改了默认路由，使用默认路由出去的。

此时如果H1想与R2中的4.4.4.1通信的话，那么方案一中的明细路由就失效了，就无法抵达4.4.4.1；方案二中依然可以使用4.4.4.0/30这条路由去到正确的路线；方案三通过默认路由也可以去到正确的路线。

这就是最简单的一个通过模拟发包来判定路由的例子，在路由的问题上，一定要明确目的地址，然后确定是否指定了正确的下一跳，下一跳就是直连网关的IP地址：一定是直连的那个跟你的接口地址在一个网段的IP。

正所谓好记性不如烂笔头，可以将各个路由节点从头到尾整理一个表，然后一个一个查看参与到路由中的节点是否有正确的路由：
![@七禾页话][3]

如何查看路由是否正确？
- Linux上直接用`ip route get <dst oip>`；
- 路由器上根据不同厂家大概会有`show route <dst ip>`、`show ip route <dst ip>`等等。

原则就是：不要管掩码，也不要纠结路由具体是怎么配置、学习到的。查看的目的只有一个：针对目的地址是否有正确的下一跳，有继续看一下，没有就死磕到有，然后继续看下一个，一直看到目的地。

再比如Kubernetes中，POD对外发起的Egress消息需要做SNAT，路由梳理就要如下图，要有SNAT和DNAT的体现，从而非常明确端到端的数据走向和路由需求：
![@七禾页话][4]

至于路由中到底要用什么路由协议，无所谓！静态还是动态，BGP还是OSPF，这个就是看方案选择了，Linux里的路由目前基本都是静态，中间路由器动态最好，因为可以动态发布学习路由，方便维护管理。

敲黑板，划重点，通信或者更大点儿说网络通信的三要素，一定要牢记，那就是路由、路由、还是XXX的路由！

得路由者得天下呀！

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eC9eFNicvUukSmrZbKibia3jicuoEeY9Jp0TplTIQM9UJgm8NhqNibWwGTjASd11OHnSb9f1F2JSYz9q5A/640?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAAboxibASqpBIiaYFfzI9GCLqrkntFJQ7EicOhqzy3SerZbh9InibmCuNvytBcy02xNiaHHztt5BpEJ8g/640?wx_fmt=png&amp;from=appmsg

[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAAboxibASqpBIiaYFfzI9GCLssTxICUxAJO6CAAkbYCy0yKvfKEyzsDmS0gVcmibFm5t3X57u5k7liag/640?wx_fmt=png&amp;from=appmsg

[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCPG56A4fKpRsIEwEswnZl0thAsQqlpJjJPL32moiaLCHpb34oLo9KO1pEfyRTaibn8IZnE73PBAowA/640?wx_fmt=png&amp;from=appmsg