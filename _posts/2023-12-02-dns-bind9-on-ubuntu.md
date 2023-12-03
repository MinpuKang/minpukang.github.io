---
layout:     post
title:      通信｜启动一个自己的DNS服务器呀？
date:       2023-12-02
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 通信
    - DNS
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

技术的学习，唯手熟尔，学习再多的理论，不通过实践加以强化，是无法深入理解和掌握的。

所以针对DNS，整理了一份儿在ubuntu系统中安装bind9并完成配置的完整流程。

其他Linux发行版本可以自行搜索如何安装。

#### 安装Bind9

首先检测系统中是否有bind9服务：
```bash
###左右滑动
$ systemctl status bind9
Unit bind9.service could not be found.
```

没有服务的前提下需要进行安装，Linux系统软件安装最烦人的地方就是依赖包，如果系统可以联互联网就万事大吉了，通过一个命令即可安装：
```bash 
###左右滑动
$ sudo apt-get install bind9
```

但是很多的时候因种种原因，无法联网，所以需要手动安装bind9，依赖包是一个非常令人头疼的事情，但是只要功夫深、依赖就不是问题，一个一个排雷。

首先要先看当前系统中是否已经安装了某些bind9的服务，比如下边的结果显示已经安装了部分服务，版本是9.18.18的，这些服务是给dns相关业务使用的，比如允许我们可以运行dig或者nslookup的命令：
```bash
###左右滑动
dpkg -l | grep -i <string>

e.g.:
#  dpkg -l | grep bind9
ii  bind9-dnsutils                             1:9.18.18-0ubuntu0.22.04.1              amd64        Clients provided with BIND 9
ii  bind9-host                                 1:9.18.18-0ubuntu0.22.04.1              amd64        DNS Lookup Utility
ii  bind9-libs:amd64                           1:9.18.18-0ubuntu0.22.04.1              amd64        Shared Libraries used by BIND 9
```

