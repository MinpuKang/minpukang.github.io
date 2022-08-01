---
layout:     post
title:      应用｜IP子网计算器和一揽子附加项
date:       2022-08-01
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 
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

![后山的夏天，总是有落霞和天线同窗][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

直入正题，今天分享一个自己写的IP子网计算器。

为什么要写这个软件？

作为通信工程师，IP划分是一个不可躲避的任务，划分IP的时候经常会遇到两个问题，一个是客户直接给你一个很大的地址段，需要按需分配进行子网切割；另一个就是在一个IP段中需要知道哪些是可以用的主机地址，方便进行网络地址分配到具体的业务。

基于以上两个问题，对于我这种非计算机型的人脑就需要借助一些工具辅助计算。

用了几个离线的或者在线的IP计算系统后，遇到了两个问题，一个是IPv4需要一个手动填写每一个点分十进制后，在选择对应的掩码长度，最后点一个确认按钮后得出结果；另一个就是没有找到一个IPv4和IPv6同时支持的工具。

![][10]

遇到这样的问题大概率是我没有进行深度的网络搜索，信息时代，爆炸的是信息，但是有用的就不出现在你身边。

所谓自己动手丰衣足食，车到山前不一定有路，但是走的人多了就自然成了路，所以就决定私人订制一个自己的IP子网计算器，正好在Github里发现了一个叫做ipnetwork的Repository，基于C#写的一个IP计算的类，看了下完全是自己想要的，这个Repository完全可以命令行使用，如果有兴趣的朋友可以直接使用，仓库链接是：https://github.com/lduchosal/ipnetwork

C#是我在众多编程语言中相对熟悉的了，果断套壳封装一个图形化界面。

最终主要应用，也就是IP子网计算如下，直接在IP地址栏输入IPv4或者IPv6即可，在输入的过程中就完成了IP格式的检测，一旦检测到一个合规的IP地址后，根据IPv4或者IPv6直接输出相应的结果，简单直接粗暴。

![][2]

上图演示的是版本1，今天更新到了版本1.1，针对IPv4的结果添加了反掩码的输出，想来对ACL的书写会有帮助。

![][11]

输出框是一个RichTextBox，可以直接全选复制出来。

另外如果IP地址中没有掩码，默认使用其类掩码，也就是A类是8，B类是16，C类24，IPv6默认使用64作为默认前缀长度。

除了主要功能之外，IPCalculate也封装了其他的功能，可以通过主程序的“More”按钮开启。

![][3]

第一个子功能就是IP子网划分，就是将一个大段的网络，根据输入的新的掩码或者前缀长度，进行全新的划分，为了保证快速的输入，如果划分后的子网数量超过10个，目前只输出前十个：

![][4]

![][5]

第二个子功能是判断一个IP或者IP段是否包含在其他的地址段中，也是IPv4和IPv6都支持。

![][6]

第三个子功能是子网合并。

![][7]

目前此功能要求是相邻的两个子网合并，而且合并后只能包含这两个才可以完成合并，举个例子：

比如：
- 想合并2.2.0.0/29和2.2.0.8/29，是可以成功的，因为合并后的2.2.0.0/28不再包含任何其他的/29的子网。
- 想合并2.2.0.8/29和2.2.0.16/29，就是失败的，因为合并后是一个/27的掩码，但是2.2.0.0/27中不仅包含了2.2.0.8/29和2.2.0.16/29，还包含2.2.0.0/29和2.2.0.24/29，所以目前这个在此软件汇中是不能合并的。

![子功能的自我验证][12]

最后，当然少不了关于和帮助，直接在主功能中点击“About”进入。

![][8]

About里包含了软件的名字版本和版权，以及各个版本的迭代，当然也少不了一波宣传，就是扫码关注七禾页话(qiheyehk)公众号，可以反馈bug或者功能需求。

![][9]

当然也可以通过“阅读原文”直接在这个项目的Github仓库提交Issue或者新功能需求。

那么如何获取此软件呢？因为主体也是使用开源库，所以这个软件完全免费使用，可以七禾页话(qiheyehk)公众号后台回复“IPC”获取下载链接（不区分大小写），也可以直接“阅读原文”在下载仓库/bin/Debug/IPCalculate.exe，也欢迎直接fork此仓库。

最后的最后，希望这个小工具能发光发热！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAUwojmjZ4ngQiaYbYWGiasBITDT3ND9AKndARzkav7bQyp9LMLnEib0phsjpcOakxwA3mdwNF42ibk9A/0?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCYVwibzfT6iaHFN3uphDTFL51McuuMU1wz9dXUZcyjYEzFBKn2AqYchjdbWC1wwva3WVcttpQzd1Yw/0?wx_fmt=gif


[3]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCYVwibzfT6iaHFN3uphDTFL5ACgnOLYnpl40YFWDcphaSicvlxrqDV2nibsiboyIBMtLkONxHqWzuDg0Q/0?wx_fmt=gif


[4]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCYVwibzfT6iaHFN3uphDTFL5BZ9ViaCCauKfLPvIRlCnibjicQygueS1nhrsCjNNlVjrLdbhlHFJ3l7fw/0?wx_fmt=gif


[5]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCYVwibzfT6iaHFN3uphDTFL5UERY44XA72Qf9LibVp1E6pZUuudbKyTH5G9VOFQo3lgia6OPnLxicaltw/0?wx_fmt=gif


[6]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCYVwibzfT6iaHFN3uphDTFL5WCKWeyLRCJVGKCycFsnM4hFQiaYxVN8v2qdEnBoaibUia6ial6Q11dFDRQ/0?wx_fmt=gif


[7]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCYVwibzfT6iaHFN3uphDTFL5yK20gEibI0IiaHib2lDO5wEpjfibFWMQRT01Sm2SE2nALSwCtjdT0U0ibiaQ/0?wx_fmt=gif


[8]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCYVwibzfT6iaHFN3uphDTFL5BBy7HbcxpfLTetYLUiakJhpbLtU3b8ibGI0UXz7yhJpCGnmibgicQBIeJg/0?wx_fmt=gif


[9]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCYVwibzfT6iaHFN3uphDTFL5RwgN2bR9muwk1sJCEMmnJIU48nS2ia7HzF5icUNYiayfmgte27icgswKzg/0?wx_fmt=gif


[10]:  https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAUwojmjZ4ngQiaYbYWGiasBICBz9xQ1GIeg3Bx76gicXen92LsJvnVqfMIbIn3eReNcINJDaMxZ0BPg/0?wx_fmt=png


[11]:  https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAUwojmjZ4ngQiaYbYWGiasBIwdpV4a5TcK9xqdw08cxM255z9pg1dtLM0HhraXAYlKpSNqC3TkTKKw/0?wx_fmt=png


[12]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAUwojmjZ4ngQiaYbYWGiasBIWx8InaLKKiaymRVlKrj951MgGlMzrTr72r0u4k1o3p9OoFVJyiaPhqJg/0?wx_fmt=png

