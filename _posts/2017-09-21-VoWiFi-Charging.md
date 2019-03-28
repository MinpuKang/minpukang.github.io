---
layout: post
title: Wi-Fi Calling到底漫没漫游？
date: 2017-09-21
categories: blog
tags: [通信, WiFi]
description: 漫游无国界！
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

这是Wi-Fi Calling的第二篇，先不赘述流程，来聊聊漫游资费的问题。

其实计费没有什么过多可以说的，国内参考VoLTE就可以了，毕竟只是接入方式不一样，没有什么本质区别，都是为了解决语音业务，就如同VoLTE的目的跟使用2/3G打电话的目的一样，为了交汇彼此的意见，沟通彼此的感情，从而达到心灵相通、情意相投，促进社会主义的和谐进程。

然而目前不管是商用的还是测试的Wi-Fi Calling，都面临一个漫游收费的问题，当然国内漫游已经在总理的大力促进之下，全面取消了，终于在计费这个问题上可以万里横行一刀切了。

## 1、Wi-Fi Calling的好处
聊资费之前先来看看Wi-Fi Calling都有什么好处呢？

- 1、解决无线覆盖死角问题：
    + 无线覆盖的问题尤其是国外，大面积“无人区”，所以不像国内到处建站，因此Wi-Fi的存在完美弥补了覆盖的空白。
    + 而在国内虽然这两年大范围铺设基站，无线覆盖已经越来越不是一个问题，但是在居民区、偏远地区等等仍然严重。
    + 所以Wi-Fi的使用不管是在国外还是国内，都可以完美弥补当前网络的覆盖死角问题。
- 2、降低资费：
    + 在国内的时候，不管是针对Wi-Fi接入进行独立套餐，还是跟VoLTE共享一类套餐等等，都是可以的。
    + 重点是国际漫游的时候，资费是可以降低的，接下来细说。
- 3、可以跟目前的4G网络完美切换，这个也是目前OTT业务无法做到的。

接下来说说国际漫游资费。

## 2、4G的国际漫游
对于国际漫游，不管是出国的，还是来中国的，从网络来说毕竟都用着别人的设备，以4G举例：

![4G漫游拓扑图][3]

比如出国旅游，从无线到核心网的MME、SGW等设备都是要租用漫游地的设备，各个运营商不可能也不会被允许到别国建立网络设备，尤其是中美这种大国，通信好歹也算是一个国家的核心产业。

所以怎么办呢？那就是各个国家的运营商之间签署协议，比如中国的用户到美国，那么可以暂时使用美国运营商的设备，美国的用户来到中国，也可以使用中国运营商的设备。

那么既然有使用到别人的设备，必然是要交钱的。这样就产生了国际漫游，由于不同国家政策、消费等等，各个地方的漫游费自然不等，比如移动就有一元区、两元区、三元区等等，也就是一分钟几块钱，对于爱煲个电话粥的情侣，那分分钟百八十的就没了，这也算是爱的代价吧。

所以呢，大家在迈出大陆的时候虽然有开了国际漫游，但是使用率就下来了，没办法，谁让他贵呢，不过还好现在有很多的APP拯救了我们。

## 3、Wi-Fi漫游了么？
再来看看我们的Wi-Fi Calling，复习一下Wi-Fi calling的拓扑：

![Wi-Fi calling拓扑图][4]

上一篇，我们说在Wi-Fi里手机找ePDG是根据手机配置的如下FQDN通过DNS查询得到的，甚至可以直接将ePDG的IP写入到手机：

    epdg.epc.mnc<MNC>.mcc<MCC>.pub.3gppnetwork.org

我们先来看看这个域名，最主要的就是这个mnc+mcc，它表示了这个用户的开户运营商，目前来说开户运营商固定了，比如使用的是中国移动，那么这个FQDN也就基本固定了。

那么如果一个FQDN固定了，不管你在什么地方——大江南北长城内外还是国内国外，解析的地址就基本固定了，那么最终所使用的ePDG设备也就固定了。

我们来看看目前如果Wi-Fi的国际漫游的设备是怎么使用的呢？

![Wi-Fi Calling漫游拓扑图][5]

从图中可以看到，Wi-Fi中仅有无线的AP和其DHCP server，另外一部分Public DNS使用了漫游地的设备，其他的核心设备依然都在归属地，也就是开户地。

那么在这样的一个拓扑使用情况下，出国使用Wi-Fi Calling到底算不算漫游了呢？也就是到底要不要额外收费呢？

就目前方案来说，个人观点这算哪门子漫游？要多收哪门子费？

那么问题来了，某某声音：“这没有国际漫游了呀，我怎么多收钱？不行，坚决不上”，这也是之前给某一个运营商做演示的时候运营商的一个老大当场就跳了起来嚷嚷，可见虽然我等小民基本不用国际漫游，但是国际漫游依然是一个重要的收费关口呀。

