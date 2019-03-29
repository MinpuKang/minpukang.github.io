---
layout:     post
title:      换换Cygwin的Terminal吧
date:       2017-12-30
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - Linux
    - cygwin
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

17年最后一篇技术相关帖来介绍一下使用secureCRT等工具连接cygwin吧。

cygwin可以说是一个很好的在Windows上迅速模拟和学习Unix/Linux的工具了，不过其对应的Terminal却弊端重重，比如复制粘贴记log等等都不是很方便。

今天就介绍一下打开其ssh的功能后使用secureCRT等工具来使用cygwin。

##### 1、安装openssh
一般来说很少有人在默认安装的时候会选择安装这个service，所以第一步就要先安装openssh了，当然如果安装cygwin的时候选择了全安装或者很自然的安装了这个service，那么可以pass这个章节了。

那么如何安装openssh呢？

首先依然是打开cygwin的安装工具比如64位的setup-x86_64，一路下一步直到**Select Packages**，这里默认显示的Pending View，只能看到需要更新的包，所以需要先将View选择为**Full**：

![安装1][1]

接下来在**Full**的View下搜索**openssh**相关的安装包，会有如下四个：

![安装1][2]

单击各个包的New字段后Skip变成了对应版本，这样代表选择安装对应的包，然后点击下一步进行安装即可：

![安装1][3]

![安装1][4]

等待安装完成点击Finish就结束了整个安装openssh的过程：

![安装1][5]


#### 2、配置sshd

首先需要使用**Administrator**的权限打开**Cygwin Terminal**：

![配置1][6]

打开之后运行**ssh-host-config**命令，然后按照提示配置就好，以下是我的配置过程：

![配置2][7]

![配置3][8]

![配置4][9]

配置完成后，使用**net start sshd**开启sshd服务并需要得到successfully的提示：

![配置5][10]

#### 3、SecureCRT连接cygwin

cygwin开启sshd服务之后，其ip为127.0.0.1也就是localhost，默认的port为22，用户名密码为Windows的用户名和其密码（小提示：在SecureCRT的Authentication中将Password移动到第一个防止其尝试其他的方式造成不必要的浪费）：

![连接1][11]

然后点击Connect连接后输入此用户在Windows中的密码：

![连接2][12]

输入密码后点击OK就可以完美使用SecureCRT连接到Cygwin了：

![连接3][13]

#### 4、后记

1、如果担心127.0.0.1:22的组合被其他程序占用，那么可以修改其ssh的端口，配置文件为**/etc/sshd_config**，其中参数为**Port**：

![后记1][14]

更新配置后需要使用如下命令重启sshd服务使其生效：
```
net stop sshd
net start sshd
```

2、这样之后就可以使用任何的登录软件来使用Cygwin了，并且重启电脑后不必再打开Cygwin Terminal，也就是其他的软件代替了Terminal，让操作更方便快捷。

以上。

------------
<p align="center">欢迎关注公众号，摄影，旅行，瞎聊，等等等：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD1g2UjYu4VfCGHmbhgVqOAnNnJQfN7ZhRVUCopYOsfpPtIEB95VNEqu8trAxJXzGDg01ka6z6wzQ/0?wx_fmt=jpeg" width="30%" />

  [1]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYlg9ywc8kHnSRUfDy6Haw46ndqMnppElPHYib6lxwm6ZicUy3fZgtEKfg/0?wx_fmt=png
  [2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYibCibLicY2dCnBHlIDEelnMMjiackHj0ESG7XCcoe8IMUdc1OVmRicuCia1g/0?wx_fmt=png
  [3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYC9dQ8kCfrakBBvoiaMxePac2OAfRnpCDLkOibXdSUhHBLjTzONMxHlWw/0?wx_fmt=png
  [4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYab4Dun5Le7Qe01iaPBS7AHFVsQGTMrNdgcVqFfiahBG9CrRFmQQzMkWQ/0?wx_fmt=png
  [5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYmbGTxKib0VGDbGicicm93ian6HdxaqMoh4dluNzO3c8MsrzW9tKkvMK6Og/0?wx_fmt=png
  [6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYoRDxEHiaLQDVJPrwice2y4dCjlZKyC82x7Isliaqp5rQFQzbxbHb3lGrQ/0?wx_fmt=png
  [7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYIRM1VZYm9GzYAju1UQvKw2jNv2icdRdf0hUehJhDetrnKx9cGK3wvUw/0?wx_fmt=png
  [8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyY5VOg3LwHZnQWbkMGESKZicW3uIwQ7OicibnUJTCGnff5lA5TwNfSECglA/0?wx_fmt=png
  [9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYhALmiaHcYvf4rbtaIJ5hnRWVzjMwJ7SF1kZw7GNHUUdhkdO2zMI2ibBQ/0?wx_fmt=png
  [10]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYLhgdqaZ0LHvHiajic1vLiarNiarMwoYPZY9tEGjFju4iajiaHTiau0b5gLwBA/0?wx_fmt=png
  [11]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYvQHxDMXUYBjEj1sRBwaN8C5eA2icKNwDLUyv0oFbicvyconf0S8zCLYg/0?wx_fmt=png
  [12]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYMmrjcHzMZ6P7NOhaVKbjUiaTxuYukGrLS7DjUib54iaqVNXM31AtVKYmA/0?wx_fmt=png
  [13]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYQhiaeibbpbzxFc4iaRl5adt83SpMWY7DhRatQ40bfIfl1geVY3zAca7sg/0?wx_fmt=png
  [14]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC3PbURGLCuCBfMAgayrOyYORkkREicZH9iaSHLhiaGw4bWA7jcdqDARwBTNZMyAXJ1cNeYjQWLHns3w/0?wx_fmt=png


