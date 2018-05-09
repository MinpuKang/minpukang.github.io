---
layout: post
title: 从Wi-Fi到LTE的切换过程
date: 2017-09-24
categories: blog
tags: [通信, WiFi]
description: 移动是通信中必然和不可或缺的过程！
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

这是Wi-Fi Calling的第三篇，聊一聊用户是怎么从Wi-Fi Handover(切换)到LTE的。

之前我们说VoWiFi这套方案作为VoLTE业务的补充，从而解决覆盖死角等等不足，因此也必然需要跟LTE进行无缝切换才能达到最完美的补充。

首先我们称这套方案为VoWiFi，也就是说是针对语音业务的，那么也就只需要IMS这一个APN的PDN就足够了，当然多PDN也是支持的，比如xcap业务、彩信业务等等都可以走在Wi-Fi的IPsec这条道路上，但是其他的PDN可以说是用完即走——也就是业务做完通道就拆掉了。

至于LTE中默认的那个上网APN目前是永远不会建立到Wi-Fi中，而且也没有必要，因为我们说VoWiFi所使用的无线接入就是可以跟互联网连接的WiFi，那么当然切换到WiFi的时候上网就不再需要通信的PDN连接了。

因此从WiFi到LTE的切换也就是只涉及到IMS这个APN的PDN，所以也可以称为是一个PDN的切换过程。

## 1、Handover前拓扑和用户状态

![HO from WiFi to LTE前拓扑][3]

Handover的网络拓扑图上图所示，不再过多解释。

需要注意的是在切换之前的用户状态，绿色的表示用户在LTE下的Session，紫色的表示在Wi-Fi下的Session，也就是说在发生WiFi到LTE切换之前是需要用户同时注册在LTE和WiFi上的，而且LTE上的Session也就是其LTE附着时候建立的默认上网的PDN，WiFi下的Session就是IMS的PDN来承载语音业务的。

## 2、Handover中的变与不变
我们说世事无常，也就是说世事的变化性无法琢磨，比如人体的细胞由于新陈代谢每三个月会替换一次，随着旧细胞的死去，新细胞华丽诞生。由于不同细胞代谢的时间和间隔的不同，将一身细胞全部换掉，需要七年。也就是说，在生理上，我们每七年就是另外一个人，那么七年你变了，但是又没变，变的是无法感知的细胞，而不变的是你的经历和记忆。

那么在网络切换的时候也是一样，有变化的地方，也有不变的地方：

- 变的地方就是接入方式从Wi-Fi变到LTE，那么所承载用户IMS的Session的网元也变成了LTE下的eNodeB、MME、SGW。
- 不变的地方是PGW和IMS网络不变，首先从核心网测来说PGW是在所有不管是怎么样的一种Handover切换都不可以改变的，就目前来说，如果一旦改变，那么Session必然不再连续，这也是切换失败的一个体现；然后IMS网络不变的意思是切换前后用户在IMS网络里的Session是不变的，这个很好理解，如果Session变了，那么如果保证语音的持续性呢？

## 3、如何保证PGW不变
通过拓扑结构可以看到在LTE中PGW使用的是S5/S8 service，而在WiFi中使用的是S2b service，那么在Handover切换的过程中如何保证PGW不变呢？
总结起来有两种方案：

- a、通过判断PGW ID(FQDN)来确保其不变。
- b、通过直接判断PGW Service IP来确保其不变。

