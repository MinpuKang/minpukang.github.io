---
layout:     post
title:      通信｜DNS配置生成工具大进化！
date:       2023-11-26
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

直奔主题吧！

通信发展到5G之后，DNS其实已经弱化了不少，不是为了兼容4G，估计5G里就没有DNS什么事儿了，毕竟SBI全部依赖于NRF-disc来发现了。

但是既然4G依然会有很长的一段存活时间，2/3G都还没完全退网，那么DNS在通信核心网里还是有用武之地的，正好最近又在整理DNS的配置了，就有了一些想法，在项目忙碌的空隙中又优化了8年前开始开发的一个小工具：DNS Config Generator。

首先是界面和颜色的更新，同时适配了Windows10的高清显示：
![@七禾页话][2]

然后是更新了logo，虽然不擅长做设计，色彩搭配也没有什么研究，但是感觉新logo比原来的要好看不少，用了一个全球搜索的图标，然后直入正题就是DNS相关的工具，P代表这是一个完全基于3GPP的Public发行版：
![@七禾页话][3]

最最重要的当然是增加了什么新功能？

最基本的功能依然是填写好所需要的内容，然后Submit生成配置，但是在Submit的时候，不仅仅会生成配置文件，也会按照一定格式保存到Excel中，可以作为一份DNS的Design使用，在保存的时候也会把所有填写的内容保存到一个叫做“DNS Input”的sheet中；然后点击Save或者在DNS Config的框中使用Ctrl+S都可以将配置保存成一个txt文件，文件名字会跟Excel的名字使用相同的时间戳，从而保证design和配置可以匹配。
![@七禾页话][4]

Excel中的内容除了有DNS Input，还有根据输入的ID和IP对应关系所生成的Sheet，包括了SGW、PGW、SGSN、MME和AMF：
![@七禾页话][5]

保存的配置文件可以直接放到比如Bind9这样的DNS应用里使用：
![@七禾页话][6]

然后还有个新功能就是可以加载特定格式的excel作为Input，一键读取，不需要一个一个手动填写了，这个特定的格式就是通过Submit生成的Excel里的DNS Input表格，需要注意worksheet的名字必须是“DNS Input”：
![@七禾页话][7]

这个版本还优化了配置输出的内容，比如合并了一个FQDN的命名，同时加上了5G相关的DNS查询。

如何获取？

- 项目的Repository是[DNS-Config-Generator](https://github.com/MinpuKang/DNS-Config-Generator).

- Windows下的可执行文件在Repository中的[DNS Config Public/bin/Release
/](https://github.com/MinpuKang/DNS-Config-Generator/blob/main/DNS%20Config%20Public/bin/Release/DNS%20Config%20Public.exe)文件夹下。

这个工具不仅仅是一个生产力工具，同时也对通信核心网中对于DNS需求有学习辅助的功效，欢迎下载使用，有任何的问题和想法都可以留言或者通过Repository提交Issue。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eDb4SceqnTqGGtoxhWup4ztPe2iapZeYMGVlibLaEQdUjiacEhb5icddQ9vS92KAR7ibNdRVwWhpcDYVdw/640?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD16pibxiaZp02JUd7bCzzWJAzrqC43yP5nR5GSySHxwCZVvuhtSExgcHOxhRrrLibovZsAGtJdiamwXA/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD16pibxiaZp02JUd7bCzzWJAeCvfetcA8mgHEbUTrYOpMX5x2ChJ1rIYtEvPEmbU5xVIpElcfXqhkQ/640?wx_fmt=png&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eD16pibxiaZp02JUd7bCzzWJAMeIsU9Ch9lvgTJDJnvg9zIHv0g6rSTv62ynIx3Rt0lb1cbh2ch1JVA/640?wx_fmt=gif&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eD16pibxiaZp02JUd7bCzzWJAXKobwuApxibCgyicjXrvcEicPRmYlaCr9IibcumFicBMzicKvr0CRnhVQK5w/640?wx_fmt=gif&amp;from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eD16pibxiaZp02JUd7bCzzWJA148nJ6P2M31ebB1ey0XujxnTs8Rc3rnr1buKXtNZKFGahJ1rCqUTQg/640?wx_fmt=gif&amp;from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eD16pibxiaZp02JUd7bCzzWJACMcKaJwKzIKz69UvNhOMB1icNo7S3ic4aRsZU7JI07nm9cib9AIrhbP6g/640?wx_fmt=gif&amp;from=appmsg



