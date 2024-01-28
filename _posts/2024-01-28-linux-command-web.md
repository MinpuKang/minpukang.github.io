---
layout:     post
title:      Linux｜27K+Start的Linux命令行宝藏级网站
date:       2024-01-28
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - Linux
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

![@七禾页话，那是夏季双彩虹的幸运][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

这篇绝对是一个宝藏干货贴，文末有一键获取的指令！

Linux学习和使用离不开命令行。

为什么要使用命令行？因为很多服务器为了节省资源，是不安装桌面服务的，而且也没有必要，因为谁没事儿天天打开Linux的桌面去看东西，一旦服务器上线后，能不碰就不要碰了，安装桌面服务就是浪费资源。

所以ICT行业熟练使用Linux命令是基本要求，最起码很多常用的命令要熟能生巧。

但是Linux里命令成百上千，还在不断迭代增加，看帮助的时候一堆英文也是稀里糊涂，有的时候想做点儿什么的时候，也不知道有没有对应的命令。

然后就在想，如果有一个命令黄页就好了，打开黄页可以看到每个命令的解释、用法和用例，简直不要太幸福。

然后它就来了，命令黄页向我们走来了。

Github上发现了一个有27K+Start的一个项目，叫做`linux-command`，功能如其名，就是用来检索`Linux`命令的。

![@七禾页话][2]

目前收录了从常用的`ls`到不知道的`znew`，从文件传输、备份压缩到磁盘管理、系统设置等等，一共有600多个命令了。

![@七禾页话][3]

大部分命令都有详细的参数介绍和用例，介绍也被翻译成中文了，所以对于命令的使用来说会非常有帮助。

这个项目有在线使用的网站[https://wangchujiang.com/linux-command/](https://wangchujiang.com/linux-command/)。

除了网页版，项目还有微信小程序，有手机Android版，电脑版（Mac/Win/Linux ），以及Docker版本等等，可以在各种系统上使用，各个版本是不同的大神开发的，所以命令数量不是很统一，而且有些版本是有广告的比如小程序：

![@七禾页话][4]

如果由于网速或者一些莫名问题，在线网站不好用，我们可以使用docker安装一个本地版本使用。

我在VMWare虚机里启动了一个docker版本，如果有朋友在Windows下使用`wsl2`，理论上可以直接在`wsl2`所加载的linux子系统中使用docker，我的电脑不支持`wsl2`，所以没有尝试，docker启动的命令非常简单：
```bash
docker run --name linux-command --rm -d -p 9665:3000 wcjiang/linux-command:latest
```
运行后就会从docker hub上拉取镜像启动网站，启动后可以通过`http://<vm ip>:9665`访问：

![@七禾页话][5]

使用VMWare虚拟机的一个劣势就是每次关机后，虚拟机就关机了，docker的运行就停止了，所以每次都要开机后再运行`docker run`来启动，因此为了方便，写一个小的批处理脚本，调用`vmrun`命令启动虚拟机后再通过`runProgramInGuest`调用`docker run`直接启动`linux-command`即可，前提需要一个免密运行sudo权限的用户，然后将批处理脚本做成开机自启动，批处理脚本我写了一个，基本符合要求了，免密运行sudo的VM可以使用之前`k3s`例子中的template即可，需要再安装一下docker，这个模版里运行docker下拉镜像的时候发现了一个小问题，算作一个bug吧，跟DNS相关的，可以自行排错或留言交流。

脚本拿到后根据注释修改后放到`%AppData%\Microsoft\Windows\Start Menu\Programs\Startup\`下即可完成开机自启动配置：

![@七禾页话][6]

当然也可以部署在自己的服务器上，通过服务器地址或者域名解析访问都可以，也会更加方便。

如果命令没有在网站收录，我们可以使用markdown提交新命令的详细解释，做一个contributor：

![@七禾页话][7]

再说一下网站的使用方法，搜索栏里直接输入命令或者中文即可，会有联想提示，进入详细界面后，命令后面还有一个复制按钮，点一下就能直接复制使用，很多功能是非常人性化的：

![@七禾页话][8]

这个命令黄页对于ICT的从业者来说真的是一大神器，对于ICT相关专业比如网络工程、通信工程、计算机、软件等在校生的学习也会非常有帮助的，对于检索和熟悉命令非常实用，强烈推荐。

命令千千万，汇总万万千。

最后，就是一键指令：网站链接、VM模版、自启动的批处理脚本以及小程序版本的二维码都可以通过公众号后台回复`cli`直接获取，一起Enjoy吧！

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBfrGdUicaiatA4IVLhmZ1EnHkHjkK4phhvkoht8ibrB0n6kfgricJaBw2FwaWmcMfcW17G7dzIiac4v8g/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eBfrGdUicaiatA4IVLhmZ1EnH9Q3s1NnOPzYMjxKvwpvoibF5tVcdkGESgAwr9ReSxibJXMeG4TU5NRUQ/640?wx_fmt=gif&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eBfrGdUicaiatA4IVLhmZ1EnH7e7FoT7HSGDUGjkianIbp8OJHppCkSmzlibqfiajPTZLibotqu0wyswgLA/640?wx_fmt=gif&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBfrGdUicaiatA4IVLhmZ1EnHTsg0NhYKyEOKA14FHEmIViaVbhZnra3AVicPo9YHSo45tfqkicKeuJc0g/640?wx_fmt=png&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBfrGdUicaiatA4IVLhmZ1EnHYxLMibxvicdP9INAiatn13m06GRJIQILwgrwG3zc2KxQuZXmZ0zBBiaVqA/640?wx_fmt=png&amp;from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBfrGdUicaiatA4IVLhmZ1EnHPJIAXNackUvekIFDQneQm2ADLmpRvdrSqPokCO8hPmc4y6aD3CdxEQ/640?wx_fmt=png&amp;from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eBfrGdUicaiatA4IVLhmZ1EnHhaEy9ibV8kQcddNbP2OH4ajc3TVMwJYeeTZ9Y6HlHictp2vtibKZaVjQg/640?wx_fmt=gif&amp;from=appmsg


[8]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eBfrGdUicaiatA4IVLhmZ1EnHGKmic8j7Tu4o44GwibC4U11NaQBsAl2smZxQicaLARg7FbeialAhj8sd8A/640?wx_fmt=gif&amp;from=appmsg