---
layout:     post
title:      Linux｜WSL打造Windows下更顺畅的双系统之终篇
date:       2024-06-29
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

![@七禾页话][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

前几天有朋友问我找一下WSL的文章，说公众号里的东西太多没翻到，所以我就定义了一个关键字 **wsl** 的自动回复，不过还是将二更的文章发给了朋友，又说了下设置开机自启动在一更的文章里。

自己说完之后都感觉很麻烦，所以就想着把两篇合称一篇做一个最终章。

WSL的全称是Windows Subsystem for Linux，是微软拥抱开源的里程碑式功能，从Windows10开始支持，从而使得在Windows系统的基础上可以运行类原生Linux系统，为什么想到了类原生这个概念呢？因为Linux的某些功能确实在WSL上无法实现，比如docker在WSL1上是不可用的。

虽然WSL有一些限制，但是对于日常学习、了解和使用还是非常方便的，那么如何在Windows10以及高版本使用WSL呢？大概分三步：

1. 开启wsl功能；
2. 安装一个发行版本；
3. 初始化：初始化系统、安装其他的服务、开机自启动等。

## 1、开启WSL功能

默认情况下Windows没有开启WSL的功能，所以第一步就是要在控制面板——程序——启用或关闭Windows功能里开启WSL，开启后需要重启电脑才能生效：

![@七禾页话][2]

![@七禾页话][4]

![@七禾页话][5]

## 2、安装一个Linux发行版本

开启了WSL功能后，需要安装一个Linux系统，可以通过两种方式安装：

1. 直接通过微软商店的GUI安装；
2. 通过wsl命令安装。

#### 2.1、通过微软商店GUI安装

打开微软商店，搜索Linux后就会看到诸多WSL下的Linux发行版本，比如安全类的Kali，日常用的Debian、Ubuntu、Suse等等。

![@七禾页话][6]

![@七禾页话][7]

![@七禾页话][8]

选择一个想使用的版本安装即可，比如我选择的是Ubuntu：

![@七禾页话][9]

#### 2.2、通过wsl命令行安装

如果微软商店被限制下载非授权的软件或者其他原因等无法使用GUI安装，或者想做自动化开发，可以尝试使用 **wsl** 命令来安装管理Linux系统。

首先可以通过 **wls --list --online** 查看可以安装的Linux版本，这里可能会遇到“无法解析服务器名词或地址”的错误：
```
左右滑动
PS C:\> wsl --list --online
无法解析服务器的名称或地址
PS C:\>
PS C:\>
```

解决办法是规避魔法的DNS查询，直接将URL的解析放到**C:\Windows\System32\drivers\etc\hosts**里，这里推荐使用notepad--软件进行更新（懂的都懂！！！），更新内容如下：
```
##左右滑动
185.199.108.133   raw.githubusercontent.com
```

然后就可以通过 **wsl --list --online**查看可以安装的版本了：
```
左右滑动
PS C:\> wsl --list --online
以下是可安装的有效分发的列表。
请使用“wsl --install -d <分发>”安装。

NAME                   FRIENDLY NAME
Ubuntu                 Ubuntu
Debian                 Debian GNU/Linux
kali-linux             Kali Linux Rolling
Ubuntu-18.04           Ubuntu 18.04 LTS
Ubuntu-20.04           Ubuntu 20.04 LTS
Ubuntu-22.04           Ubuntu 22.04 LTS
Ubuntu-24.04           Ubuntu 24.04 LTS
openSUSE-Tumbleweed    openSUSE Tumbleweed
PS C:\>
```

之后通过 **wsl --install -d <Linux发行版NAME>** 安装，比如选择Ubuntu（安装的是最新的Ubuntu版本）：
```
左右滑动
C:\Users\username>wsl --install -d ubuntu 
Downloading: Ubuntu 
Installing: Ubuntu 
Ubuntu has been installed. 
Launching Ubuntu... 
```

## 3、初始化Linux系统

不管是通过微软商店GUI后需要手打还是 **wsl** 命令安装都需要初始化Linux系统。

如果是通过微软商店GUI安装的，需要手动在开始菜单栏打开运行Linux系统（当然也可以直接搜索Linux的名字打开）：

![@七禾页话][10]

打开后等一段时间初始化，就会提示添加一个新用户，此用户会默认带sudo功能（什么是sudo？不知道的可以留言），然后设置用户名密码即可，如果是通过 **wsl** 命令安装的，安装完成有可能会自动加载(Launch)，跳出下图初始化界面了，如果没有，按照上述介绍手动加载(Launch)：

![@七禾页话][11]

## 4、运行Linux子系统

有三种方法运行Linux系统。

一是搜索bash打开：

![@七禾页话][12]

二是通过Win下的命令行模式：打开cmd之后运行bash命令，就可以进入Linux系统。

![@七禾页话][13]

三是通过开始菜单栏打开安装的Linux系统或搜索Linux系统名，比如我使用的Ubuntu：

![@七禾页话][14]

## 5、开启远程登录

不管是通过cmd运行bash，还是开始菜单栏运行Linux系统，在使用中不是很方便，比如复制粘贴、记录log等等，都无法很舒服的实现。

在简单的摸索后，终于实现了使用putty、secureCRT、xshell等软件的远程登录。

第一步是要先检查子系统中是否开始了sshd服务，如果没有开启，使用相应命令开启。

![@七禾页话][15]

然后配置一下sshd服务（vi /etc/ssh/sshd_config），比如修改一下port等等，比较重要的是要检查是否允许使用密码登录，修改完成需要重启ssh服务：

![@七禾页话][16]

Note：对于图片中could not load host key的错误，可以通过执行“dpkg-reconfigure openssh-server”来解决：

![@七禾页话][17]

接下来就可以通过远程登录Linux子系统了（IP：127.0.0.1，Port：sshd_config中设置）。

![@七禾页话][18]

注意：最新的Ubuntu比如24.04已经默认安装了ssh，但是尝试启动ssh服务的时候可能会遇到 **no hostkeys available**的问题：

```
左右滑动
[root@wsl:~]# service ssh status 
 * sshd is not running 
[root@wsl:~]# service ssh start 
 * Starting OpenBSD Secure Shell server sshd 
 sshd: no hostkeys available -- exiting. 
[fail] 
```

解决方案是通过ssh-keygen -A创建密钥即可：
```
左右滑动
[root@wsl:~]# ssh-keygen -A 
ssh-keygen: generating new host keys: RSA DSA ECDSA ED25519 
[root@wsl:~]# 
[root@wsl:~]# service ssh start 
 * Starting OpenBSD Secure Shell server sshd         [ OK ] 
[root@wsl:~]# 
```
- -A 对于不存在主机密钥的每种密钥类型（rsa、dsa、ecdsa 和 ed25519），生成具有默认密钥文件路径、空密码、密钥类型的默认位和默认注释的主机密钥。

## 6、开机自启动安装的Linux中的ssh服务

在使用的过程中发现一个问题，就是每次Windows系统重启后，都需要打开Linux系统，然后开启ssh服务后才能再次远程登录，完全没有了什么便利性。

多次尝试后找到了其解决办法，就是通过Windows开机自启动功能调用一个子系统里的脚本来实现，具体设置如下。

首先，要在Linux系统里写一个脚本，并通过chmod命令给脚本添加运行权限，脚本内容如下：

```
​##左右滑动
echo "<在第三章初始化中设置的密码>" | sudo -S /usr/sbin/service ssh start
```

![@七禾页话][19]

然后在Windows下写一个vbs程序（如wsl.vbs），并放到开机自启动的路径下，vbs内容如下：
```
set ws=wscript.createobject("wscript.shell")
ws.run "C:\Windows\System32\bash.exe",0
ws.run "C:\Windows\System32\bash -c '/usr/sbin/ssh_start'",0
```
自启动路径：**%AppData%\Microsoft\Windows\Start Menu\Programs\Startup**

![@七禾页话][20]

接下来就可以开机后直接打开secureCRT等远程登录软件顺畅使用Linux子系统了。

Note：朋友说Mobaxterm直接集成了wsl，所以如果使用Mobaxterm不需要vbs自启动可直接登录wsl。

## 7、后记和体验

自从电脑使用Win10系统后，Cygwin再也没有安装过，VMWare安装也是为了做K8s等实验，至于其他的日常Linux工作，基本都在使用wsl，wsl的Linux里可以使用几乎所有的Linux服务，比如验证自己写的shell脚本、Python脚本等，而且可以在Windows主系统里写代码，然后直接在wsl中验证，这是因为Linux子系统可以直接读取主系统的文件，主系统的各个分区也是自动挂载的：

![@七禾页话][21]

最后，虽然wsl还有一些局限，比如没有Linux图形界面（但是Linux使用，尤其是运维，还是得靠CLI），再就是不能使用docker等容器技术（wsl2已经可以了），所以想要做更复杂的实验比如K8s等还是要使用BVirtualBox或者VMWare等虚拟机方案，或者自建服务器。

最最后，​公众号后台回复 **wsl** 可以直接获取相关文章！

话说这好像不会是终章，因为wsl2还没有玩儿o(╯□╰)o

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAZjJEQ6sQSXx9xhqH1ib778WTNZrInAuG4YEcntOPiaavYBuxAmQKeEZW8eZaHvHdcKMD1Q3VShrgw/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCCWZiaVnLjZwtBHv8RTF4RADQQ6AjfcDUQGD4Q18V9BxWCWpib0FZcWtw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCNnyauNq5YPYqeHW6KMR0WKvVCY2DvL8bjLSEQQXicyXBArSspdcBrUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCRpU9ZGV3iaedTdEu1l15s84fl2Yr9cgXCRH0wJwMKZSuGMw8y9BrXRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCzTMF8yQqkTs6lPEwnfnSzUcGxJHLQSj4ibQPQB9PYKhHAobtEo3EtDg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCa1suuibrDiaVMWfpELSlWRHk7UXSPELYopEfx4PVPlxHiaH68QGaN9k1Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCp5MrRq2edRCmgZsl00j0SE2LWMWGVW4zJTvv9cusib5VeBktVxkowPg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCic0nzibPwgaGBBkib5AtHb1sDgauQWRtpPsv0CFK08VmI4DRJqkVs8elw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[10]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCiaabjJI55QIfYZnJiaPymCpAjYUCZufYkejrTgyziasLKDicHicwTleXIsA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[11]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCpKNxJeUiapHwINPAtqsgicBoUL43QZ9TvztV4IGuv00ZNehB5GGhmCZA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[12]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCHMKwQaM91XtBLWpcHQZgWA4aF2CMRcc3XBsv9gF57449hQg20UHg6A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1

[13]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuC5icI50kUQJo1iaCg9IRTof5gdVdUVKZc8OtibxaX7qWrG3tntw76DHTAw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1

[14]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCxePGXGiabsMvzuoMPEtw9dgKWztFYOwumfqPyjYibIO4OV5nemEmxyjQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[15]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCuOrmA4hXqpsjhCQov7RuYcpgQT9EJGaq46SxjmkWsSHQ6msgr8n3ibg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[16]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCnibNuTibPlLlhQTiaT1JTiaJP0B37RrjyCX3uzoralsLjzrux4XvHAMgxg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[17]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCCIwpBKrHfFLh0gOI6tjII0WznQnxgVyPYOIaoictREyBYTuHlgqYjUw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[18]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCo0fbUPalibF2IRzvyIJqE7x8xfdQEMib964TlfgIJqxAXEGqgaf56YzQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[19]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCzBSleghgGxiaAjuL5EQ9LK7WLf7qWPwrc1Xhb4MLXoich8nRc7LfXkbw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[20]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuCAgDqWmyf81E0KC9WVxIxLVs0upaMwyf4tKexjXhL39vdJZwrUvJ21A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1


[21]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eD8dpl0zXTQcTYnbfsChnuC0UdwrbtYs1PX83uAGHeGoWDZQHCbH5yfNzmVUV0ia1MRWBBs8wGesAw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1