---
layout:     post
title:      应用｜创建SNMPEngineID版本2
date:       2023-03-05
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

![大连傅家庄海滨浴场][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

时隔十天，版本2发布。

这个版本主要是丰富了主要功能，增加了针对ipv6、mac地址和任意的字符串的支持，也就是如下的四类全方位支持了，至于其他的format比如5 - Octets，没有想到使用场景，暂时就不考虑了：
```
#左右滑动
    1     - IPv4 address (4 octets) lowest non-special IP address
    2     - IPv6 address (16 octets) lowest non-special IP address
    3     - MAC address (6 octets)lowest IEEE MAC address, canonical order
    4     - Text, administratively assigned Maximum remaining length 27
```
主功能针对Private Enterprise Number做了限制，允许0-99999的输入，目前IANA里分配到了60147，等啥时候越界再更新这个地方了。

![][2]

本版本增加了另外一个功能界面就是Decode，针对一个engineid直接做解析，从而可以快速知道PEN、engine id data的forma和engineID data的具体内容，engineID data的解析也只针对IPv4、IPv6、MAC地址和Text四类：

![][3]

最后就是一些小地方的优化，一类是针对.NET6的一些性能建议的优化，比如字符串切割等，对于使用来说是隐藏式的；另外一类就是使用中的所有的URL都做了超链接，主页面和Decode中的PEN超链接会根据是否有数值做到是否带搜索打开。

![][4]

![][5]


关于页面中的介绍原来使用TextBox，这次改成了RichTextBox并使用LinkClicked事件，可以针对文本中的url做成超链接。

![][6]

以上就是版本二的新增和优化。

如何获取这个应用？
- 可以clone这个项目的repository： [https://github.com/MinpuKang/SNMPv3EngineId-Format](https://github.com/MinpuKang/SNMPv3EngineId-Format)

   其中运行文件在 [https://github.com/MinpuKang/SNMPv3EngineId-Format/tree/main/SeiFor/bin/Debug/net6.0-windows](https://github.com/MinpuKang/SNMPv3EngineId-Format/tree/main/SeiFor/bin/Debug/net6.0-windows)
   
   注意：​需要下载​net6.0-windows整个文件夹之后exe才可以运行。

- 另外也可以在公众号后台回复**snmp**获取下载链接，下载即用。

以上，如果有任何使用中的问题和想法欢迎随时反馈。

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCweicKrVsSpf08U6z8K2czEX4aZggyCMXY0oObQ9KusIZckhVPWdF5kqMxL0sbHC0npYeb0B7LQLA/0?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCweicKrVsSpf08U6z8K2czEtR84icMSIwZVC6Cqh81QoXArAmxZ459EEvaMe2Af6BOahApGMQ8MblQ/0?wx_fmt=gif


[3]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCweicKrVsSpf08U6z8K2czEsfbDtX4b9CMLVGf8dFI8UibZAmkyP84guD6tEib1icDznPorp0YaHUe4A/0?wx_fmt=gif


[4]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCweicKrVsSpf08U6z8K2czEjap2ickpEgxHHtqzN7pASP63swSpWgAsYtbDnqCRcrOMrrZpHLX564Q/0?wx_fmt=gif


[5]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCweicKrVsSpf08U6z8K2czEU3oySJgy5sTFxlCLr3XUsdv3P9nfrkzxTPuujPb9MJwuMTja2WKtSg/0?wx_fmt=gif


[6]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCweicKrVsSpf08U6z8K2czEvmic25nUxGxPoQa9p4XyAwkg4tQS9PAVJypFlvMKeeML4OWd6NtQYicA/0?wx_fmt=gif


