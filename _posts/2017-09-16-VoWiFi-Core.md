---
layout: post
title: 咦？我用Wi-Fi打了个Call
date: 2017-09-16
categories: blog
tags: [通信, WiFi]
description: 既然Wi-Fi无处不在，那么也让它来帮我们做最快最稳的连接吧！
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

好久没有写通信技术贴了，最近一直在做Wi-Fi相关的东西，所以心血来潮的整理一下。

伴随着5G、云和虚拟化的到来，通信也迎接着新的变革，从最开始的语音业务，到之后2、3G加入数据业务，到4G控制和用户面的基本分离，再到5G提出的彻底分离空着和用户面，但是不管构架和概念是怎么变化，通信业务的宗旨是不会变的：我们就是要提供稳定的网络业务包括主要的语音业务、数据业务以及之后的物联网等等。

那么作为通信最基础的一个语音业务，在4G中一共有三种方案，分别是网络测两种和终端测一种：

- 网络测：
- 1、最开始的CSFB——Circuit Switched FallBack——是通过在通过联合附着在4G和CS域，在打电话或者接电话的时候回落到2/3G的CS域来实现。
- 2、之后的演进就是伴随着IMS网络的建立而提供的VoLTE——Voice over LTE，从而实现4G中可以直接接打电话，同时也可以上网，最终达到上网和电话可以同步进行的目的；但是受限于4G的信号覆盖，所以VoLTE需要SRVCC的同步支持，也就是在4G覆盖质量差的时候依然要借助2/3G来完成语音业务。
- 终端测：
- 3、终端测也就是在手机上做手脚，也就是俗称的双待机（非双卡双待），这种终端是一卡注册到2G或者3G，同时注册在4G，这样4G用来上网，2/3G的注册用于语音短信等业务，比如我之前使用的HTC 816t，但是弊端就是成本增加并且费电。

而伴随着无处不在的Wi-Fi，同时借助IMS网络，我们又提出了一种在Wi-Fi下直接打电话的新方案，也就是VoWiFi——Voice over WiFi，也可以叫做Wi-Fi Calling。

Wi-Fi Calling已经有3、4年的基础了，所以技术上也越来越成熟，功能也越来越丰富，最主要的是它已经被越来越多的运营商采纳并部署，那么今天就简单的聊一聊Wi-Fi Calling的基本构架和信令流程。

## 1. VoWiFi vs VoIP
说到WiFi的语音，必然要涉及到一直存在的VoIP（Voice over Internet Protocol），也就是OTT（Over the top）业务，通过使用微信、QQ、Skype、Whatapp等等APP来进行的语音视频等。

那么问题来了，既然已经有了这么多的语音视频软件，为什么还要VoWiFi呢？原因可以总结为以下几点：

- a、VoIP必须要下载APP才可以使用；VoWiFi手机可原生支持。
- b、VoIP的APP必须注册，并且大部分都是互相添加好友之后才可以联系，而且大部分APP都是封闭生态圈的，不同的APP之间目前还不能互通；VoWiFi既然手机原生支持，所以不存在APP的弱点，使用运营商的开户手机卡，互相拨打手机号即可。
- c、VoIP的语音视频质量无法保证，完全依赖当前网络质量；而VoWiFi跟VoLTE一样，运营商提供Qos质量保证，以自己测试体验来说VoWiFi的视频质量完全是高清显示，而且无卡顿。
- d、如果离开WiFi环境下，VoIP也就断了或者需要切换到流量模式下使用；而VoWiFi可以跟VoLTE实现完美切换，从而可以保证离开WiFi环境下依然保持高效的语音视频聊天。

## 2. Wi-Fi Calling拓扑图
Wi-Fi Calling不管是理解为4G语音的一个补充，还是独立存在的一个新的方案，但是它依然有着自己的一套拓扑和新节点，拓扑如下，红色接口为新引入的接口，那么接口两端的节点就是WiFi Calling中不可或缺的：

![Wi-Fi Calling拓扑图][3]

#### 2.1 无线接入
Wi-Fi Calling又称为Untrusted Wi-Fi，因为这套方案的无线接入既可以是运营商自己部署AP（Access Point），也可以是各个Wi-Fi公司部署的无线比如上海的iShanghai、花生地铁，辽宁的iLiaoning，万达商场里的无线，星巴克的无线，机场的无线，还有就是自己家里办宽带之后所使用的无线路由器。

所以Wi-Fi Calling的无线接入就是现在无处不在并且被大家没事儿就问密码的可以连接互联网的无线网络。

