---
layout:     post
title:      通信｜DNS配置生成工具再更新，什么是递归和迭代查询？
date:       2024-01-14
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 通信
    - DNS
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

![@七禾页话][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

其实元旦的时候就更新完了。

上一个版本急急忙忙写的，没有做太多的验证，后来使用中发现写入excel会有bug，同时在bind9里做配置的时候，感觉之前产生的结果将不同zone的查询都杂糅到了一起，虽然有注释简单讲解了不同查询的应用场景，但是不方便在bind9这种DNS里直接配置，所以在2023年末的时候就优化了，发布了7.1这个小版本。

这个版本更新如下：

```bash
###左右滑动
Version 7.1(2023-12-31)
- Re-design the config result based on gprs/eps/5gs record
- Fix bug to save excel
- Add input as appendix in configu result
- Update FQDN generating for DNS query
```

第一个就是将所有的result输出重新定义，按照GRPS、EPC和5GC三个zone输出，同时每个zone里FQDN和A/AAAA Record也分开显示，配置更加简洁，各个call flow需要的DNS解析在FQDN解析前有解释：

![@七禾页话][2]

第二个修复了存储到excel的bug，当时在加入这个功能的时候，对于SGW和PGW，按照IP数量的多少计算应该从第几行的单元格记录不同的record，结果导致如果SGW和PGW只输入一个IP的时候，excel存储就会丢失TAC或者APN的NAPTR解析：

![@七禾页话][3]

第三个将input也同步存储到result文件里，虽然已经存储到excel里，但是这次也将input存储到了配置结果中，方便随时查看dns配置和对应的input：

![@七禾页话][4]

​还有一个重要的没有在about里写，就是加入了NRI的解析，虽然2G在有些国家已经退网，3G也慢慢要面临退网，但是依然有地区在用，而作为2、3G中重要的功能SGSN Pool也是不可或缺的，相应在2/3G和4G的IRAT中NRI、MMECode等之间的转换都有可能涉及到DNS解析：

![@七禾页话][5]

最后优化了DNS的FQDN，比如之前MME的FQDN在TAU和Handover时候最终会有两个不同的FQDN如`oldMME`和`targetMME`，这次全部都优化成了一个`s10-mme01`：

![@七禾页话][6]

最后，还直接加入了`Setup Bind9 as DNS Server`的Github链接，从而可以方便在自己的ubuntu上安装bind9，用此工具生成DNS配置后，就可以使用`dig`命令做实验了：

![@七禾页话][7]

以上就是7.1版本的内容更新。

而[通信｜DNS配置生成工具大进化！](https://mp.weixin.qq.com/s/USgpqysaXcUReoU9SPVy9Q)文章基于的版本7作为重要的发型版本介绍了此工具的详细功能。

如何获取？

- 项目的Repository是[DNS-Config-Generator](https://github.com/MinpuKang/DNS-Config-Generator).

- Windows下的可执行文件在Repository中的[DNS Config Public/bin/Release
/](https://github.com/MinpuKang/DNS-Config-Generator/blob/main/DNS%20Config%20Public/bin/Release/DNS%20Config%20Public.exe)文件夹下。

关注公众号七禾页话(qiheyehk)回复`dns`可以获取直接使用的exe下载链接。

-----------------

DNS查询如果在本地DNS没有结果的时候有两种查询方案：递归DNS查询和迭代DNS查询。

我们已经聊过，DNS按照[Trailing Dot](https://mp.weixin.qq.com/s/lZwzo_nA-7QyCa-Q0gXMGQ)划分为不同的域名级别，不同的DNS里可以解析到的域名因此而不同，那么各个运营商或者互联网公司只能维护自己DNS里自己的域名配置，并将其连接到互联网中，最终形成DNS的网状结构：

![@七禾页话][8]

那么我们的电脑连接的DNS或者各个运营商的DNS不可能定义全球所有域名的解析，因此DNS收到一个域名解析请求后就会根据配置方案来向上级DNS请求，方案两种：一个叫做递归，一个叫做迭代。

#### 迭代DNS查询

迭代查询可以简单地理解为我可以不知道，但是我可以告诉你谁知道。如下图所示：

![@七禾页话][9]

各个步骤如下：
1. 电脑发起DNS请求解析www.hk314.top到本地DNS；
2. 本地DNS说我没有，你去根DNS看看；
3. 电脑继续向根DNS询问，根DNS说我也不知道，你去zone top.的DNS看看；
5. 电脑又到zone top.的DNS询问；
6. zone top.的DNS说我也不知道，你去zone hk314.top.的DNS问问；
7. 电脑又到zone hk314.top.的DNS询问；
8. zone hk314.top.的DNS说我这里有，www.hk314.top的IP地址是1.1.1.1，最终直接将信息告诉给电脑。

可以看到整个的过程都是电脑自己一次又一次向各个DNS询问，各个DNS就是我不知道但是我告诉你谁可能知道，这就是迭代，宗旨就是你得自己去跑业务。

#### 递归DNS查询
递归DNS查询就是不用你亲自跑腿儿了，我们是联合部门，内部查好告诉你最终结果即可。如下图所示：

![@七禾页话][10]

各个步骤如下：
1. 电脑发起DNS请求解析www.hk314.top到本地DNS；
2. 本地DNS说我没有，我去根DNS看看吧；
3. 根DNS收到请求一看我也不知道，我去zone top.的DNS问问吧；
4. zone top.的DNS收到请求一看自己也不知道，那我去zone hk314.top.的DNS问问吧；
5. zone hk314.top.的DNS说我这里有，www.hk314.top的IP地址是1.1.1.1，将信息告诉给zone top.的DNS；
6. zone top.的DNS收到信息后转给根DNS；
7. 根DNS收到消息后转给本地DNS；
8. 本地DNS最终将收到的域名和IP的信息转给电脑。

整个过程就是一个DNS内部运作，对外不公开的节奏，但是最终会告诉你答案。

递归查询对于最终用户来说更为简单，但可能给单个DNS服务器带来更多的压力；而迭代查询则分散了查询压力，但需要客户端具备更复杂的逻辑来处理多步查询过程。

上述介绍只是简单的聊聊两种查询的理论过程，在实际应用中需要两种的结合使用，总之是一个相对复杂的过程。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBBOONVxYfjFYIhwjC4Pz4DKcIW5icibaeAwWHGAPiaa7ApdLh8g8fbCNB0dyoPZqiaYJH3eaF4fA3eyQ/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCcAXibdXarMLolBUeiaLgoRIbjyZIDc6y5s6TP1My9RMFpAKmSjxS0xd0hO0WQRRvp02OuvfaAM53A/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCcAXibdXarMLolBUeiaLgoRIwFgVO7al3W2JYC5Hdlzaj6Kot6giaJ2owg5icCciaIRZMYyx3nticpNWjQ/640?wx_fmt=png&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCcAXibdXarMLolBUeiaLgoRIRWH3YXVichao8Yiaicv3d5cvVy5XHIybU0FiaEicFsaw1gOMm8dmHgjKwGQ/640?wx_fmt=png&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCcAXibdXarMLolBUeiaLgoRIlB9txBIXgw5bsicicLRRqNADuLiaaypz4cQqXBt7wVYSfMwslOkCNhQUQ/640?wx_fmt=png&amp;from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCcAXibdXarMLolBUeiaLgoRIyeDN68CglQTZib1PRxDEa6evBwqSechXbRBJOsNqKtXnQWAqTwpln1Q/640?wx_fmt=png&amp;from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCcAXibdXarMLolBUeiaLgoRIu3o1zKCeqXHnYaA1bUUrUuNKjpIlbpGKH6vfmgzDE2uOUafZAH9icdw/640?wx_fmt=png&amp;from=appmsg


[8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBx5A8pLy6Cdb7IpiadOYogSG8V7VkNZ3z1AJIS3deoxwl3aUTFzZenoic3ia8aYdwlTR9vPJqYveicag/640?wx_fmt=png&from=appmsg


[9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCcAXibdXarMLolBUeiaLgoRID1tVhf4sOTv6YcklqDrrICbia1rbF5qdhBr4S5uW7JcQicZfwFX9CsCA/640?wx_fmt=png&amp;from=appmsg


[10]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCcAXibdXarMLolBUeiaLgoRIgEZCLLEZxtUyFSthlWBBlgIXI0XWn9pe0zU6JkcwibE1vGbF4GZshGw/640?wx_fmt=png&amp;from=appmsg
