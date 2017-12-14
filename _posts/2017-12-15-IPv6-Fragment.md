---
layout: post
title: IPv6数据包的分片和重组
date: 2017-12-15
categories: blog
tags: [通信, IP]
description: IPv6是一个神奇的东西
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

日本工作的拼劲常常倍大家津津乐道，不过实地接触后才感觉拼的无以复加了，这边有一个同事据说一天只吃一顿量大的晚餐，然后据观察每晚最后一封邮件都基本凌晨1点之后发出，然后每天第一封邮件早晨5点多发出，也就是说中间睡眠4个小时不到，而有的时候甚至早晨4点多就有邮件发出。

跟国内同事聊此，大家调侃着应该是做了定时任务吧，还有同事说着应该是北辰一刀流的体系人员，忍术修习到了一定阶段了吧。

还有两个日本同事，一个大概有50多岁的大爷了吧，典型的日本人，每天必须西装，还有一个比较年轻大概30左右的样子，跟他们一起工作的时候每晚都会熬到10点之后才走，后来发现了他们不管有没有工作了都会慢慢的熬到那个时候，所以慢慢的也就是差不多工作都做完了客套一下就撤了，实在是熬不起。

说了些杂七杂八的东西，接下来正题，前几天聊完了IPv4的分片，今天就聊聊IPv6的分片和重组情况。

#### 1、IPv6 Header Format
首先来看一下RFC2460定义了如下的IPv6的Header Format：

<img src="https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDtmPrItAmQ6HUso7zNyRontyDibvmebPia1u5CHq5ZlPdOXWGib3tu6cEaJoAAu64KLTpMTXuicetAibg/0?wx_fmt=png" width="80%" />

1. 由于IPv6头中没有了可变位，所以IPv6的包头长度是固定的为40bytes，
并且移除了IPv4包头中的Internet header length(4)、Identifier(16)、Flags(3)、Framented Offset(13)、Options(Length variable、used for test)、Padding。

2. __Payload Length__为IPv6有效载荷长度，即IPv6包头之后的长度，如果存在扩展包头，此Length中包括扩展包头的长度。

3. __Next Header__中标识了紧接在IPv6首部后下一个首部的类型，目前规划的IPv6 Next Header如下：

<img src="https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBZxn3Xk7ZcEECps6CLmiaibKoqFI0Bo5xgfOe89vJCGzm5A0SbvibMO4l22bbdovjfYOOjj8rib71IFw/0?wx_fmt=png" width="40%" />

&ensp;&ensp;&ensp;&ensp;__Note: __在最新的定义中Type 59——No Next Header不再作为Extension Header出现。

#### 2、IPv6和IPv4分片的差异化

#### 2.1、中间节点的处理方式不同
首先IPv6和IPv4不同的是IPv6只允许在源节点分片和目的节点重组，中间节点路由器只做转发，不再对IPv6数据包重组或再次分片，当收到的分片数据包依然大于PMTU（Path MTU Discovery）的时候，给源端发送ICMPv6的Packet Too Big消息来告知其MTU，消息体如下：

<img src="https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eB8uaIdT7WsXpoNhWyafkT0WvZrDjtzMZbFrpiaKLPE91icluyUDKGVATic66xciceTMjQNWusIexsiaMA/0?wx_fmt=png" width="80%" />


而对于IPv4来说，中间节点路由器可以针对分片消息进行重组和重新分片等操作。

#### 2.2、对于分片包的标示位不同
对比IPv4和IPv6的Header Format可以看出，IPv6中包头中移除了IPv4中Fragment的相关位如Identifier(16)、Flags(3)、Framented Offset(13)。

IPv6中引入了扩展包头（Extension Headers），而Fragment Header作为其中的一个类别表示了IPv6中的分片信息。

#### 2.3、分片包长度计算不同
IPv4中分片之后依然使用IPv4中的Total Length，并且Total Length包含了IPv4的包头长和数据净长度。

IPv6中分片之后依然使用Payload Length这个字段，但是此字段不包括IPv6的包头长，但是却包括扩展包头Fragment Header的长度和数据净长度，下边就来聊聊Fragment Header。

#### 3、Fragment Header的介绍
首先在IPv6的Header中__Next Header__值需要为__Fragment Header for IPv6（44）__。

根据RFC2460中的定义，Fragment Header格式如下：

<img src="https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDtmPrItAmQ6HUso7zNyRonzjzdNvGsdQx7z7ic04LCvOw76rvQmEEtBuwpNY5UiaeT7lufQ6GXQcHA/0?wx_fmt=png" width="80%" />

其中：