然后需要到网站[https://pkgs.org/](https://pkgs.org/)中搜索`bind9`，找到跟已经安装好的软件版本一样的bind9安装包`bind9_......deb`，比如`bind9_9.18.18-0ubuntu0.22.04.1_amd64.deb`是跟上述打印结果一样的版本，都是`9.18.18-0ubuntu0.22.04.1`。

需要注意的是如果不安装一致的版本会导致bind9服务虽然可以使用，但是其他的服务比如dig等命令无法使用。

接下来将下载的deb包传到系统中，使用`dpkg -i <包名字>`（需要使用root或者sudo）进行安装，这个时候如果没有安装依赖包，会提示并终止安装：
```bash
###左右滑动
Selecting previously unselected package bind9.
(Reading database ... 125142 files and directories currently installed.)
Preparing to unpack .../bind9_9.18.18-0ubuntu0.22.04.1_amd64.deb ...
Unpacking bind9 (1:9.18.18-0ubuntu0.22.04.1) ...
dpkg: dependency problems prevent configuration of bind9:
 bind9 depends on bind9-utils (= 1:9.18.18-0ubuntu0.22.04.1); however:
  Package bind9-utils is not installed.
 bind9 depends on dns-root-data; however:
  Package dns-root-data is not installed.

dpkg: error processing package bind9 (--install):
 dependency problems - leaving unconfigured
Processing triggers for ufw (0.36.1-4ubuntu0.1) ...
Processing triggers for man-db (2.10.2-1) ...
Errors were encountered while processing:
 bind9
```

依赖包可以在`https://pkgs.org/`搜索结果中找到对应的link：
![@七禾页话][2]

一个一个的安装后直到`bind9_.....`的包安装成功，通过`systemctl status bind9`查看状态是否是`active(running)`，如果不是可以尝试`systemctl restart bind9`启动。

当在ubuntu上安装完bind9之后，就可以配置规划的域名IP解析了，这就需要了解一下其配置文件的架构。

#### 配置文件

bind9的配置文件可以分为两部分，一部分是全局配置，一部分是zone文件配置。

全局配置的文件默认在`/etc/bind/`文件夹中，需要了解主要的三个文件：

1. `named.conf`: 主要文件，关联了其他配置文件，基本不用改动。
2. `named.conf.options`：这个文件定义了zone文件的文件夹，监听的IP、Port，以及forwarders DNS——即如果本dns没有解析需要向forwarders dns转发解析请求，比如下边的配置说明了zone的文件需要放到`/var/cache/bind`文件夹下，并且监听所有IP，同时没有设定上级forwarding dns，此文件按需改动，实验中基本不动：
```bash
###左右滑动

acl "trusted" {
        localhost;
};
options {
        directory "/var/cache/bind";

        listen-on { any; };
        allow-transfer { none; };

        //forwarders {
        //    1.1.1.1;
        //    8.8.8.8;
        //};

};
```
3. `named.conf.local`：这文件里主要定义了zone和其文件的对应关系，比如下边的配置中定义了两个zone，其中zone `hk314.top`的文件是`db.hk314.top`：
```bash 
###左右滑动

//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "hk314.top" {
    type master;
    file "db.hk314.top";
};
zone "mnc001.mcc666.3gppnetwork.org" {
    type master;
    file "db.mnc001.mcc666.3gppnetwork.org";
};
```

上边三个文件配置完之后就可以定义zone文件，即域名和IP的解析文件，zone文件需要放到`/etc/bind/named.conf.options`所定义的文件夹下默认是`/var/cache/bind`，zone文件的名字需要跟`/etc/bind/named.conf.local`中定义的一致比如`db.hk314.top`：
```bash
###左右滑动
# cd /var/cache/bind/
# ls -l
total 12
-rw-r--r-- 1 root root  747 Dec  2 21:23 db.hk314.top
```

zone文件中的内容最主要的就是域名和IP的对应关系，还有其他的比如TTL、ORIGIN等，比如下边的例子，如果你想换成一个新的zone，只需要将`ORIGIN`和对应的解析换掉即可，另外这里需要特别注意Trailing Dot的使用：
```bash
;左右滑动
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     localhost. root.localhost. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      localhost.
@       IN      A       127.0.0.1
@       IN      AAAA    ::1

;$ORIGIN    hk314.top.

;===========================================================
;DNS configuration: services including all are examples here
;===========================================================

test.hk314.top.                                 IN A   1.1.1.10
```

任何修改都需要重启服务使其生效：
```bash
### 左右滑动
# Check status
systemctl status bind9.service

# Restart bind9
systemctl restart bind9.service

# Check status after restart
systemctl status bind9.service
```

#### 小知识点

DNS中定义的zone就是dns domain，也有的称呼为realm，或者FQDN、域名等等。zone定义成什么，根据域名从右往左来规划，比如我有域名`www.test.top`和`www.example.top`，如果想简化DNS中的配置，可以在`named.conf.local`中定义个`top`的zone：
```bash 
###左右滑动

zone "top" {
    type master;
    file "db.top";
};
```

然后在`/var/cache/bind`文件夹下创建文件`db.top`并定义所有的域名解析：
```bash
;左右滑动
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     localhost. root.localhost. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      localhost.
@       IN      A       127.0.0.1
@       IN      AAAA    ::1

;$ORIGIN    top.

;===========================================================
;DNS configuration: services including all are examples here
;===========================================================

www.test.top.                               IN A   2.1.1.10
www.example.top.                            IN A   3.1.1.10
```

但是在一个DNS中要防止域名重叠，比如针对`hk314.top`已经有了自己的文件`db.hk314.top`，那么所有的`hk314.top`相关的域名都要定义到自己的文件中，如果定义到`db.top`中，解析就会失败，因为DNS解析是要从右到左做域名匹配，永远使用最长匹配的zone定义。

最后，对于依赖包我创建了一个Repository [https://github.com/MinpuKang/DNS-with-Bind9-on-Ubuntu](https://github.com/MinpuKang/DNS-with-Bind9-on-Ubuntu)，基本可以下载直接使用，不过bind9的依赖包虽然也比较多，但是很多已经预先安装了，剩下的就比较少，一个一个下载安装也未尝不可，我之前安装wireshark的时候最多下载了24个依赖包，一个一个下的。。。

到这里DNS相关的内容基本就告一段落了，如果使用过程中有什么问题，可以留言交流，互相学习。

- [通信｜DNS配置生成工具大进化！](https://mp.weixin.qq.com/s/USgpqysaXcUReoU9SPVy9Q)
- [通信｜DNS域名中的点儿和通配符](https://mp.weixin.qq.com/s/lZwzo_nA-7QyCa-Q0gXMGQ)

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCBdCru7ia443bDvMfOcX6uMcLe8Z2WQeVYWnQvQ2MibpBgZ9BJsLPEY4J934CpReW2IeCiakicwv0EDQ/640?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCNT4cF4TWthyicp4ZYGQooqsT7ZPnGiaibhFxIpxTTEicdl10KqIOp4NTcqCxic2ekrbXYQANF7GObFfQ/640?wx_fmt=png&amp;from=appmsg



