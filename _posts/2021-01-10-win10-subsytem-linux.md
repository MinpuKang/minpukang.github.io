---
layout:     post
title:      Linux｜WSL打造Windows下更顺畅的双系统
date:       2021-01-10
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

这篇文章已经在list待了太久（好像有两年了吧），作为一名通信工程师，Linux是必不可少的操作环境，所以在公司升级到Win10系统后果断就放弃了Cygwin和VMWare虚拟机，通过WSL建立了一个微软原生支持的Linux操作环境。

WSL是Windows Subsystem for Linux的缩写，是微软和Canonical公司合作开发的一个兼容层，从而在Win10上运行原生Linux成为现实。

![1][1]

在之前不管是使用Cygwin还是VMWare虚拟机，都会额外的占用很大的CPU、内存、硬盘等资源，对于日常需要快速使用，甚至直接引用Windows下的文件等等，都需要额外的操作来完成。

所以借助Win10的WSL功能，可以打造一个轻快便捷的Linux环境，对于日常工作带来了更多的方便。

那么如何使用呢？

#### 1. 开启子系统功能

默认情况下对于子系统的支持是关闭的，所以首要的一步就是要开启此功能，开启后需要重启电脑。

![2][2]

![3][3]

![4][4]


#### 2. 安装一个Linux系统

接下来就是要安装一个Linux系统，打开微软商店，然后搜索Linux就会看到诸多Linux版本，比如注重安全的Kali，日常熟悉的Debian、Ubuntu、Suse等等。

![5][5]

![6][6]

![7][7]

选择自己想使用的系统安装即可，比如我选择了Ubuntu。

![8][8]

#### 3. 初始化Linux配置

安装完成后，通过开始菜单栏打开运行Linux系统（当然也可以直接搜索Linux的名字打开）。

![9][9]

打开后会通过一段时间初始化，然后就会提示添加一个新用户，此用户会默认带sudo功能（什么是sudo？不知道的可以留言），然后设置用户名密码。

![10][10]

#### 4. 运行Linux系统

有三种方法运行Linux系统。

一是搜索bash打开：

![11][11]

二是通过Win10的命令行模式，打开cmd之后运行bash命令，就可以进入Linux系统：

![12][12]

三是通过开始菜单栏打开安装的Linux系统或搜索Linux系统名，比如我使用的Ubuntu：

![13][13]

#### 5. 开启远程接入

不管是通过cmd运行bash，还是开始菜单栏运行Linux系统，在使用中不是很方便，比如复制粘贴、记录log等等，都无法很舒爽的实现。

在简单的摸索后，终于实现了使用putty、secureCRT、xshell等软件的远程登录。

第一步是要先检查子系统中是否开始了sshd服务，如果没有开启，使用相应命令开启。

![14][14]

然后配置一下sshd服务（vi /etc/ssh/sshd_config），比如修改一下port等等，比较重要的是要检查是否允许使用密码登录，修改完成需要重启ssh服务：

![15][15]

Note：对于图片中could not load host key的错误，可以通过执行“dpkg-reconfigure openssh-server”来解决：

![16][16]

接下来就可以通过远程登录Linux子系统了（IP： 127.0.0.1. Port：​sshd_config中设置）。

![17][17]

#### 6. 开机自启动WSL的ssh服务

在使用的过程中发现一个问题，就是每次Windows系统重启后，都需要打开Linux系统，然后开启ssh服务后才能再次远程登录，完全没有了什么便利性。

多次尝试后找到了其解决办法，就是通过Windows开机自启动功能调用一个子系统里的脚本来实现，具体设置如下。

首先，要在Linux系统里写一个脚本，并通过chmod命令给脚本添加运行权限，脚本内容如下：
```
echo "<在第三章中设置的密码>" | sudo -S /usr/sbin/service ssh start
```

![18][18]

然后在Windows下写一个vbs程序（如wsl.vbs），并放到开机自启动的路径下。​
vbs内容如下：
```
set ws=wscript.createobject("wscript.shell")
ws.run "C:\Windows\System32\bash.exe",0
ws.run "C:\Windows\System32\bash -c '/usr/sbin/ssh_start'",0
```

自启动路径：%AppData%\Microsoft\Windows\Start Menu\Programs\Startup

![19][19]

接下来就可以开机后直接打开secureCRT等远程登录软件顺畅使用Linux子系统了。

