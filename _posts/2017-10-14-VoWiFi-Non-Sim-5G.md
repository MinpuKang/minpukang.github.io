---
layout: post
title: Wi-Fi Calling之多设备的使用和5G时的构架
date: 2017-10-14
categories: blog
tags: [通信, WiFi]
description: 越来越多的设备，也是越来越多的羁绊！
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

妞儿趁着十一来玩儿了几天，上周末日本的三天连休就陪着妞儿游逛了富士山、再玩儿镰仓最后又看了一遍日本的夜景。

所以上周没有任何更新希望大家不要见怪。

今天继续Wi-Fi calling，首先先聊一个比较有意思的Wi-Fi calling应用——Non-sim Device也可以称为Multi-Devices的Wi-Fi calling。

然后再絮叨一下5G之后的Wi-Fi calling架构。

## 1、多终端的Wi-Fi calling
先来看一个视频，是美国T-Mobile的DIGITS的介绍(通过QuickTime Player录制和iMovies剪辑，已经上传到腾讯[https://v.qq.com/x/page/x0557btqbu8.html](https://v.qq.com/x/page/x0557btqbu8.html)：
<iframe height= 400 width= 100% src="http://v.qq.com/iframe/player.html?vid=x0557btqbu8" frameborder=0 allowfullscreen></iframe>

大家也可以点击T-Mobile官网查看[https://www.t-mobile.com/offers/t-mobile-digits?icid=WMM_TM_Q317NAVIGA_PHX8J88ZA9679](https://www.t-mobile.com/offers/t-mobile-digits?icid=WMM_TM_Q317NAVIGA_PHX8J88ZA9679)

T-Mobile的这段DIGITS视频其实说了两件事情：一个是一卡多号，一个就是接下来要说的多终端的Wi-Fi Calling，。

#### 1.1、一卡多号
现在很多的朋友应该都会有两个或者更多的手机，或者拥有双卡双待机器，为了就是解决工作生活的电话分离。

T-Mobile的一卡多号就是让一个手机卡既支持工作号码，同时也可拥有私人号码，甚至更多的私人订制号码，比如某某俱乐部专属号等等，那么这样一个手机一张卡，轻松解决工作生活分离的号码问题，是不是很有意思？

#### 1.2、多终端的Wi-Fi Calling
而对于多终端的Wi-Fi Calling也比较有意思。

苹果的生态圈已经越来越庞大了：iPhone、iPad、iWatch、iMac、Mac笔记本系列等等。

其中iPhone因为必须要插入手机Sim卡可以称为Sim-Device，而其他的都称为Non-Sim Device，也就是无卡设备(当然最新的iPad、iWatch等也开启了蜂窝网络的支持，只是仅仅支持数据业务)。

###### 1.2.1 当前的苹果实现
对于当前苹果的庞大生态系统，其也致力于一套多终端的电话接听系统，具体实现就是要所有苹果设备使用同一个apple ID、同时蓝牙开启并且在同一个Wi-Fi下，当有电话的时候iPhone和其他设备都会有振铃，这样就可以用其他设备比如iPad接电话了。

但是如果注意的话会发现，iPad等设备应该显示类似“使用iPhone通话中”的字样，原因就是此时依然是使用iPhone在接电话，只是作为一个中继器将语音转给iPad等设备，从而实现了多设备的接电话，而此时在iPhone和其他设备会有如下显示为一个**“When Nearby”**的模式：

![When Nearby][3]

从使用条件可以看出这个其实有很大的弊端：
- 1、首先必须要在同一个Wi-Fi下。
- 2、非常依赖与iPhone，iPhone关机了，其他设备也就无法接电话了。

那么怎么办呢？

###### 1.2.2 演进的多终端Wi-Fi Calling
既然苹果已经在系统上作了一部分的支持，那么是否可以通过技术手段解决弊端？在人类发展到现在的高技术支持下，答案当然是肯定的。

演进之后就是视频里展示的结果了：

![多终端Wi-Fi Calling T-Mobile][5]

同时在激活之后iPhone和其他设备会显示如下，**“When Nearby”**变成了**“On”**：

![多终端Wi-Fi Calling][6]

演进之后多个终端跟iPhone共享同一个手机号，并且完全脱离iPhone宿主机的限制，也就是iPhone可以关机、落在家里、甚至丢失，iPad等设备依然可以在Wi-Fi下**接打电话**，是不是很神奇(⁎⁍̴̛ᴗ⁍̴̛⁎)？

至于安卓阵营，我想慢慢的也会统一战线，最终可以同样的实现多终端的Wi-Fi Calling的。

## 2、5G中的Non-3GPP
由于4G时代的时候non-3GPP已经越来越成熟，所以在5G进化的道路上，没有将non-3GPP的规范丢弃，也制定了其5G的规范，在3GPP 23501-140中定义了一个叫做**N3IWF(Non-3GPP InterWorking Function)**的设备来承载non-3GPP的业务，拓扑如下：

![Wi-Fi in 5G][7]

至于具体的内容，如何接入、如何鉴权、如何建立等等，等有时间慢慢聊吧(^_^)

Wi-Fi Calling的基本内容到这篇基本就结束了，希望这五篇文章可以在通信的道路和生活上有所帮助。

Wi-Fi历史文章：

- 1、[Wi-Fi初始附着](http://www.hk314.me/blog/2017/09/16/VoWiFi-Core/)
- 2、[Wi-Fi Calling漫游问题](http://www.hk314.me/blog/2017/09/21/VoWiFi-Charging)
- 3、[从Wi-Fi到LTE的切换(Handover)](http://minpukang.github.io/blog/2017/09/24/VoWiFi-HO-from-WiFi-to-LTE/)
- 4、[从LTE到Wi-Fi的切换(Handover)](http://www.hk314.me/blog/2017/10/01/VoWiFi-HO-from-LTE-to-WiFi/)

------------
<p align="center">欢迎关注公众号：</p>
![公众号][1]

<p align="center">感觉内容不错，读后有收获？欢迎小额赞助：</p>
![赞赏][2]

  [0]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCZ6gG5NJjutfc6ZHJLrS03l9SOZbtcUVZpjg7KpA8mLsSEk8FZjlicsluXXorAoDAKFBIQWDBtr0g/0?wx_fmt=jpeg
  [1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAoGfjsaJt2NQ0a9AKmrIRoR9gKlX1I78Z4AoPtjyEPM56slw9gAQBdAHjHckbw4h93FvVVATBuLQ/0?wx_fmt=jpeg
  [2]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD3anvFetwgNHv3X1AiaXIzWPvazEMIEralm9vs42XsVfoniaXRCSkSpNpz9icsIYFgq84Eic2whLdAfg/0?wx_fmt=jpeg
  [3]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBxw1gof8mgvevg1Hr72MYG6pcOPOCCJgLslju2PaesNRXueW8PILQt0rPgOla5PZ1T3uNRlYZXxw/0?wx_fmt=jpeg
  [4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBxw1gof8mgvevg1Hr72MYGfpNYVqvMeNQLibenBrgaATLuxR3EVoYkWCMMtn4YW7zjSTpEzKDwf1A/0?wx_fmt=png
  [5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBxw1gof8mgvevg1Hr72MYGfpNYVqvMeNQLibenBrgaATLuxR3EVoYkWCMMtn4YW7zjSTpEzKDwf1A/0?wx_fmt=png
  [6]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBxw1gof8mgvevg1Hr72MYG13eVFKhmxyxcmMpseBicSz3ribCfozrTPOhC9v1r1xNAZibsNJ7iarKtpg/0?wx_fmt=jpeg
  [7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBxw1gof8mgvevg1Hr72MYGMBJEZdtWdKLicq7ic8UOqYFYBpvmf3QMrxT9ZuMohz5HOF4JkasHVnzw/0?wx_fmt=png







