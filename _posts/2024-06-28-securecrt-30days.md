---
layout:     post
title:      软件库｜SecureCRT无限续期30天，附CRT激活工具，MAC和Win都有
date:       2024-06-28
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 软件
    - SecureCRT
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

所谓工欲善其事必先利其器，作为ICT的运维工程师，远程操作Linux、路由器、交换机、云、云原生等都是日常任务，所以必然需要一款ssh客户端利器。

SecureCRT是行业中数一数二的ssh软件了，唯一的缺点就是需要氪金。

一直以来都是使用破解版的，因为某些原因不被允许了，所以就尝试使用其他的替换比如xshell、MobaXterm等。

但是因为诸多功能需求，比如多级端口转发，随时记录log、打时间戳等等，Xshell、Moba等都不如SecureCRT好用（据说MTPutty作为套壳putty还不错），而SecureCRT的30天只能用一次。

怎么办呢？一个方法就是旧版本的30天试用完，可以安装一个新版本继续试用，但是他们的发布速度赶不上过期速度。

后来在多方位立体搜索后，有朋友找到了一个可以无限续期30天的方案。

要怎么做呢？

首先可以从官网下载并安装最新版本的SecureCRT 9.5.2，使用起来后就开始倒计时了：

![@七禾页话][2]

第一次开始使用SecureCRT之后，会在注册表中“HKEY_CURRENT_USER\Software\VanDyke\SecureCRT\”生成一个Evaluation License的注册信息：

![@七禾页话][3]

那么如何无限期续30天呢？就是要删除注册表里Securecrt的“Evaluation License”。

两种方式，一个是通过GUI删除：
1. 打开注册表。
2. 找到“HKEY_CURRENT_USER\Software\VanDyke\SecureCRT\Evaluation License”右键删除即可。

![@七禾页话][4]

![@七禾页话][5]

更快捷的方式是通过命令行直接删除，打开CMD之后输入 **REG DELETE "HKEY_CURRENT_USER\Software\VanDyke\SecureCRT\Evaluation License"** ，然后 **yes** 确认即可：

![@七禾页话][6]

当然也可以通过 **/f** 强制删除：

![@七禾页话][7]

需要注意的是，如果SecureCRT安装后没被打开，就不会在注册表里生成“Evaluation License”，那么在CMD里删除的时候会报错"**ERROR: The system was unable to find the specified registry key or value.**"，可以忽略。

​既然可以命令行删除，那么就可以实现自动化做成开机自启动，这样就可以自动无限续期30天了。

首先是要写一个批处理bat脚本，比如命名为 **sr_trial_30.bat** ：批处理的主体就是删除注册表里的信息，然后有两秒的展示后自动关闭​。

```shell
## 左右滑动（不要带这句）

@echo off
REG DELETE "HKEY_CURRENT_USER\Software\VanDyke\SecureCRT\Evaluation License" /f

echo.
echo "The "ERROR: The system was unable to find the specified registry key or value." due to secureCRT not opened before or not agree the trial license, ignore it"
echo.
timeout /t 2 /nobreak >nul 

```
然后需要将 **sr_trial_30.bat** 这个批处理文件放到自启动的文件夹 **%AppData%\Microsoft\Windows\Start Menu\Programs\Startup** 里，查看自启动中出现批处理的任务后每次开机就会自动运行删除SecureCRT的注册信息了：

![@七禾页话][8]

有一个问题就是因为使用的是试用模式，所以每次打开都会有倒计时提醒，并且需要点击 **I Agress** 之后才能打开，目前没有找到办法来规避这个问题，如果有朋友知道，欢迎留言或者私信。

![@七禾页话][9]

最后就是如果不想使用试用版，而且也没有公司规定需要使用正版要求，可以公众号后台回复 **crt** 获取安装包和激活工具。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBNw0a53H5nzgplRLmI38ZVBH9BpzwaLEIZUyDdwVZ5ZicfbNia8qakZ5909AXHxTS7zzHduXNAm8tg/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBNw0a53H5nzgplRLmI38ZVQKiafyCWxFydicRcWCuEkq7Gsib3cXJeic4kdTNymdibEXctwyuBHO2e6XQ/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBNw0a53H5nzgplRLmI38ZV87P2ibPEys7QN2797ZGteh7jW30pJP4ALHB1IU28MCbP1Imn9Y6qicWA/640?wx_fmt=png&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBNw0a53H5nzgplRLmI38ZVeYEtATg4TOLRVWM9FRVreICM145naMBvcvexB9Yz6CQaP8lA5oRHYw/640?wx_fmt=png&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBNw0a53H5nzgplRLmI38ZVrnUVWW6DPbz7VxtAXRggBApqXq0Nic7XsDibUBPEUxRWFeicZe89nxyLQ/640?wx_fmt=png&amp;from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBNw0a53H5nzgplRLmI38ZVTujqSuUmWaMp7rlKe0eEC0Pj9Yae74fDibYibn460lOUwicLQ3LtxL9pw/640?wx_fmt=png&amp;from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBNw0a53H5nzgplRLmI38ZVVfMb5HC5OenAUZYKnw912iatOZJ5NqiaBSlLcwvjyseQT2gguCn9Pw3Q/640?wx_fmt=png&amp;from=appmsg


[8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBNw0a53H5nzgplRLmI38ZVGt2a8hHGsO8WCiaIe1XK2sb8icjI3laBicibP43XumDxvqJcP4hqDmSQyg/640?wx_fmt=png&amp;from=appmsg


[9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBNw0a53H5nzgplRLmI38ZVRicJVQI7RVosxTMOah2yiayyGWsf8fZjPGibVWXCcIqYs9RRSPvI89ibcQ/640?wx_fmt=png&amp;from=appmsg