#### 7. 体验如何？

目前已经使用两年多，整体体验舒爽，已经作为日常工作不可或缺的Linux环境。

子系统直接调用Windows主机资源，默认联网，这样就可以直接安装自己需要的Linux软件，比如没有python，安装之。

![20][20]

另外子系统也可以直接读取主系统文件，主系统的各个分区自动挂载到子系统里。

![21][21]

但是，子系统也有一些限制，比如不能直接使用Linux图形界面(CLI不香吗？)、使用docker也有问题等等，所以如果想做更多的实验还是最好使用VirtualBox或者VMware的虚拟机方案。

以上！

希望可以让日常的工作环境更加舒畅。

------------
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)，旅行、摄影。。。偶尔分享技术周边</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCe28qFL5JZ4maVw0HXPiadSXoKHeqh7oTtibbGksVVIOyR9U8Ab3y5WOw/0?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCCWZiaVnLjZwtBHv8RTF4RADQQ6AjfcDUQGD4Q18V9BxWCWpib0FZcWtw/0?wx_fmt=png


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCNnyauNq5YPYqeHW6KMR0WKvVCY2DvL8bjLSEQQXicyXBArSspdcBrUA/0?wx_fmt=png


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCRpU9ZGV3iaedTdEu1l15s84fl2Yr9cgXCRH0wJwMKZSuGMw8y9BrXRw/0?wx_fmt=png


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCzTMF8yQqkTs6lPEwnfnSzUcGxJHLQSj4ibQPQB9PYKhHAobtEo3EtDg/0?wx_fmt=png


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCa1suuibrDiaVMWfpELSlWRHk7UXSPELYopEfx4PVPlxHiaH68QGaN9k1Q/0?wx_fmt=png


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCp5MrRq2edRCmgZsl00j0SE2LWMWGVW4zJTvv9cusib5VeBktVxkowPg/0?wx_fmt=png


[8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCic0nzibPwgaGBBkib5AtHb1sDgauQWRtpPsv0CFK08VmI4DRJqkVs8elw/0?wx_fmt=png


[9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCiaabjJI55QIfYZnJiaPymCpAjYUCZufYkejrTgyziasLKDicHicwTleXIsA/0?wx_fmt=png


[10]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCpKNxJeUiapHwINPAtqsgicBoUL43QZ9TvztV4IGuv00ZNehB5GGhmCZA/0?wx_fmt=png


[11]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCHMKwQaM91XtBLWpcHQZgWA4aF2CMRcc3XBsv9gF57449hQg20UHg6A/0?wx_fmt=png


[12]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuC5icI50kUQJo1iaCg9IRTof5gdVdUVKZc8OtibxaX7qWrG3tntw76DHTAw/0?wx_fmt=png


[13]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCxePGXGiabsMvzuoMPEtw9dgKWztFYOwumfqPyjYibIO4OV5nemEmxyjQ/0?wx_fmt=png


[14]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCuOrmA4hXqpsjhCQov7RuYcpgQT9EJGaq46SxjmkWsSHQ6msgr8n3ibg/0?wx_fmt=png


[15]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCnibNuTibPlLlhQTiaT1JTiaJP0B37RrjyCX3uzoralsLjzrux4XvHAMgxg/0?wx_fmt=png


[16]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCCIwpBKrHfFLh0gOI6tjII0WznQnxgVyPYOIaoictREyBYTuHlgqYjUw/0?wx_fmt=png


[17]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCo0fbUPalibF2IRzvyIJqE7x8xfdQEMib964TlfgIJqxAXEGqgaf56YzQ/0?wx_fmt=png


[18]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCzBSleghgGxiaAjuL5EQ9LK7WLf7qWPwrc1Xhb4MLXoich8nRc7LfXkbw/0?wx_fmt=png


[19]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCAgDqWmyf81E0KC9WVxIxLVs0upaMwyf4tKexjXhL39vdJZwrUvJ21A/0?wx_fmt=png


[20]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuC46gQdkB3VmyYhBYUvuWLj0WPYsYDhaMLYVRsVzvQrw9fhFiaGCicTwsw/0?wx_fmt=png


[21]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuC0UdwrbtYs1PX83uAGHeGoWDZQHCbH5yfNzmVUV0ia1MRWBBs8wGesAw/0?wx_fmt=png

