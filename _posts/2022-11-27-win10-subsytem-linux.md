---
layout:     post
title:      Linux｜二更WSL打造Windows下更顺畅的双系统
date:       2022-11-27
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

二更此文，是因为公司换电脑后出于安全角度，将微软商店做了很多限制，导致无法通过微软商店下载WSL的软件，所以针对这个部分做了调整。

#### 1. 开启子系统功能

第一步开启WSL的功能没有任何变化。

默认情况下对于子系统的支持是关闭的，所以首要的一步就是要开启此功能，开启后需要重启电脑。

![2][2]

![3][3]

![4][4]


#### 2. 安装一个Linux系统

接下来就是要安装一个Linux系统，可以通过微软商店安装，也可以通过**wsl**命令直接安装。

###### 2.1 通过软软商店安装

打开微软商店，然后搜索Linux就会看到诸多Linux版本，比如注重安全的Kali，日常熟悉的Debian、Ubuntu、Suse等等。

![5][5]

![6][6]

![7][7]

选择自己想使用的系统安装即可，比如我选择了Ubuntu。

![8][8]

###### 2.2 通过wsl命令直接安装

如果微软商店被限制下载非授权的软件，在第一步开启子系统功能并重启电脑后，就可以在命令行模式下直接使用**wsl**命令来安装软件:

```
左右滑动
C:\Users\username>wsl -l 
Windows Subsystem for Linux has no installed distributions. 
Distributions can be installed by visiting the Microsoft Store: 
https://aka.ms/wslstore 

C:\Users\username>wsl --install -d ubuntu 
Downloading: Ubuntu 
Installing: Ubuntu 
Ubuntu has been installed. 
Launching Ubuntu... 
```

安装子系统软件之后的操作跟之前变化甚少，可以直接参考一更[Linux｜WSL打造Windows下更顺畅的双系统](https://mp.weixin.qq.com/s/NOygeTkQlLZFWDzjChfTtQ)

#### 3 遇到的问题

本次安装的ubuntu直接安装了ssh，但是尝试直接start ssh服务的时候遇到了no hostkeys available的问题：
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


子系统Ubuntu也默认安装了python3，
```
左右滑动
[root@wsl:~]# python3 -V 
Python 3.8.2 
[root@wsl:~]# 
```

但是使用**python**的命令依然不好用，
```
左右滑动
[root@wsl:~]# python -V 
Command 'python' not found, did you mean: 
  command 'python3' from deb python3 
  command 'python' from deb python-is-python3 
[root@wsl:~]# 
```

两个方案，一是安装直接安装python，也就是安装python2的版本，另外一个就是直接创建一个软连接，将**python**命令链接到**python3**即可：
```
左右滑动
[root@wsl:~]# which python3 
/usr/bin/python3 
[root@wsl:~]# 
[root@wsl:~]# ln -s /usr/bin/python3 /usr/bin/python 
[root@wsl:~]# ls -l /usr/bin | grep python 
lrwxrwxrwx 1 root   root          23 Jun 23 04:18 pdb3.8 -> ../lib/python3.8/pdb.py 
lrwxrwxrwx 1 root   root          31 Mar 13  2020 py3versions -> ../share/python3/py3versions.py 
lrwxrwxrwx 1 root   root          16 Nov 25 07:34 python -> /usr/bin/python3 
lrwxrwxrwx 1 root   root           9 Mar 13  2020 python3 -> python3.8 
-rwxr-xr-x 1 root   root     5502744 Jun 23 04:18 python3.8 
[root@wsl:~]# 
[root@wsl:~]# python -V 
Python 3.8.2 
[root@wsl:~]# 
```

以上！

继续enjoy WSL吧！

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

