---
layout: post
title: IPv4数据包的分片和重组
date: 2017-12-11
categories: blog
tags: [通信, IP]
description: 大不了就切一下
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

最近这两天在解决一个问题的时候遇到了IP分片的问题，之前总是关注信令的东西，数据业务很少研究，所以也就保持在知道个大概的阶段，但是涉及到VoLTE和VoWiFi的SIP消息时，就可能碰到IP分片和重组等相关内容。

#### 1、为什么要分片？
虽然伴随着产业的发展，目前网络传输的带宽已经越来越大、越来越不是瓶颈，但是在最开始的设计中因为带宽限制等，就有了太大的数据包如何传输的问题？

比如，运输一个大衣柜（大的数据包），因为城市道路等限高等等以及最终入户时候的电梯和单元门屋门大小的限制等等，从出场到入户必然是一个大问题，那么怎么办呢？

这个时候厂家就想到了何必直接打成一个整体衣柜呢？按照衣柜的上下左右前后等等部位独立制作（分片）。

这样将分片后的衣柜各个部位可以独力运输，不再受限。

#### 2、分片大小有什么规定？——MTU
大衣柜是有自己的标准的，那么对于IP数据包这样的数据流分片要参照什么标准呢？

首先大衣柜的分片因为运输受限，所以在IP中也是因为运输限制导致了分片，那么这个限制一个是固定的带宽（比如马路宽都是标准的），另外就是IP中存在的一个MTU（maximum transmission unit，比如马路上的限高），当数据包超过MTU的时候就需要分片。

在以太网网络中，默认的MTU是1500。

#### 3、分片有什么标准？
大衣柜的各个部分都是有对应标准，或者认为标号来规定各个分片的连接方式以方便再次重组。

那么IP网络中是如何定义这种连接的呢？

来看一下在RFC791中的IPv4的Internet Header Format：

![IPv4_Header_Format][1]

其中分片相关的有16bits的Identification、3bits的Flags和13bits的Fragment Offset。

Identification：发送端发送的IP数据包标识字段都是一个唯一值，该值在分片时被复制到每个片中。

#### 3.1、16bits的Identification
Identification共16bits，是发送端发送IP数据包的时候唯一值，并且如何数据包被分片后，此字段复制到各个分片中：

```
  Identification:  16 bits

    An identifying value assigned by the sender to aid in assembling the
    fragments of a datagram.
```

#### 3.2、3bits的Flag
Flag一共有三位，其中：

1. 第一个是预留位，因此目前一直保持值为0；
2. 第二位为是否分片，0表示分片，1表示不分片；
3. 第三位表示是否为最后一个分片数据，0表示是最后一个分片，1表示非最后分片也就是后续依然还有分片数据包。

```
  Flags:  3 bits

    Various Control Flags.

      Bit 0: reserved, must be zero
      Bit 1: (DF) 0 = May Fragment,  1 = Don't Fragment.
      Bit 2: (MF) 0 = Last Fragment, 1 = More Fragments.

          0   1   2
        +---+---+---+
        |   | D | M |
        | 0 | F | F |
        +---+---+---+
```

#### 3.3、13bits的Fragment Offset
Flags已经表示来此数据包是否为分片包，那么这个包是第几块儿分片包呢？

因此就有了Fragment Offset的设定，这一个区域表示了此数据包在完整原始数据包中的位置，以偏移量表示此数据跟完整原始数据包中第0位的偏移单元，而一个偏移量以8 octest(64bits)为一个单元。

```
  Fragment Offset:  13 bits

    This field indicates where in the datagram this fragment belongs.

    The fragment offset is measured in units of 8 octets (64 bits).  The
    first fragment has offset zero.
```

#### 4、万变不如一例
#### 4.1、IPv4分片实例
例如一个定义了MTU为1280的设备要转发一个数据包长度(Total Length)为3000且IHL为5的IPv4数据包，由于包长受到了MTU的限制，必然要采取分片的流程。