不管是方案a还是b，在第一篇[Wi-Fi附着](http://minpukang.github.io/blog/2017/09/16/VoWiFi-Core/)中介绍过当用户在WiFi附着的时候PGW都要通过S6b这个接口将其ID(FQDN)或者IP等信息更新给AAA最终更新到HSS，然后HSS再将其插入到MME中，这样MME收到Handover请求的时候就可以知道用户在Wi-Fi的时候使用的PGW了。

## 4、Handover信令流程
最后来梳理一下Handover过程的信令流程，结合了3GPP和自己测试的总结，在某一些信令流程的时候各个厂家的实现可能会不太一样，但是步骤应该不会缺少，信令流程图如下：

![HO from WiFi to LTE 信令1][4]

![HO from WiFi to LTE 信令2][5]

#### 4.1、Precondition(前提)：
首先需要用户已经完成Wi-Fi注册，打不打着电话都无所谓。

LTE注册要求：

- 可以在WiFi注册的同时，LTE中的默认承载同步建立，这种情况就是信令图中的Precondition，当用户完成Wi-Fi注册的时候，PGW更新自己的ID到HSS之后，HSS通过Insert消息下发给MME。
- 如果打开飞行模式注册的Wi-Fi，依然PGW需要将自己的ID更新到HSS，在关闭飞行模式的瞬间，手机可以完成LTE的注册，这样IMS所使用的PGW ID是在HSS的Update Location Answer中通知给了MME。

#### 4.2、主要流程
主要流程也就是当用户在离开Wi-Fi的瞬间开启了到LTE的切换，由于用户已经附着到了LTE，并且建立了上网APN的默认承载，此时用户只需要将IMS的PDN迁移到LTE即可：

- 1、用户发起**PDN Connectivity Request**的请求，其中需要注意的是Request Type是**Handover**，如下图所示：

<img src="https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjdbsB0oFMfribQXLhQVCMadKqQKxNe09oOB1UVQR1icLJ6UnYGwiazORIkA/0?wx_fmt=png" width="60%" height="60%" />

- 2、当MME收到一个带有**Handover** Type的PDN建立请求之后，需要得到其在Wi-Fi时使用的PGW的IP地址，因此从PDN建立请求中获取APN，并得到HSS提供的此APN下的**AVP: MIP6-Agent-Info**中的PGW信息：
    + 如果是**AVP: MIP-Home-Agent-Address**，也就是一个PGW IP地址，那么MME可以直接发送** Create Session Request**来建立GTP Tunnel。
    + 如果是**AVP: MIP-Home-Agent-Host**，也就是一个PGW ID(FQDN)，MME可以有如下两种行为：
        - 第一种是直接使用整个PGW ID进行一个A/AAAA的DNS查询，但是弊端就是MME无法知道此PGW是否支持S5/S8的Service。
        - 第二种就是拿到这个PGW ID后，根据协议规则，取出ID中的**canonical-node-name**，然后进行一个NAPTR的DNS查询，这样MME收到返回结果之后需要进行服务匹配(**x-3gpp-pgw:x-s5-gtp:x-s8-gtp**，以PGW使用GTP协议为例)成功之后，最终再得到PGW的S5/S8 IP。
    + 通过以上的过程，MME最终得到PGW的S5/S8 IP地址，同时SGW的S11 IP地址已经在一开始的LTE附着获取到了。

- 3、MME开始GTP Tunnel的建立，发送一个**Handover Indication**为**True**的Create Session Request到SGW，然后SGW经过信息更新(SGW的S5/S8信息)之后发给PGW，Handover标签如下图所示：

<img src="https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjdL6l9E8e49VfCnhnGnHkGuY5IIgAhicHHXjj8f2TNe1WJ6aCcYLTHxeA/0?wx_fmt=png" width="60%" height="60%" />

- 4、PGW收到切换的Create Session Request之后，可能需要进行Gx接口的IP-Can-Session的更新并得到PCRF的确认。

- 5、PGW回复成功的**Create Session Response**给SGW，然后SGW更新信息(SGW的S11/S1U信息)后发给MME。

- 6、MME将成功的信息通过**Activate default EPS bearer context request**的NAS消息回复给UE，包括APN、Qos、UE IP等信息，同时将SGW的S1U信息(IP和TEID)告知给eNodeB.

- 7、UE收到Request之后进行各种完整性校验确认无误之后，回复**Activate default EPS bearer context accept**的NAS消息通过eNodeB转发给MME，同时eNodeB也将其S1U的信息(IP和TEID)封装后一并上交给MME。

- 8、MME收到Accept和eNodeB的S1U信息后，将信息通过**Modify Bearer**消息通知给SGW。

- 9、SGW回复**Modify Bearer Response**给MME确认更新完毕。

#### 4.3、释放Wi-Fi中的Session
PGW完成LTE的Session建立之后，发起Wi-Fi Session的释放，这个释放的过程可能会有所不同，但是主要信令可以总结为如下几步：

- 10、PGW通过**Session-Terminate**的信息通知AAA释放S6b Session。
- 11、PGW发送**Delete Bearer Request**给ePDG.
- 12、ePDG收到消息之后发送**Session-Terminate-Request**给AAA来释放SWm Session.
- 13、AAA收到Terminate之后通过带有类似USER_DEREGITRATION标签的Server-Assignment的消息来通知HSS释放用户的Wi-Fi Session信息。
- 14、AAA收到HSS的确认消息之后，发送**Session-Terminate-Answer**给ePDG来告知AAA/HSS已经完成Wi-Fi Session的释放。
- 15、ePDG回**Delete Bearer Response**给PGW以确认Wi-Fi Session释放成功。
- 16、为了同步网络侧和用户侧的Session，ePDG通过带有**Delete**标签的**Information Request**通知UE释放其Wi-Fi Session。
- 17、但是由于UE已经离开Wi-Fi，所以delete的消息可能不会到达UE，所以UE的确认可能是收不到的。

#### 4.4、IMS网络的更新
在用户已经完成Handover的主要信令，也就是完成LTE侧的消息之后，需要迅速通过SIP消息更新IMS网络的Session。

同时如果用户是带有电话的切换，此时，需要建立LTE中的专载来承载语音业务，从而保证用户从Wi-Fi切换到LTE之后所有语音业务不间断。

## 5、Handover后拓扑和用户状态
Handover完成后用户依然有两个PDN，一个是一直在LTE中的上网PDN，另一个就是从Wi-Fi切换过来的IMS的PDN，之后的用户Session状态如下图所示：

![HO from WiFi to LTE前拓扑][9]

以上就是Wi-Fi到LTE的Handover(切换)内容，欢迎公众号留言讨论。

PS：最近迷上了晚霞和星芒，周末两天终于把这个Handover整理完了，所以下午去台场亲近一下大自然顺便参观了一下日本的小自由女神像，然后得到了下边这种晚霞下的台场夜景，星芒效果超级喜欢(不过场景太乱了【捂脸】)：

![台场晚霞][10]

最后送上四只超级萌犬：

![四只萌犬][11]

------------
<p align="center">欢迎关注公众号：</p>
![公众号][1]

<p align="center">感觉内容不错，读后有收获？欢迎小额赞助：</p>
![赞赏][2]

  [0]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCZ6gG5NJjutfc6ZHJLrS03l9SOZbtcUVZpjg7KpA8mLsSEk8FZjlicsluXXorAoDAKFBIQWDBtr0g/0?wx_fmt=jpeg
  [1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAoGfjsaJt2NQ0a9AKmrIRoR9gKlX1I78Z4AoPtjyEPM56slw9gAQBdAHjHckbw4h93FvVVATBuLQ/0?wx_fmt=jpeg
  [2]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD3anvFetwgNHv3X1AiaXIzWPvazEMIEralm9vs42XsVfoniaXRCSkSpNpz9icsIYFgq84Eic2whLdAfg/0?wx_fmt=jpeg
  [3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjdv2WlZrZsd1yFnnopudFWR4BgWJxlMzRA2HeSxZqTfS8a5zJ77wB0MQ/0?wx_fmt=png
  [4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjd7cMmQoRicfRq1nFxGicb4h6ia2wSNU6zDJksFDSn05qU8af0UxnxTWRAg/0?wx_fmt=png
  [5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjdJRkhIrKicG5ga0t921b06tpWdgKlnUR4oNUa4ckuE1M5EGf13vXPbOQ/0?wx_fmt=png
  [6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjdbsB0oFMfribQXLhQVCMadKqQKxNe09oOB1UVQR1icLJ6UnYGwiazORIkA/0?wx_fmt=png
  [7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjdL6l9E8e49VfCnhnGnHkGuY5IIgAhicHHXjj8f2TNe1WJ6aCcYLTHxeA/0?wx_fmt=png
  [8]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjdqJy7xARasK7YLPJeTGCvibjOsWcpr57FyEOCxWLUOV9vkGiaIDhVUkUQ/0?wx_fmt=jpeg
  [9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjddIQuaQFlfMIM9Q2N2AxqxSR6V0uZnPv5pcp3pDbSxYnzY31mziaRNSA/0?wx_fmt=png
  [10]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjdQNdFhLJgoEGTo31AHXl03GkbUfhkXeVxoyziakAE1ficEHkl2hRe2icAQ/0?wx_fmt=jpeg
  [11]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjdEZEEMZiaAsgPsF1pQWAK4XohI5hE4ictz7w609tH3jibCOsYCgsE8icUVg/0?wx_fmt=jpeg
  [12]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eC047Wv5lpgqYIlpqMPhjjdQNdFhLJgoEGTo31AHXl03GkbUfhkXeVxoyziakAE1ficEHkl2hRe2icAQ/0?wx_fmt=jpeg