1. Next Header：标识源数据包也就是分片之前数据包中可分片部分的数据首部的类型；
2. Reserved：保留字段，传输时初始化为零; 接收时忽略；
3. Fragment Offset：同v4中一样，13bits，表示了此数据包在完整原始数据包中的位置，以偏移量表示此数据跟完整原始数据包中第0位的偏移单元，而一个偏移量以8 octest(64bits)为一个单元。
4. Res：保留位，传输时初始化为零; 接收时忽略；
5. M flag：数据包中显示为More Fragments，同v4中的MF一样0表示是最后一个分片，1表示非最后分片也就是后续依然还有分片数据包。
6. Identification：同一个源数据包的分片标识，当源节点发送一个大于MTU的数据包时，对数据包分成若干分片包，此时需要给各个分片包定义一个标识值，并且此标识值必须不同于近期内同一对源节点和目的节点之间其他的分片包的标识值；如果存在Routing header（路由首部）， 那么目的节点是指最终目的节点。

#### 4、如何分片和重组的呢？
#### 4.1、源节点分片数据包
当源节点决定发送一个数据包，并且大于其设定的MTU时，需要对数据进行分片之后再发送。

此时，源数据包可分为如下两部分：

<img src="https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDwCWAqrQFx3UAqvpZ5sGhX4HYOcw571yoVeLL6u1mZNw3mAHicRyDzaosUhXVVpNatRdNkWmheFUg/0?wx_fmt=png" width="80%" />

1. 不可分片的部分（Unfragmentable Part）包括IPv6的包头以及必须由中间节点路由器处理的扩展包头如：Routing Header或者Hop-by-Hop Options Header。

2. 可分片部分（Fragmentable Part）包括了其他需要最终目的节点处理的扩展包头和上层数据，此部分根据MTU大小切割成若干相同大小的分片数据，且每一个分片数据为8 octets的整数倍，然后剩余小于MTU的数据组成最后一个分片包，此时源数据包分片如下如下：

<img src="https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDwCWAqrQFx3UAqvpZ5sGhXObG94hfnRHPOIHndlqmeAkc7VoNOh3dYQVicZEQ8ljIyvR7Xib6bFxmA/0?wx_fmt=png" width="80%" />

然后源节点开始进行构造各个分片数据包并发送到目的地：

<img src="https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDwCWAqrQFx3UAqvpZ5sGhXShRTnlXW7lRaOO2nojYB53WNz3xjjiaicUfMaeBxzh3sLbfLicJGDgBdA/0?wx_fmt=png" width="50%" />

其中每一个分片数据包由如下部分构成：

1. 源数据包中的不可分片部分，Payload Length为此分片数据包的排除IPv6包头长以外的所有数据长度，Next Header中必然为Fragment Header for IPv6（44）；
2. 扩展包头——Fragment Header——内容：a. Next Header为源数据包中的Next Header；b. Fragment Offset: 第一个分片为0，之后的分片为8的整数倍；c. M flag：最后一个分片为0，其他分片为1；
3. 分片数据。

__Note: __由于中间节点路由器不针对分片数据包重组和再分片，所以源节点的MTU最好定义为所有节点的MTU最小值。

#### 4.2、目的节点重组数据包
当目的节点收到各个分片数据包，通过源和目的地址、Identification、Fragment Offset和M Flag进行连接得到重组数据包：

<img src="https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDwCWAqrQFx3UAqvpZ5sGhXyo1xd41GyFic3JjsxSA4UXmzWYtmIBut8Vzpzkjlx4gQCpeeyCApiatA/0?wx_fmt=png" width="80%" />

重组后的数据包的不可分片部分（Unfragmentable Part）包括如下部分：
```
1. 第一个分片包的IPv6包头：其中需要移除Fragment Header部分
2. 最后一个Next Header为第一个分片的Fragment Header中的Next Header；
3. Payload Length可以按照如下的方式计算：
   PL.orig = PL.first - FL.first - 8 + (8 * FO.last) + FL.last
   其中：
      PL.orig  = 组合包的走长度就是分片之前源数据包的；
      PL.first = 第一个分片包的Payload Length；
      FL.first = 第一个分片包的Fragment Header之后的数据长度；
      FO.last  = 最后一个分片中的Fragment Header内的Fragment Offset；
      FL.last  = 最后一个分片包Fragment Header之后的数据长度。

   然后个人根据观察和总结可以使用如下公式：
   PL.orig = 8 * FO.last+ (PL.last-8)
      PL.last = 最后一个分片包的Payload Length。
```

#### 5、一例解千愁

#### 5.1、IPv6数据包分片
例如有一个定义了IPv6的节点需要发送Payload Length=1764的数据给另一个IPv6的终节点，要经过一个使用默认MTU=1500的路由器：

由于Payload Length加上IPv6 Header Length一共长1804大于MTU 1500，所以当数据包到达路由器时，由于MTU限制和IPv6只有源和目的节点可以分组数据包，因此路由器需要通过ICMPv6且Type为Package Too Big（2）告知源节点，并且告知源节点目前MTU为1500。

源节点收到Packge Too Big的ICMPv6消息后根据MTU开始进行分片：

由于MTU是1500，应该包含发送数据中的IPv6包头，因此可以发送的Payload Length为1460=1500-40，此时1460/8=182.5不是8的整数倍，所以向下四位取1456/8=182是8的整数倍，所以可以发送最大的Payload Length为1456。

这样分片之后的数据包如下：

