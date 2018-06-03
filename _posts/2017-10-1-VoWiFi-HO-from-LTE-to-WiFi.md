---
layout: post
title: 从LTE到Wi-Fi的切换过程
date: 2017-10-01
categories: blog
tags: [通信, WiFi]
description: 我不知道下一个会话(Session)会建立在哪里？会走怎么样的路？
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

**写在最前：**公号后台收到朋友的留言给了我很多的鼓励，在此表示深深的感谢，感谢大家的支持和鼓励，看到文章能给大家带来知识的拓展、提供些许帮助，实在是我莫大的荣幸。

今天继续Wi-Fi的话题。

大家看隋唐演义，应该对程咬金印象深刻吧，程咬金使用的兵器叫做三板斧，又名“马战斧”，而他所使用的招数也被称作三斧子半，为什么呢？

因为他的招式只有三招：一斧劈脑袋，二斧鬼剔牙，三斧掏耳朵，然后在秦琼的指点下又自创了半招，而隋唐中的程咬金也靠着三板斧走天下。

那么我们Wi-Fi呢，也可以说有三板斧：

- 一斧为[Wi-Fi初始附着](http://minpukang.github.io/blog/2017/09/16/VoWiFi-Core/)。
- 二斧为[从Wi-Fi到LTE的切换(Handover)](http://minpukang.github.io/blog/2017/09/24/VoWiFi-HO-from-WiFi-to-LTE/)。
- 三斧就是今天要说的从LTE切换(Handover)到Wi-Fi的招式。

而今天要写的虽然是Wi-Fi第四篇文章，但是却是Wi-Fi三板斧里的第三斧——从LTE回到Wi-Fi的切换过程。

在从Wi-Fi到LTE的切换中，我们说其只涉及到一个IMS PDN的切换，同样从LTE到Wi-Fi也是只涉及到IMS的这一个PDN。

## 1、Handover前的拓扑和用户状态

![HO from LTE to Wi-Fi前拓扑][3]

如上图所示，首先用户已经完全注册到LTE网络，分别建立两个PDN，一个是在附着时建立的默认PDN，一般用于上网；另一个就是因为开启VoLTE之后而建立的IMS PDN，同时终端需要完成IMS网络的SIP注册。

## 2、永恒不变的PGW和PGW的选择
我们说过不管是LTE内部的还是网络间的移动性管理，只要涉及到移动性管理，千变万变但是都要有一个终究不变的就是PGW，因为PGW作为核心网侧的最后一道关口，承担着衔接外部的Internet、IMS网络重要使命，所以其上的用户Session需要在移动性管理中保持不变。

那么由于PGW要在移动性管理中保持不变，当切换到Wi-Fi的时候，要怎样保持不变么？

跟从Wi-Fi到LTE的切换类似，当用户注册到LTE的时候，每完成一个PDN的建立，MME都会通过**Notify-Request/Answer**的流程将此PDN所使用的PGW ID(FQDN)上报给HSS，也就是下图中红框内的AVP：

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAvIAp9Yfa68OQetd71duNR9KX3GjOtOTluLgD7kYLA6giaDe96iaV4yJRX3oAVchV3EBB4VQlwShug/0?wx_fmt=jpeg" width="80%" height="50%" />

当用户切换到Wi-Fi的时候，HSS通过AAA将此信息同步下发给ePDG，然后ePDG根据自己的设计实现，可以有如下两种DNS查询方式：

+ 第一种是直接使用整个PGW ID(FQDN)进行一个A/AAAA的DNS查询，但是弊端就是ePDG无法知道此PGW是否支持S2b的Service。
+ 第二种就是拿到这个PGW ID(FQDN)后，根据协议规则，取出ID(FQDN)中的**canonical-node-name**，然后进行一个NAPTR的DNS查询，这样ePDG收到返回结果之后需要进行服务匹配(**x-3gpp-pgw:x-s2b-gtp**，以PGW使用GTP协议为例)成功之后，最终再得到PGW的S2b IP。
+ 通过以上的过程，ePDG最终得到PGW的S2b VIP。

#### 2.1 什么是canonical-node-name
3GPP 29.303中规定了如下FQDN格式：

    <"topon" | "topoff">.<single-label-interface-name>.<canonical-node-name>

比如一个FQDN为**topon.pgw-s5s8s2b.epg0.epc.mnc045.mcc123.3gppnetwork.org**，那么他的canonical-node-name就是**epg0.epc.mnc045.mcc123.3gppnetwork.org**。

## 3、从LTE到Wi-Fi的切换流程
接下来梳理一下从LTE到Wi-Fi的Handover的信令流程，结合了3GPP和自己测试的总结，在某一些信令流程的时候各个厂家的实现可能会不太一样，但是步骤应该不会缺少，信令流程图如下：

![HO from LTE to WiFi 信令1][4]

![HO from LTE to WiFi 信令2][5]

#### 3.1、Precondition(前提)：
切换前提就是用户已经完成了LTE的注册并且完成了IMS PDN的连接和IMS网络的注册，同时如**章节3**所说，在IMS PDN连接完成后，MME必须要将IMS所使用的PGW ID(FQDN)通过Notify消息上报给HSS。

#### 3.2、主要流程
如果已经读过[Wi-Fi初始附着](http://minpukang.github.io/blog/2017/09/16/VoWiFi-Core/)的话，可以看到从LTE到Wi-Fi切换之后的信令基本无差，所以不过多叙述，强调一下跟初始附着不同的地方：

- 1、第一个不同就是在第一个**IKE_AUTH MID=01 Request**中，也就是信令图中的第3步，用户需要将在LTE的时候PGW分配的IP地址，我们也称其为Wi-Fi中的内层地址上报给ePDG，但是在初始附着的时候必然不会带任何的内层地址，如下图所示(这个是一个残缺数据包，因为理论上在**Payload: Configuration (47)**中会同步请求P-CSCF的地址，当然这个是终端的行为，都可以通过刷机等手段改变)：

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAvIAp9Yfa68OQetd71duNRRvqYgUarNFrXUtqeryU2OYibAVJIrmmr5n3ic72v8n3Hf9dSribiaChKaA/0?wx_fmt=jpeg" width="80%" height="50%" />

- 2、第二个不同就是，在切换的时候，第二个**Diameter-EAP Answer**中，也就是信令图中的第15步，AAA必须要将HSS中下发的APN(IMS)对应的PGW ID(FQDN)下发给ePDG，如下图所示：

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAvIAp9Yfa68OQetd71duNRqwCkoE2DMORiamIHpwkb5w52ldhEavOicXMf48QX1Xs4NHd7qcptfrCA/0?wx_fmt=jpeg" width="80%" height="50%" />

- 3、ePDG通过**IKE_AUTH MID=01 Request**判断出这是一个切换过程，通过第15步的**Diameter-EAP Answer**得到了用户在LTE时候使用的PGW ID(FQDN)，接下来第18步，ePDG发起了DNS的请求来查找PGW的S2b VIP，而此处必须使用PGW ID(FQDN)进行DNS查询，而不是APN的FQDN，具体内容已经在**章节3**中详述。

- 4、ePDG通过以上步骤得到了PGW的S2b VIP，在信令图第19步开始发出一个带有切换标志的**Create Session Request**给PGW，如图所示：

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAvIAp9Yfa68OQetd71duNRmyIIPj9HQQIADSC2iaIibETyIURmI1OCtQmSVHbt5ibTia7SoXZ6deT1hA/0?wx_fmt=jpeg" width="80%" height="50%" />

以上四个地方是从LTE切换过来的时候跟在Wi-Fi下的初始附着不同的地方，其余信令基本一致。

#### 3.3、释放LTE中的Session
由于PGW已经完成了Wi-Fi的Session更新，因此发起了一个**RAT changed from 3GPP to Non-3GPP**原因值的LTE Session释放以通知SGW/MME/eNodeB来释放资源。

#### 3.4、IMS网络的更新
在用户已经完成Handover的主要信令，需要迅速通过SIP消息更新IMS网络的Session。

同时如果用户是带有电话的切换，此时，可能需要建立Wi-Fi中的专载来承载语音业务，从而保证用户从LTE到Wi-Fi切换后所有语音业务不间断。

## 4、Handover后拓扑和用户状态
Handover完成后用户依然有两个PDN，一个是一直在LTE中的上网PDN，另一个就是切换到Wi-Fi之后的IMS PDN，之后的用户Session状态如下图所示：

![HO from LTE to Wi-Fi后的拓扑][6]

以上就是LTE到Wi-Fi的Handover(切换)内容，欢迎公众号留言讨论。

------------
<p align="center">欢迎关注公众号：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD1g2UjYu4VfCGHmbhgVqOAnNnJQfN7ZhRVUCopYOsfpPtIEB95VNEqu8trAxJXzGDg01ka6z6wzQ/0?wx_fmt=jpeg" width="30%" />

<p align="center">感觉内容不错，读后有收获？欢迎小额赞助：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAzA577Ce49rCLiby9EtT195GRiaqKCT6QCQ5Weia9OZD72MJz4ABlqAy1gbHepk5hHM464hCiarQRI7w/0?wx_fmt=jpeg" width="30%" />

  [0]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCZ6gG5NJjutfc6ZHJLrS03l9SOZbtcUVZpjg7KpA8mLsSEk8FZjlicsluXXorAoDAKFBIQWDBtr0g/0?wx_fmt=jpeg
  [1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAoGfjsaJt2NQ0a9AKmrIRoR9gKlX1I78Z4AoPtjyEPM56slw9gAQBdAHjHckbw4h93FvVVATBuLQ/0?wx_fmt=jpeg
  [2]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD3anvFetwgNHv3X1AiaXIzWPvazEMIEralm9vs42XsVfoniaXRCSkSpNpz9icsIYFgq84Eic2whLdAfg/0?wx_fmt=jpeg
  [3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDPleibln3nibwV8ZWQC8VfMTOmvqajZyGG561mGwciaq7Pliaj5vIyhqRtHUcNZfTOrRmQMcZiceW2cCg/0?wx_fmt=png
  [4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAvIAp9Yfa68OQetd71duNRGSIZGwMA9cias4HF6pJLYaByMHIm4p7csxbNOCqk8O4N8sfffnsj9Yw/0?wx_fmt=png
  [5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAvIAp9Yfa68OQetd71duNR1FvLueENEiaOKibyF353kiabrdMcR5Gt2V4y4Hltd8tt3vZXFz7JfYWBw/0?wx_fmt=png
  [6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAvIAp9Yfa68OQetd71duNRRnDQqLQKkXrmibde3UOy2JwkJePyczZcqx8rY98obNrIbOicCR3QXbUg/0?wx_fmt=png