那么分片结果是怎样的呢？首先净数据包长是多少呢？

首先IP包里的Total Length中包括了数据包头长（IHL=Internet Header Length）和数据包的净长度，因此数据包净长度为Total Length-IHL\*4，因此此数据包的净长度为3000-5\*4=2980.

1、由于Fragment Offset的单位8 octets，所以不是最后一个分片的其他数据包长度必然为8的整数倍，由于MTU为1280，除去IP头的IHL5\*4(字节)=20之后得到1280-20=1260，由于1260/8=157.5不是8的整数倍，所以取1260-4=1256，1256/8=157为8的整数倍，所以第一个Fragment的数据净长度为1256，数据包总长度为1256+20=1276，剩余的数据净长度长2880-1256=1724；

2、由于分片1之后净长度依然大于MTU 1280，所以继续分片，依然可以发送净长度为1256的数据，数据包总长度为1256+20=1276；此时由于已经是第二个分片，那么其Fragment Offset为1256/8=157，剩余的数据净长度为1624-1256=468；

3、经过第二次分片子后，剩余的净长度为468，小于MTU，因此此分片为最后分片，其中总长度为468+20=488，Fragment Offset=1256*2/8。

因此分片数据包如下：

- Fragment 1：IHL=5; Total Length=1276; Flag=001; Fragment Offset=0;
- Fragment 2：IHL=5; Total Length=1276; Flag=001; Fragment Offset=157;
- Fragment 3：IHL=5; Total Length=488; Flag=000; Fragment Offset=314;

#### 4.2、IPv4数据包重组实例
现在收到一个如下的分片数据包：

- Fragment 1：IHL=5; Total Length=508; Flag=001; Fragment Offset=0;
- Fragment 2：IHL=5; Total Length=312; Flag=000; Fragment Offset=61;


1、Fragment 2的Offset为61，61\*8=488，所以第一个Fragment的数据包净长度为488，通过第一个Fragment可以得到Total Length-IHL\*4=508-5\*4=488，所以这两个分片包为相连数据包，且由于Fragment 2的Flag为为000，有Section 3.2可以知道Fragment 2为最后一个分片；

2、那么原始数据包的总长度是多少呢？Fragment 2的净长度为Total Length-IHL\*4=312-5\*4=292，所以数据包的净长度为292+488=780，因此原始数据包总长度为780+IHL(5)*4=800。

更简单的一个算法就是使用__最后一个分片的Offset*8+最后一个分片的长度__即为原始数据包的总长度：61*8+312=488+312=800。

以上就是IPv4的内容。

------------
<p align="center">欢迎关注公众号：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAoGfjsaJt2NQ0a9AKmrIRoR9gKlX1I78Z4AoPtjyEPM56slw9gAQBdAHjHckbw4h93FvVVATBuLQ/0?wx_fmt=jpeg" width="40%" />

<p align="center">感觉内容不错，读后有收获？欢迎小额赞助：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAzA577Ce49rCLiby9EtT195GRiaqKCT6QCQ5Weia9OZD72MJz4ABlqAy1gbHepk5hHM464hCiarQRI7w/0?wx_fmt=jpeg" width="30%" />

  [1]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDtmPrItAmQ6HUso7zNyRonsIGNicMLIlc9GsrMCPxuSjyjd2fFticMDIKRNss2nbEe4WWPF7DDDOIA/0?wx_fmt=png
  [2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDtmPrItAmQ6HUso7zNyRontyDibvmebPia1u5CHq5ZlPdOXWGib3tu6cEaJoAAu64KLTpMTXuicetAibg/0?wx_fmt=png
  [3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDtmPrItAmQ6HUso7zNyRonzjzdNvGsdQx7z7ic04LCvOw76rvQmEEtBuwpNY5UiaeT7lufQ6GXQcHA/0?wx_fmt=png


