---
layout:     post
title:      Linux｜反向路径过滤(rp_filter)导致Linux业务不通
date:       2023-10-27
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

![@七禾页话 城市大连][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>


最近项目遇到了一个非常神奇的问题，细节不过多描述了，问题大概跟下图中的拓扑类似，就是路由器将数据包发给了服务器的eth1口，但是服务器的路由是从eth2出去，导致了此服务器不响应外部任何的业务请求。

![@七禾页话][2]

理论上来说，服务器从任何口收到包之后，可以通过查询路由表后从任何口发出响应信息才对，到这里，如果对Linux很熟悉的人可能想到了问题所在，就是今天的标题反向路径过滤——Reverse Path Filtering——起了作用。

那么什么是Reverse Path Filtering（rp_filter）呢？

rp_filter是Linux Kernel以及众多网络设备采用的一种保护机制，以用来检查收到的数据包的原地址是否可路由，也就是如果开启了rp_filter功能，当服务器收到一个数据包之后，将此数据包的源地址和目的地址对调后也就是构建的响应数据包，依然从收到数据包的网卡发出去，就表示此消息通过了rp_filter的检测。

以ipv4的rp_filter为例，可以配置的值如下：

| Configured Value | Description |
| --- | ----------- |
| 0 | No source validation |
| 1 | Strict mode as defined in RFC3704 Strict Reverse Path, each incoming packet is tested against the FIB and if the interface is not the best reverse path the packet check will fail. By default failed packets are discarded. |
| 2 | Loose mode as defined in RFC3704 Loose Reverse Path, each incoming packet's source address is also tested against the FIB and if the source address is not reachable via any interface the packet check will fail. |

以上来自https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt

中文含义
| 可配置的值 | 数值的含义 |
| --- | ----------- |
| 0 | 关闭反向路由校验 |
| 1 | 开启严格的反向路由校验，对每一个进来的数据包，如果反向路由不是通过收到数据的包接口，校验失败后收到的数据包丢弃 |
| 2 | 不严谨的反向路由校验，对每个进来的数据包，校验源地址是否可达，如果反向路由可以通过任何接口到达，那么校验成功，否则校验失败后数据包丢失 |

简单来说，就是如果配置成1，那么就是从哪个接口进来的包，其回应的包就要从那个接口出去；如果配置成2，就是看一下路由表如果有路由匹配上就可以从匹配的路由接口响应，不管这个接口是否是收到的数据包的接口；如果是0，就是不做任何校验，收到包就根据路由表响应。

目前大部分Linux默认的rp_filter配置是1，所以对于最开始的拓扑图来说，如果Linux服务器里只有一条默认路由从eth2出去，那么服务器收到的所有到30.30.30.1的数据包都会被其丢弃，现象就是服务器不响应任何业务请求。

tcpdump如下：

```
#左右滑动
LinuxServer:~ # tcpdump -i any host 30.30.30.1
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
04:32:29.987277 eth1 In  IP 40.40.40.40.49791 > 30.30.30.1.diamondport: Flags [S], seq 1504279942, win 64896, options [mss 1352,nop,wscale 8,nop,nop,sackOK], length 0
04:32:30.987277 eth1 In  IP 40.40.40.40.49791 > 30.30.30.1.diamondport: Flags [S], seq 1504279942, win 64896, options [mss 1352,nop,wscale 8,nop,nop,sackOK], length 0
04:32:31.987277 eth1 In  IP 40.40.40.40.49791 > 30.30.30.1.diamondport: Flags [S], seq 1504279942, win 64896, options [mss 1352,nop,wscale 8,nop,nop,sackOK], length 0
```

当然最佳的solution就是修正路由器上的路由，将到30.30.30.1的下一跳改成eth2的ip。

另外也可以关闭服务器上的rp_filter，通过sysctl这个命令的-w参数修改，操作如下：

```
#左右滑动
查看当前的值：
LinuxServer:~ # sysctl net.ipv4.conf.all.rp_filter
net.ipv4.conf.all.rp_filter = 1
LinuxServer:~ # sysctl net.ipv4.conf.eth1.rp_filter
net.ipv4.conf.eth1.rp_filter = 1

修改参数值：
LinuxServer:~ # sysctl -w net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.all.rp_filter = 0
LinuxServer:~ # sysctl -w net.ipv4.conf.eth1.rp_filter=0
net.ipv4.conf.eth1.rp_filter = 0

再次查看当前的值
LinuxServer:~ # sysctl net.ipv4.conf.all.rp_filter
net.ipv4.conf.all.rp_filter = 0
LinuxServer:~ # sysctl net.ipv4.conf.eth1.rp_filter=0
net.ipv4.conf.eth1.rp_filter = 0
```

这里需要注意的是，要修改两个地方，一个是全局all的rp_filter，一个是收到数据包接口的rp_filter。

这样的修改后，不影响其他接口的rp_filter。

修改后tcpdump可以看到如下的结果，数据包从eth1进，回复的包从eth2出：

```
#左右滑动
LinuxServer:~ # tcpdump -i any host 30.30.30.1
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
04:33:11.261550 eth1 In  IP 40.40.40.40.64401 > 30.30.30.1.diamondport: Flags [S], seq 4008651141, win 64896, options [mss 1352,nop,wscale 8,nop,nop,sackOK], length 0
04:33:11.261699 eth2 Out IP 30.30.30.1.diamondport > 40.40.40.40.64401: Flags [S.], seq 2852566551, ack 4008651142, win 64400, options [mss 1400,nop,nop,sackOK,nop,wscale 7], length 0
04:33:12.261550 eth1 In  IP 40.40.40.40.64401 > 30.30.30.1.diamondport: Flags [S], seq 4008651141, win 64896, options [mss 1352,nop,wscale 8,nop,nop,sackOK], length 0
04:33:12.261699 eth2 Out IP 30.30.30.1.diamondport > 40.40.40.40.64401: Flags [S.], seq 2852566551, ack 4008651142, win 64400, options [mss 1400,nop,nop,sackOK,nop,wscale 7], length 0
04:33:13.261550 eth1 In  IP 40.40.40.40.64401 > 30.30.30.1.diamondport: Flags [S], seq 4008651141, win 64896, options [mss 1352,nop,wscale 8,nop,nop,sackOK], length 0
04:33:13.261699 eth2 Out IP 30.30.30.1.diamondport > 40.40.40.40.64401: Flags [S.], seq 2852566551, ack 4008651142, win 64400, options [mss 1400,nop,nop,sackOK,nop,wscale 7], length 0
```

如何查看所有接口的rp_filter配置呢？可以使用sysctl -a查看，也可以直接查看对应接口的配置文件（写了一个for循环）：

```
#左右滑动
sysctl -a | grep -i "\.rp_filter"

for i in `ls /proc/sys/net/ipv4/conf/`;do echo "# $i rp_filter value";cat /proc/sys/net/ipv4/conf/${i}/rp_filter;done `
```

那么rp_filter有什么好处呢？生产里有一句话就是安全最重要，同样的在当前网络如此发达的情况下，网络安全也至关重要，rp_filter就是一个安全的小螺丝：

1. 减少DDos攻击：一般来说网络规划的时候是严格规划收发路径的，所以对于很多路由来说那么就是从哪儿进从哪儿出，一旦出现不匹配的情况下导致反向路径不合适，则直接丢弃数据包，避免过多的无效连接消耗系统资源。

2. 防止IP Spoofing（IP欺骗）：如果客户端伪造的源IP地址对应的反向路径不在路由表中，或者反向路径不是最佳路径，则直接丢弃数据包，不会向伪造IP的客户端回复响应。

以上！我没有尝试在当前的例子下改成2是否成功，大家觉得呢？如果成功了，那么2到底跟0有什么区别呢？欢迎留言讨论或者分享您的经验和理解。

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAWnvF4a5O1pNposYE1FbsxIu1RPz95IibYrIibDxicPPuAiaPDrvEiaKqGk3ua9G6dkd0PUkjSSibk61kw/640?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAWnvF4a5O1pNposYE1FbsxfVZn79VuibloWYFQziaUAYQqXKicXEIyTsFqS6w7hiaibvq7bA0hWOSWXmw/640?wx_fmt=png

