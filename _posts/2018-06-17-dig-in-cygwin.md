---
layout: post
title: Cygwin中无法使用dig的一种解决方案
date: 2018-06-17
categories: blog
tags: [cygwin, dig]
description: dig dig dig dig，一休哥！
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

由于Godaddy申请的老域名（hk314.me）要过期了，而续费一年就要200多大洋，正好阿里域名一直都有活动就申请了一个top十年域名（[hk314.top](http://www.hk314.top)）才148元，如果有想申请域名的可以看看，最近也一直有1元活动，不过好像只能申请一年。

顺便可以通过链接[https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=fgq62bw1](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=fgq62bw1)也可以扫描如下二维码领取阿里云福利：
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAuh79GgmaAfaP2Xoya9PuJFic4xgAaQXxOeJEZKv7Zu13WLxvWYicy5JW83zxssAW2oaNmuMhZNOyw/0?wx_fmt=jpeg" width="30%" />

6月虽然刚刚过去一半，但是已经通过Machine Learning的学习法则可以推测出这一个月都要在非常忙碌中度过了，连续三个周末都要去其他城市度过。然后最近接了一个西班牙的已经开始了很久很久的项目，算是半途入坑的节奏，所以每天从早搞到深夜，感觉睡眠要严重不足了，而且咄咄逼人的各种被MUST，着实有一种随时想要骂人的冲动，是要calm down了。

还好来了一个三天小假期，在一宿火车奔袭之后回到了乡下，然后睡了一整天，算是简单的休整了回来。

#### 1. 想使用Cygwin中的dig
前阵子做Wi-Fi项目的时候想dig一下客户的无线端域名，但是发现cygwin中dig一直为空：
```
$ dig www.baidu.com

$
```

在cygwin中有些工具跟安装的包是不一致的，比如telnet包叫做inetutils，属于net类；dig包名叫做bind-utils，也属于net类。

所以就想更新一下其软件包吧：
```
$ apt-cyg update bind-utils
--2018-06-16 18:56:07--  n.mirrors.hoobly.com//x86_64/setup.bz2
Resolving cygwin.mirrors.hoobly.com (cygwin.mirrors.hoobly.com)...
Connecting to cygwin.mirrors.hoobly.com (rs.hoobly.com)|69.64.41.166|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3707824 (3.5M) [application/x-bzip2]
Saving to: ‘setup.bz2’

setup.bz2                                          100%[=======================================================================================>]   3.54M  18.8KB/s    in 4m 17s

2018-06-16 19:00:25 (14.1 KB/s) - ‘setup.bz2’ saved [824]

Updated setup.ini
```

然鹅并木有神马卵用，在一筹莫展之际发现了一个可以在Windows下使用dig命令的帖子。

#### 2. Windowns下使用dig
在[ftp://ftp.nominum.com/pub/isc/bind9/](ftp://ftp.nominum.com/pub/isc/bind9/)中下载最新BIND的ZIP文件，解压后将以下的库文件和dig.exe拷贝到C:\Windows\System32下即可：
```
$ ls BIND9.11.3.x64/ | grep dll
bindevt.dll
libbind9.dll
libdns.dll
libeay32.dll
libirs.dll
libisc.dll
libisccc.dll
libisccfg.dll
liblwres.dll
libxml2.dll
$ ls BIND9.11.3.x64/ | grep -w  dig.exe
dig.exe
```

如果依然不能在Windows下使用dig的话，可以通过BIND包中的vcredist_x64.exe(64位系统)或者vcredist_x86.exe(32位系统)进行库函数的更新。

之后就可以在Windows的dos中使用dig了：
```
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\UNAME>dig --help
Invalid option: --help
Usage:  dig [@global-server] [domain] [q-type] [q-class] {q-opt}
            {global-d-opt} host [@local-server] {local-d-opt}
            [ host [@local-server] {local-d-opt} [...]]

Use "dig -h" (or "dig -h | more") for complete list of options
```

#### 3. Cygwin下用dig
通过在Windows下添加dig之后就可以在Cygwin下使用了，但是如此长的一个路径也是让使用起来有些许困难了：
```
$ /cygdrive/c/windows/system32/dig www.baidu.com

; <<>> DiG 9.11.3 <<>> www.baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4833
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL:

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
; COOKIE: 3bdc6495674dce26 (echoed)
;; QUESTION SECTION:
;www.baidu.com.                 IN      A

;; ANSWER SECTION:
www.baidu.com.          175     IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       158     IN      A       115.239.211.112
www.a.shifen.com.       158     IN      A       115.239.210.27

;; Query time: 22 msec
;; SERVER: 147.128.5.12#53(147.128.5.12)
;; WHEN: Thu Jun 14 10:41:53 a/S 2018
;; MSG SIZE  rcvd: 113
```

而如果直接使用dig命令，依然跟最初一样，扑了一个空。

通过which dig命令查看了当前dig依然是/usr/bin下的，并没有调用/cygdrive/c/windows/system32。

接下来因为/cygdrive/c/windows/system32在环境变量中，所以尝试删除/usr/bin/中的dig来强制其使用Windows下的。

删除之后再次直接使用dig却提示没有了：
```
$ dig www.baidu.com
-bash: /usr/bin/dig: No such file or directory
```

猜测应该是其安装包bind-utils默认就调用/usr/bin下的dig，如果不安装的话应该会直接全局环境搜索命令的。

所以最后一步就是将Windows下的命令关联到/usr/bin下即可：
```
$ ln -s /cygdrive/c/windows/system32/dig /usr/bin/dig
```

至此Cygwin下dig使用成就达成：
```
$ dig hk314.top

; <<>> DiG 9.11.3 <<>> hk314.top
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48593
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;hk314.top.                     IN      A

;; ANSWER SECTION:
hk314.top.              600     IN      A       185.199.111.153
hk314.top.              600     IN      A       185.199.110.153
hk314.top.              600     IN      A       185.199.108.153
hk314.top.              600     IN      A       185.199.109.153

;; Query time: 87 msec
;; SERVER: 193.181.14.10#53(193.181.14.10)
;; WHEN: Fri Jun 15 06:19:41 a/S 2018
;; MSG SIZE  rcvd: 102
```

#### 4. 总结
两步解决此问题，一就是解决Windows下使用dig命令；二就是使用**ln -s**建立/usr/bin和Windows下的符号连接：
```
 $ ls -l /usr/bin/dig
lrwxrwxrwx 1 User(230687) Group(513) 32 Jun 14 17:44 /usr/bin/dig -> /cygdrive/c/windows/system32/dig
```

以上，还是期待更新Windows 10之后能使用自带的Linux子系统吧^_^_

------------
<p align="center">欢迎关注公众号：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD1g2UjYu4VfCGHmbhgVqOAnNnJQfN7ZhRVUCopYOsfpPtIEB95VNEqu8trAxJXzGDg01ka6z6wzQ/0?wx_fmt=jpeg" width="30%" />

<p align="center">感觉内容不错，读后有收获？欢迎小额赞助：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAzA577Ce49rCLiby9EtT195GRiaqKCT6QCQ5Weia9OZD72MJz4ABlqAy1gbHepk5hHM464hCiarQRI7w/0?wx_fmt=jpeg" width="30%" />



