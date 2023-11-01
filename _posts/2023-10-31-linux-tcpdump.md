---
layout:     post
title:      Linux｜tcpdump的抓包在Wireshark里显示Bogus IPv4 Version
date:       2023-10-31
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - Linux
    - tcpdump
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

通信离不开看包，看包的前提是能抓到有用的包。

tcpdump是在Linux下抓包非常好用的命令，但是最近遇到了一个奇怪的抓包结果，就是使用any抓所有interface的时候会显示IP Invalid：

```
#左右滑动

# tcpdump -i any 
23:45:55.313544 eth0 P   IP8 (invalid)
23:45:55.313544 bond_port P   IP8 (invalid)
```

如果使用-w存成一个pcap文件后，在wireshark里显示Bogus IPv4 version等等无法解读的内容，严重影响了数据包分析。

![@七禾页话][2]

在tcpdump里提了issue后，其开发的大神迅速告诉了原因因为libpcap的版本bug导致的这个问题。

此问题的Issue Link是：https://github.com/the-tcpdump-group/tcpdump/issues/1092，其中有libpcap bug的issue link可以查看更详细的bug内容。

那么如何规避这个问题呢？

如果仔细看tcpdump抓取不同接口的的log，可以看到会提示使用的link-type：
```
#左右滑动
比如抓具体port的时候默认的link-type是EN10MB
$ sudo tcpdump -i eth0 host 1.1.1.1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
```

```
#左右滑动
抓所有port也就是用any的时候，不同Linux版本，具体说是不同tcpdump的版本导致默认的link-type是LINUX_SLL:
$ sudo tcpdump -i any host 1.1.1.1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
或者是LINUX_SLL2：
$ sudo tcpdump -i any host 1.1.1.1
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
```

如果是LINUX_SLL2的时候，并且libpcap的版本低于1.10.2，基本就会出现IP Invlid的问题(Wireshark中显示Bogus IPv4 version)。

解决方案是通过-y参数指定data-link-type为LINUX_SLL:
```
#左右滑动
sudo用户运行：
$ sudo tcpdump -i any -y LINUX_SLL [-w xxx.pcap]
或者root用户直接运行
# tcpdump -i any -y LINUX_SLL [-w xxx.pcap]
```

如果想写shell的时候判断是否要更改data-link-type，想到一个办法是判断当前的libpcap的版本，如果比1.10.2小并且interface是any，就使用LINUX_SLL，否则就用默认的link-type，大概的实现如下：
```
#左右滑动

#!/bin/bash
interface="any"
_usedLinkDataType=""
_currentlibpcapVersion=`tcpdump --version 2>&1 | grep -i libpcap | awk '{print $3}'`
_fixedlibpcapVersion="1.10.2"

function version_lt() { test "$(echo "$@" | tr " " "\n" | sort -rV | head -n 1)" != "$1"; }

if [[ $1 ]];then 
    if [[ $1 != "any" ]] && [[ `ip a s $1 | wc -l` != 0 ]];then
        interface=$1
    elif [[ $1 == "any" ]];then
        interface="any"
    else
        echo "ERROR: interace $1 doesnot exist!!!!"
        echo "Please set a correct interface to be captured!"
        exit 
    fi 
fi 

if version_lt ${_currentlibpcapVersion} ${_fixedlibpcapVersion};then
    if [[ $interface == "any" ]]; then 
        _usedLinkDataType="-y LINUX_SLL"
    fi 
fi 

tcpdump -i ${interace} ${_usedLinkDataType}
```

还有一种解决方案就是升级tcpdump，使libpcap的版本是1.10.2或更高的版本，这个方案会更好，因为使用LINUX_SLL2这个link-type的时候可以具体看到数据包从哪个port进入，从哪个port出。

扩展记录一些本案需要的一些tcpdump参数。

如何查看tcpdump的版本？--version：
```
#左右滑动
$ tcpdump --version
tcpdump version 4.99.1
libpcap version 1.10.1 (with TPACKET_V3)
OpenSSL 3.0.2 15 Mar 2022
ubuntu@VM-16-3-ubuntu:~$ 
```

如何查看对应的接口所支持的data-link-type？-L/--list-data-link-type参数：
```
#左右滑动
$ sudo tcpdump -i any -L
Data link types for any (use option -y to set):
  LINUX_SLL (Linux cooked v1)
  LINUX_SLL2 (Linux cooked v2)

$ sudo tcpdump -i eth0 --list-data-link-type
Data link types for eth0 (use option -y to set):
  EN10MB (Ethernet)
  DOCSIS (DOCSIS) (printing not supported)
```

如何查看tcpdump可以抓取的port列表？--list-interfaces/-D参数：
```
#左右滑动
$ tcpdump --list-interfaces
1.eth0 [Up, Running]
2.any (Pseudo-device that captures on all interfaces) [Up, Running]
3.lo [Up, Running, Loopback]
4.nflog (Linux netfilter log (NFLOG) interface)
5.nfqueue (Linux netfilter queue (NFQUEUE) interface)
6.usbmon1 (USB bus number 1)
 
$ tcpdump -D
1.eth0 [Up, Running]
2.any (Pseudo-device that captures on all interfaces) [Up, Running]
3.lo [Up, Running, Loopback]
4.nflog (Linux netfilter log (NFLOG) interface)
5.nfqueue (Linux netfilter queue (NFQUEUE) interface)
```

以上！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCujePSAwjCut4OPhvG5ZF6OMjFoKIE91bz0eMrBLwqf6euHYYkuhnROXSRtDDtTkEUkkOon5Ln1w/640?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCujePSAwjCut4OPhvG5ZF6skOkk6cCL28VkicqZMAsqWa07RuibG5RDItZkiaWqRMibQviaTmor911Qww/640?wx_fmt=png