1. Payload Length为1456；Next Header为Fragment Header for IPv6（44）；Fragment Header中Offset为0，M flag为1。那么第一个分片包发送了源数据包多长的数据呢？在数据净长度为1456-8=1448，这样还剩余316（1764-1448）长度的数据包待发送。
2. 第二个分片包需要发送的数据净长度为316，这样第二个分片包的Payload Length为316+8=324；Next Header依然为为Fragment Header for IPv6（44）；Fragment Header中Offset为181，M flag为0。其中Offset为第一个分片包的数据净长度/8也就是1448/8得到181。

这样，新的两个分片数据包可以通过路由器完美到达目的地。

例子中加深对偏移量的理解：偏移量（Offset）的计算需要数据净长度除以8得到，因此偏移量为原始数据包Payload Length的偏移位数。

#### 5.2、IPv6数据包重组
目的节点收到了如下三个数据包：

1. Payload Length=1392；Next Header=Fragment Header for IPv6（44）；Fragment Header中Offset=0；M Flag=1；Identification=0x20120105；
2. Payload Length=1392；Next Header=Fragment Header for IPv6（44）；Fragment Header中Offset=173；M Flag=1；Identification=0x20120105；
3. Payload Length=228；Next Header=Fragment Header for IPv6（44）；Fragment Header中Offset=346；M Flag=0；Identification=0x20120105；

那么源数据包的Payload Length是多少？从源到目的地中最小MTU是多少呢？

首先，根据三个分片包的Identification可以判断出这三个数据包同属一个源数据包，再由Offset和M Flag可以判断其连续性。

这样，路径中最小的MTU就为第一个分片包的Payload Length加上IPv6 Header Length得到：MTU=1392+40=1432。

那么原始数据包的Payload Length是多少呢？由公式如下公式可以计算：
```
PL.orig = 8 * FO.last+ (PL.last-8)
PL.orig=8*346+(228-8)=2988
```
所以原始数据包的Payload Length为2988，即目的节点重组后得到Payload Length为2988.


以上就是IPv6分片的内容，如有错误欢迎微信公号留言斧正。

*相关链接：*

- *[IPv4数据包的分片和重组](http://minpukang.github.io/blog/2017/12/11/IPv4-Fragment/)*

*参考文档：*

- *[RFC 2460: Internet Protocol, Version 6 (IPv6)](https://tools.ietf.org/html/rfc2460)*
- *[RFC 7045: Transmission and Processing of IPv6 Extension Headers](https://tools.ietf.org/html/rfc7045)*
- *[RFC 1191: Path MTU Discovery](https://tools.ietf.org/html/rfc1191)*
- *[RFC 2463: Internet Control Message Protocol (ICMPv6)](https://tools.ietf.org/html/rfc2463)*

------------
<p align="center">欢迎关注公众号：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAoGfjsaJt2NQ0a9AKmrIRoR9gKlX1I78Z4AoPtjyEPM56slw9gAQBdAHjHckbw4h93FvVVATBuLQ/0?wx_fmt=jpeg" width="40%" />

<p align="center">感觉内容不错，读后有收获？欢迎小额赞助：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAzA577Ce49rCLiby9EtT195GRiaqKCT6QCQ5Weia9OZD72MJz4ABlqAy1gbHepk5hHM464hCiarQRI7w/0?wx_fmt=jpeg" width="30%" />

  [1]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDtmPrItAmQ6HUso7zNyRontyDibvmebPia1u5CHq5ZlPdOXWGib3tu6cEaJoAAu64KLTpMTXuicetAibg/0?wx_fmt=png
  [2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBZxn3Xk7ZcEECps6CLmiaibKoqFI0Bo5xgfOe89vJCGzm5A0SbvibMO4l22bbdovjfYOOjj8rib71IFw/0?wx_fmt=png
  [3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eB8uaIdT7WsXpoNhWyafkT0WvZrDjtzMZbFrpiaKLPE91icluyUDKGVATic66xciceTMjQNWusIexsiaMA/0?wx_fmt=png
  [4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDtmPrItAmQ6HUso7zNyRonzjzdNvGsdQx7z7ic04LCvOw76rvQmEEtBuwpNY5UiaeT7lufQ6GXQcHA/0?wx_fmt=png
  [5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDwCWAqrQFx3UAqvpZ5sGhX4HYOcw571yoVeLL6u1mZNw3mAHicRyDzaosUhXVVpNatRdNkWmheFUg/0?wx_fmt=png
  [6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDwCWAqrQFx3UAqvpZ5sGhXObG94hfnRHPOIHndlqmeAkc7VoNOh3dYQVicZEQ8ljIyvR7Xib6bFxmA/0?wx_fmt=png
  [7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDwCWAqrQFx3UAqvpZ5sGhXShRTnlXW7lRaOO2nojYB53WNz3xjjiaicUfMaeBxzh3sLbfLicJGDgBdA/0?wx_fmt=png
  [8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDwCWAqrQFx3UAqvpZ5sGhXyo1xd41GyFic3JjsxSA4UXmzWYtmIBut8Vzpzkjlx4gQCpeeyCApiatA/0?wx_fmt=png


