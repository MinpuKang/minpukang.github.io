---
layout:     post
title:      应用｜创建SNMPEngineID的小工具
date:       2023-02-25
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 应用
    - SNMP
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

![2023年2月25日日出][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

为什么要写这个工具？

是因为在做K8s(Kubernetes)项目的时候遇到了告警监控的问题。

背景就是如果一个业务是走K8s Primary network，会发生一次SNAT，将发出去的包的原地址SNAT成对应worker上的接口IP。

然而一般来说snmp client需要定义的是业务地址，而不是某一个worker的地址，如果snmp client只通过IP层的原地址来判定这个数据的来源，就会出现混淆，导致snmp的数据解析有问题。

那么一个很好的解决方案就是在snmp server上定义一个叫做snmpEngineID的参数作为其唯一标识，然后snmp client如果支持这个engineID的解析就可以唯一区分不同的snmp server信息，做正确的数据解析了。

那么engineID要怎么定义呢？如果是一个随机的ID是否可以被snmp client识别呢？答案是不确定，要看snmp client具体实现，是可以直接定义一个engineID还是从一个有规律的engineID中自动检测。

![][2]

如果是从一个有规律的engineID中自动检测，就需要有一定的行业规范。RFC3411给出了很明确的定义，截取了最高bit位为1的Item 3规则如下:
```
左右滑动
1) The very first bit is used to indicate how the rest of the data is composed.
    0 - as defined by enterprise using former methods that existed before SNMPv3. See item 2 below.
    1 - as defined by this architecture, see item 3 below.
3) The length of the octet string varies.
    The first four octets are set to the binary equivalent of the agent's SNMP management private enterprise number as assigned by the Internet Assigned Numbers Authority (IANA).
    For example, if Acme Networks has been assigned{ enterprises 696 }, the first four octets wouldbe assigned '000002b8'H.
    The very first bit is set to 1. For example, the above value for Acme Networks now changes to be '800002b8'H.
    The fifth octet indicates how the rest (6th and following octets) are formatted. The values for the fifth octet are:
        0     - reserved, unused.
        1     - IPv4 address (4 octets) lowest non-special IP address
        2     - IPv6 address (16 octets) lowest non-special IP address
        3     - MAC address (6 octets)lowest IEEE MAC address, canonical order
        4     - Text, administratively assigned Maximum remaining length 27
        5     - Octets, administratively assigned Maximum remaining length 27
        6-127 - reserved, unused
        128-255 - as defined by the enterprise Maximum remaining length 27
    SYNTAX       OCTET STRING (SIZE(5..32))
```
规则说的挺清楚，分三块儿：
- IANA分配的Privite Enterprise Number转换成占有前四个八位字节的字符串后最高位再置为1。
- 第五个八位字节定义了第六位和之后的数据格式
- 从第六位的八位字节到最后一个就是具体的engine ID Data，根据第五位会有不同的定义，因此长度可变，最终导致一个engine ID的八位字节长度从5到32。

规则清楚算起来还是要颇费周章的，所以就应运而生了这个小软件，取名为SeiFor：S——SNMP， E——Engine，I——ID，For——Format。

![][3]

SeiFor将三大块儿规则做了格式化自动计算，然后在任何一个输入窗口输入的时候就自动检测三个条件，一旦满足就在Result里生成对应的engineID：

![][4]

如果有任何错误或者不匹配的输入，都会在result里提示错误信息，如：
- PEN不为空且必须为整数，目前没有规定必须是IANA分配的区间，具体值可以通过超链接打开IANA网站查询。。
![][5]
- 三个必须参数不能为空。
![][6]
- Engine ID Data是根据Format来判断是否正确，比如Format选择了IPv4，Data里的每一行都必须是一个IPv4，否则对应行就会有错误提示。
![][7]

然后从主窗口可以打开About窗口，有简单的软件介绍，同时可以通过关注公众号(qiheyehk)来反馈任何的问题。

![][8]

最后是一个snmp的测试结果，可以看到wireshark抓包正确解析engine ID：

![][9]

如何获取这个应用？
- 可以clone这个项目的repository： [https://github.com/MinpuKang/SNMPv3EngineId-Format](https://github.com/MinpuKang/SNMPv3EngineId-Format)

   其中运行文件在 [https://github.com/MinpuKang/SNMPv3EngineId-Format/tree/main/SeiFor/bin/Debug/net6.0-windows](https://github.com/MinpuKang/SNMPv3EngineId-Format/tree/main/SeiFor/bin/Debug/net6.0-windows)
   
   注意：​需要下载​net6.0-windows整个文件夹之后exe才可以运行。

- 另外也可以在公众号后台回复**snmp**获取下载链接，下载即用。

以上！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBnCvVlhHWbzGs3fSyib1licuahbjImUDvlKq2v2qtubBVsFIgq99Tvy1qO6EpMePDlk12UtIiaTY2rQ/0?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eA6EWAZxwatygjwC4xo6jWhZXKajU8G4MjszYPALgZHfMibtCp0bnVqIjSjJlQNhxP8ThicTLOQzTdg/0?wx_fmt=png


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eA6EWAZxwatygjwC4xo6jWhqrY2SynCJ1H0SeuvNN8dGJiaekUogTyiaYF12KRfUA12NOL6mic8Dngvg/0?wx_fmt=png


[4]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eA6EWAZxwatygjwC4xo6jWhDNSeR1TBpcia2S0vpABEnG4zBGUrWYsyJ9vicSicvfqiaXTUPVKBvPqrYQ/0?wx_fmt=gif


[5]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eA6EWAZxwatygjwC4xo6jWhYZqCrNsRzFhtKCOJ9buVibhvX6GiaaJ28ic1WUT7MfJGHicUCZHNmHaxWg/0?wx_fmt=gif


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eA6EWAZxwatygjwC4xo6jWhiasaxN7UvpeabEyEgsiarw9E5owakovbTEl2GAeg3gPrnYK9u8qmRn6A/0?wx_fmt=png


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eA6EWAZxwatygjwC4xo6jWhxUnORTBSbRRA9ic6TOiaV2aVCAIyBmslBg1sTg8W7R5iajEialE1xWI2Sw/0?wx_fmt=png


[8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eA6EWAZxwatygjwC4xo6jWhpqYDTbv7bEJ8EBtCZvTTBatBca973TeGN7uRopicy58ZxENEbCgoianQ/0?wx_fmt=png


[9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eA6EWAZxwatygjwC4xo6jWhIxIEBsvs8ibRcP2gkVIy4qnRaN2b3lovBZtiaGqh0UncbEJ6uOveFOwA/0?wx_fmt=png