#### 2.2 新旧核心设备
如2/3G的语音需要MSC、4G需要MME一样，Wi-Fi Calling里也引进了一个新的核心设备ePDG（Evolved Packet Data Gateway），如同MME的作用一样，是无线测和网络册的连接核心，但是更像2/3G的SGSN，因为也要承载用户面的数据。

然后是存储鉴权数据的HSS，同时也定义了用户的信息比如IMSI、MSIDN和APN以及Qos的相关数据。

之后就是用于连接ePDG和HSS的一个AAA server（Authentication, Authorization and Accounting），用来作为鉴权的桥梁来翻译HSS提供的数据给ePDG。

再之后就是作为核心网和IMS的衔接点，依然使用4G中的PGW。

其他就是配合这整套方案所存在的节点比如eDNS——ePDG查找PGW，Public DNS——手机查找ePDG。

#### 2.3 终端的IP
我们知道在2/3/4G的数据业务的时候终端最终只需要一个IP，就是GGSN/PGW分配用于上网或者跟IMS通信的地址。

在Wi-Fi中一样，也需要PGW分配一个用于跟IMS通信的地址，我们一般可以称之为内层地址，也就是完全用于用户面的地址。

然后呢，Wi-Fi对应内层地址还有一个地址可以称为外层地址，这个就是拓扑图中DHCP分配的地址，当然这个DHCP可以是完全独立的一个DHCP server，也可以是具有DHCP功能的无线路由器。

外层地址有两个作用，一个是用于查询ePDG SWu地址的时候跟Public DNS通信，二是用于跟ePDG通信建立SWu IPSec隧道。

## 3. Wi-Fi附着信令
说完拓扑，我们来聊聊信令，首先先来看看信令图，下图中表示VoWiFi的核心网信令，包括SWu IPsec隧道的建立、用户鉴权和GTP隧道建立这三部分，在此之前还需要一些基本的准备工作，接下来我们简单的梳理一下：

![Wi-Fi Calling信令图][4]

#### 3.1 前景提要
手机要发起上图第一条IKE之前是经历漫长的准备过程的，首先当然是手机支持Wi-Fi Calling这个功能；其次手机需要通过运营商和终端设备商的合作配置一个ePDG的FQDN，目前商用所使用的FQDN格式如下：

    epdg.epc.mnc<MNC>.mcc<MCC>.pub.3gppnetwork.org

MCC+MNC即PLMN定义了此ePDG所属的运营商是谁。

** Note：** 当然FQDN不是完全固定的，比如在测试阶段，可以随意规划，只要定义的域名能成功解析到ePDG的SWu IP即可，再或者直接在手机中配置ePDG的SWu IP地址都是可以的。

手机配置完毕在连接WiFi之后，手机通过DHCP从DHCP server中拿到外层地址，同时得到DNS server的IP。

然后手机向DNS server发起ePDG FQDN的解析请求，从而得到一个或者一系列ePDG SWu的IP地址。

#### 3.2 主体信令
当以上工作完毕之后，手机开始建立IPsec隧道：

