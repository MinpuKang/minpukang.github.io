---
layout:     post
title:      通信｜5GC信令干货资料分享
date:       2021-04-25
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 通信
    - 5G
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

5G的概念已经炒很久了，而且从全民感受度来看比4G的时候要普及不少，不管是通信行业的从业者，还是其他行业的工作人员，即便是街道办事处的大爷大妈们，你问一下知道5G否，都能回答：当然知道。这个依赖于国家在5G上渐趋领先地位，打破了之前几代通信系统一直被欧美国家领先的传统，然后就是最近这两年华为被美国制裁，孟晚舟女士依然扣留加拿大，然后备胎转正、芯片自研等等诸多事件，导致了全民皆5G。

但是要再问问5G有啥优点，那就一个字，我只说一次：快。然后也就没啥5G的概念了。最近面试的时候，大家的回答也基本就是快字当头，别的不懂。

![1][1]

从我本人作为一个通信从业人员来说，5G确实是快字为先，毕竟要一代更比一代强，这会导致很多的产业升级。之前听一个分享会有一个码头的案例，现在很多码头已经非常智能化了，比如装卸集装箱这个任务早已开始自动化，但是目前自动化依赖于有线和无线宽带的技术，光纤如同神经末梢般铺设在码头从而达到准确控制分拣等自动化的目的，但是光纤的维护费用和磨损率很高，导致这样一套自动化系统维护费用是水涨船高。

但是如果引入5G呢，架设5G天线，然后设备通过无线接入通信网络，5G的速度快、低时延等等优势，可以迅速替代光纤宽带，从而降低成本。

![2][2]

然后就是业务类型的多样化，现在有一个概念是VR和AR，一个是虚拟现实VR，一个是增强现实AR。拿看房子这件事来说，现在普遍都是一堆图片，然后看好约时间上门看，一个是图片作为平面媒介，没有立体感，也不直观，然后上门看浪费时间，所以借助VR/AR戴个镜子先体验一遍，大平层、海景房，仿佛置身其中。

再一个5G可以“私人订制”，原来2、3、4G的时候，运营商一张网，千万业务共享。虽然4G中的DECORE/eDECORE已经开始引入这种私人订制，但是5G中开始发挥的淋漓尽致，比如边缘计算MEC，车联网V2V（车与车）、V2I（车与基础设施）、V2C（车与云），AGV（Automated Guided Vehicle）自动导引运输车，等等都可以私人订制网络，比如号称全自动的京东供货中心，完全可以打造一个自己的5G通信网络，造船厂也可以拥有自己独立的通信网络，不过这些依然需要通过运营商铺设维护、设备商提供设备和技术。

当然，当前5G建设不管是从技术还是场景，还没有非常清晰明确，反正就是亦步亦趋、静默花开吧。

![3][3]

前面说了很多虚头巴脑的东西，然鹅这些场景依然是上层应用的天下，作为通信行业来说，依旧是管道工的角色，我们的目的就是怎么让管道看起来更漂亮、更高大、更耐用、更顺畅。

所以回到今天的正题，5G里的管道是怎么建立的？又是怎样维护的？这涉及到一个信令流程的问题。其实吧，这个信令流程跟之前的并没有什么实质性的变化，依然是要完成三个A，至于哪三个？欢迎移步：[通信｜新人如何学信令？](https://mp.weixin.qq.com/s/pZPtYmCF-1WrUcuRneLECg)

只是完成这三个A的内容有了变化，比如核心网里不再区分什么Diameter，GTP，统一都是SBI走HTTP2(你看了真的会吐)。

今天整理电脑，发现了之前得到的一个slide，就是介绍5GC相关的信令，已经忘记从哪儿获取的，也没有发现原作者是谁。

网上搜索了相关内容，有两个署名：一个是通信某设备商公司，一个是PPT交流分享。以通信人的角度来说，原作者多半也就是通信某设备商公司的某位大神了。

原slide一共78页，涵盖了3GPP 23.502标准里大部分​的5G信令流程，包括注册、PDU建立、PDU更新、PDU释放，去注册，NF的选择，分片的介绍，4-5G互操作，语音解决方案，5G短消息SMS的解决方案，等等诸多内容。

简单截两张图大家感受一下：

![4][4]

![5][5]

从头到尾基本看了一遍，内容非常全面、详实，然后根据自己的测试经验简单的添加了一些内容。

比如一些缩略语和相关内容的批注：

![6][6]

![7][7]

流程介绍上的补充：

![8][8]

​增加了SSC1的流程示意图：

![9][9]

所以现在的slide变成了79页。

整体上没有设计到任何厂商，这应该也是最开始作者公开分享的原因吧。

如何获得slide？关注公众号，后台回复“5GC“即可（不区分大小写）。

如果原作者觉得不妥，可以联系删除。

更多通信内容，欢迎移步话题：[通信](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1343503379909394432&__biz=MzIxNjIyNzM0Mw==#wechat_redirect)

------------
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)，旅行、摄影。。。偶尔分享技术周边</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[1]:https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCmkzQkG7ibHVwicylGGR4tn3odKwnj2cj3xibTMH68FqY0RictibhQPhOLblFTQH0NsVGZAiaf1aX5NU1Q/0?wx_fmt=jpeg


[2]:https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCmkzQkG7ibHVwicylGGR4tn3XPsOsAGwgDq0zFudUGq9ZgdlCKDLx3BpAfMKeJicnoia9qJgFXFU6xZQ/0?wx_fmt=jpeg


[3]:https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCmkzQkG7ibHVwicylGGR4tn3wTKA02MGydWhCfhpqickVIn5217DrnEtodbCEvPlOGoTgIdJP6d5FjQ/0?wx_fmt=jpeg


[4]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCmkzQkG7ibHVwicylGGR4tn33pSMPPffaZ24p7nLMdsoHAJ111AoHVWnPC7QzRj6lESKS08iar0W0UQ/0?wx_fmt=png


[5]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCmkzQkG7ibHVwicylGGR4tn3zO33s22HM5CoKy7VMKVJjEMdhSibHFzXeyyhOIAVKFpCfa0UicwrWTEg/0?wx_fmt=png


[6]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCmkzQkG7ibHVwicylGGR4tn3T7jfX0xxAeNsRwXtt30na90icuGWEhj602u6StI866xJplJwdy7HibRw/0?wx_fmt=png


[7]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCmkzQkG7ibHVwicylGGR4tn3piaf0UvribWhuhBI5C7eR2fNCsx6vDiartJ1dCxODaDQ5mSnfkzVf5G1A/0?wx_fmt=png


[8]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCmkzQkG7ibHVwicylGGR4tn3ibWF8APtPIfibzd19zdbp37tN6TjNicnUicOAStRlnH7lfiaQkjNBBHiaqJQ/0?wx_fmt=png


[9]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCmkzQkG7ibHVwicylGGR4tn3kROAUM7hlwmIoxJ5TJ0ib4hiaqKljsnqNZc8hJmW0sC0NDQOx3XN7TUw/0?wx_fmt=png