之前就此问题跟同事聊了很久，为什么我坚持认为目前方案不算漫游呢？

首先Wi-Fi Calling是被称为Untrusted Wi-Fi，也就是说接入测不管是什么，运营商自己铺设的Wi-Fi比如CMCC无线网、还是星巴克/万达等网络、亦或者就是家里办的宽带，都可以作为接入点，甚至你有钱拿个手机做无线热点共享都可以，而且无线接入不受运营商限制，你是移动的号，家里宽带是电信的，没问题，只要移动开了Wi-Fi Calling的功能，依然可以以电信宽带作为接入点。

那么这样说来我在国内和出国用Wi-Fi Calling没差呀！

难道国外的Wi-Fi就不是正经的Wi-Fi了么？

## 4、为了生存
但是运营商是绝对不会轻易放弃国际漫游的，而且也不会轻易降价的，这里面的诸多利益不是我等可以想象的；即使到了随处可见Wi-Fi，大量部署Wi-Fi calling的时候，国际漫游依然要存在的，全球大同的社会道路还要很远很远很远。

那么现在对于国际漫游有什么可行的方案呢？
- a、一种比较极端的方案，直接拒绝Wi-Fi Calling的国际漫游，这种方案就比较直接了，既然不是漫游，索性到其他地区就别用了。
- b、一种就是比较大方的方案，Wi-Fi Calling包，包月使用，不管你到哪里，这个也诠释了Wi-Fi Calling无漫游。
- c、第三种就比较合理，从技术上讲还是能区分出来你到底是在国内还是在国外的，一种是可以借助EPC网络的位置信息来判定，另一种是Wi-Fi中最终是要使用公网IP的，那么各个地区的公网IP范围固定，那么就可以根据手机的IP来判定其是在国内还是国外，从而对其进行不同的收费。

不管怎样，以上的收费统统都是要进入到开户运营商的腰包的，必然要受到广大人民群众的唾弃。

那么合理的方案是什么呢？3GPP里其实也给了一种解释，就是在FQDN上做手脚，在3GPP 23.003里讲ePDG的FQDN提到了如下的Visited Country FQDN：

![ePDG Roaming][6]

也就是用户在游走在各个地区的时候生成对应地区的FQDN从而在设备上使用对应地区的ePDG，也就是达成了如下漫游拓扑的使用条件：

![ePDG Roaming拓扑][7]

这就如同2/3G的漫游一样，ePDG跟SGSN作为漫游地的设备被使用，从而漫游地和开户地继续共同收费。

以上！

------------
<p align="center">欢迎关注公众号，摄影，旅行，瞎聊，等等等：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD1g2UjYu4VfCGHmbhgVqOAnNnJQfN7ZhRVUCopYOsfpPtIEB95VNEqu8trAxJXzGDg01ka6z6wzQ/0?wx_fmt=jpeg" width="30%" />

  [0]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCZ6gG5NJjutfc6ZHJLrS03l9SOZbtcUVZpjg7KpA8mLsSEk8FZjlicsluXXorAoDAKFBIQWDBtr0g/0?wx_fmt=jpeg
  [1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAoGfjsaJt2NQ0a9AKmrIRoR9gKlX1I78Z4AoPtjyEPM56slw9gAQBdAHjHckbw4h93FvVVATBuLQ/0?wx_fmt=jpeg
  [2]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD3anvFetwgNHv3X1AiaXIzWPvazEMIEralm9vs42XsVfoniaXRCSkSpNpz9icsIYFgq84Eic2whLdAfg/0?wx_fmt=jpeg
  [3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eA67Zsv6yZm2ARRmWIE1jDP8DiasjibyBtHg8dNqauOLjjOFj8pMBdXF9MkztVpNISZvTJA9Xs1bjfA/0?wx_fmt=png
  [4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCb5jqPzRL7OicNOVfFibtqz12YFMWiarQb4hAdIibaePpGbZcLRz2MgbJsjtBQ9P7SZQyJ5hezaa2YMA/0?wx_fmt=png
  [5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBichZfIjUfibdtiav4bicWOqYrichzDMORDqjY6ax876byEtIMDtG7H4U3NObOH7NMuKXlCrpcOjIpGsQ/0?wx_fmt=png
  [6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBichZfIjUfibdtiav4bicWOqYrfBtrckztVaaUkyo9eQq3B7SKGgYh84yXQOlW8mlbze6Qv5FVy1VGWw/0?wx_fmt=png
  [7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBichZfIjUfibdtiav4bicWOqYr3bpayiaZiats2jmRDRDT9vn4nnVDXwONS49umCvsJOqONTG6AZicsDeIA/0?wx_fmt=png