- 1、UE（User Device即终端）发送 **IKE_SA_INIT MID=00** 的请求到ePDG，用来交互IKE的各种算法。
- 2、ePDG回复 **IKE_SA_INIT MID=00 Response** 给UE来确认算法。
- 3、然后UE发送第一个AUTH请求 **IKE_AUTH MID=01 Request** 消息给ePDG，其中包括了终端User-Name（内含IMSI）、APN、以及终端请求的内层地址的IP类型/DNS/P-CSCF，同时还有终端支持的IPsec算法
- 4、ePDG收到UE的请求之后开启网络测鉴权流程，ePDG向AAA发送 **Diameter-EAP-Request（DER）** 消息。
- 5、AAA收到DER之后，因为本地没有任何的用户数据，因此给HSS发送一条 **Multimedia-Auth Request（MAR）** 用来请求鉴权向量。
- 6、HSS收到MAR之后检测用户已定义之后回复带有鉴权向量的 **Multimedia-Auth Answer（MAA）** 给AAA。
- 7、AAA收到鉴权向量之后，立即发送一个 **Server-Assignment Request（SAR）** 到HSS来请求用户数据包括APN、Qos信息等。
- 8、HSS回复带有用户数据的 **Server-Assignment Answer（SAA）给AAA**。
- 9、AAA收到SAR之后给回复 **Diameter-EAP Answer（DEA）** 给ePDG，其中包括鉴权向量，并且EAP-AKA Subtype为AKA-Challenge，Result-Code为DIAMETER_MULTI_ROUND_AUTH。
- 10、到此，ePDG回复 **IKE_AUTH MID=01** 给UE，并且包括鉴权数据以及Challenge的要求。
- 11、UE收到MID=01的回复之后发送 **IKE_AUTH MID=02** 以响应来自网络测的Challenge。
- 12、ePDG收到MID=02之后将UE提供的鉴权数据通过 **Diameter-EAP Request（DER）** 消息提供给AAA。
- 13、AAA完成鉴权后通过 **Server-Assignment Request（SAR）** 通知HSS：用户鉴权通过。
- 14、HSS发送 **Server-Assignment Answer（SAA）** 消息给AAA。
- 15、AAA收到SAR之后，发送 **Diameter-EAP Answer（DEA）** 到ePDG，其中包括鉴权成功标签、用户信息IMSI/MSISDN等、APN信息以及对应的Qos，其中**QCI**是**5**。
- 16、ePDG收到AAA的消息之后确认信息无误，回应 **IKE_AUTH MID=02** 给UE来告知其鉴权成功。
- 17、UE收到MID=02之后很是兴奋，终于得到了核心网的认可，立马发送 **IKE_AUTH MID=03** 请求到ePDG来开启下一步GTP隧道的建立。
- 18、ePDG收到UE的MID=03之后，构建用户APN的FQDN，并且到DNS中查询PGW的S2b VIP。
- 19、ePDG收到DNS回复的IP并且确认service正确之后，发送 **Create Session Request** 到PGW开始建立用户的GTP隧道，其中**RAT-Type**为**WLAN**。
- 20、PGW收到Create Session Request之后通过**Credit-Control-Request Initial（CCR-I）**通知PCRF建立用户的IP-CAN-Session，并得到其响应的**Credit-Control-Answer Initial（CCA-I）**消息。
- 21、如果S6b定义，PGW发 **AA Request（AAR）** 到AAA来建立S6b Session，其中包括PGW的FQDN。
- 22、AAA将PGW的FQDN通过 **Server Assignment Request（SAR）** 更新给HSS。
- 23、HSS回复 **Server-Assignment Answer（SAA）** 给AAA来确实更新完成。
- 24、AAA回复 **AA Answer（AAA）** 给PGW。
- 25、一切信令完成，本地资源分配完成，PGW回复 **Create Session Response** 给ePDG来完成GTP隧道的建立，包括给UE分配的内层地址以及DNS/P-CSCF等信息。
- 26、GTP隧道建立完毕，ePDG回 **IKE_AUTH MID=03** 给UE，并将内层地址、DNS/P-CSCF等信息告知UE。
- 27、到此核心网信令完成，为UE准备的IPsec和GTP隧道建立完毕，接下来UE可以发起SIP请求来完成IMS网络的注册，以便可以接打电话。

以上就是Wi-Fi Calling的基本内容包括拓扑节点和附着信令，如有问题可以关注公众号——七禾页话——留言讨论。

------------
<p align="center">欢迎关注公众号：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD1g2UjYu4VfCGHmbhgVqOAnNnJQfN7ZhRVUCopYOsfpPtIEB95VNEqu8trAxJXzGDg01ka6z6wzQ/0?wx_fmt=jpeg" width="30%" />

<p align="center">感觉内容不错，读后有收获？欢迎小额赞助：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAzA577Ce49rCLiby9EtT195GRiaqKCT6QCQ5Weia9OZD72MJz4ABlqAy1gbHepk5hHM464hCiarQRI7w/0?wx_fmt=jpeg" width="30%" />

  [0]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCZ6gG5NJjutfc6ZHJLrS03l9SOZbtcUVZpjg7KpA8mLsSEk8FZjlicsluXXorAoDAKFBIQWDBtr0g/0?wx_fmt=jpeg
  [1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAoGfjsaJt2NQ0a9AKmrIRoR9gKlX1I78Z4AoPtjyEPM56slw9gAQBdAHjHckbw4h93FvVVATBuLQ/0?wx_fmt=jpeg
  [2]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD3anvFetwgNHv3X1AiaXIzWPvazEMIEralm9vs42XsVfoniaXRCSkSpNpz9icsIYFgq84Eic2whLdAfg/0?wx_fmt=jpeg
  [3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCb5jqPzRL7OicNOVfFibtqz12YFMWiarQb4hAdIibaePpGbZcLRz2MgbJsjtBQ9P7SZQyJ5hezaa2YMA/0?wx_fmt=png
  [4]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCb5jqPzRL7OicNOVfFibtqz1TxUZsGBeEicUN4Q3eLsZnRtWbIxbtxDjfGRDnJrVaFWfwTch2fibhM6Q/0?wx_fmt=jpeg



