---
layout:     post
title:      Linux｜聊聊Linux系统中的路由策略
date:       2023-11-19
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - Linux
    - Routing
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


路由是沟通任何双边关系的基础，比如现实世界中的邮路，网络世界中的路由，都是用来连接任何需要联系的双方实体。

那么在路由的基础上，会根据流量管控、路由分发、负载均衡等不同需求对路由选择进行规划，这便是路由策略(Routing Policy)。

Routing Policy是一种网络管理技术，通过一系列工具或方法对路由进行各种控制的“策略”。这种策略能够影响到路由产生、发布和选择等，进而影响报文的转发路径，也就是选择不同的路由。这些工具包括ACL、route-policy、ip-prefix、filter-policy等，这些方法包括对路由进行过滤，设置路由的属性等。

Routing Policy允许管理员定义规则和条件，从而确定网络上的数据包应该如何转发。这些规则可以基于多种因素，如源地址、目标地址、服务类型等。

那么在Linux系统里如何设置路由策略呢？

首先要了解的是Linux系统的路由也是有很多的路由表存在的，默认的配置基本如下：
```bash
###左右滑动
ubuntu@VM-16-3-ubuntu:~$ cat /etc/iproute2/rt_tables
#
# reserved values
#
255     local
254     main
253     default
0       unspec
#
# local
#
#1      inr.ruhep

```

其中，local编号255是本地路由表，main编号254是主要路由表，default编号253是默认路由表，unspec编号0是未指定的表。

一般来说我们在Linux系统中打印路由表是直接使用`ip route`这条命令的，更准确的来说`ip route`其实是`ip route show table main`的简化，也就是说直接运行`ip route`打印的就是主路由表里的路由：

![@七禾页话][2]


local路由表里记录的是本地路由，举个最直接的例子就是loopback地址的路由就在local表里，比如：

查看当前loopback接口的配置：
```bash
###左右滑动
ubuntu@VM-16-3-ubuntu:~$ ip a s lo
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
```

查看当前的local路由表内容：
```bash
###左右滑动
ubuntu@VM-16-3-ubuntu:~$ ip route show table local
local 10.0.16.3 dev eth0 proto kernel scope host src 10.0.16.3 
broadcast 10.0.19.255 dev eth0 proto kernel scope link src 10.0.16.3 
local 10.42.0.0 dev flannel.1 proto kernel scope host src 10.42.0.0 
local 10.42.0.1 dev cni0 proto kernel scope host src 10.42.0.1 
broadcast 10.42.0.255 dev cni0 proto kernel scope link src 10.42.0.1 
local 127.0.0.0/8 dev lo proto kernel scope host src 127.0.0.1 
local 127.0.0.1 dev lo proto kernel scope host src 127.0.0.1 
broadcast 127.255.255.255 dev lo proto kernel scope link src 127.0.0.1 
local 172.17.0.1 dev docker0 proto kernel scope host src 172.17.0.1 
broadcast 172.17.255.255 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
ubuntu@VM-16-3-ubuntu:~$ 
```

给loopback加一个地址是1.1.1.1/32：
```bash
###左右滑动
ubuntu@VM-16-3-ubuntu:~$ sudo ip addr add 1.1.1.1/32 dev lo
ubuntu@VM-16-3-ubuntu:~$ ip a s lo
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 1.1.1.1/32 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
ubuntu@VM-16-3-ubuntu:~$ 
```

再次打印local路由表的内容，可以看到loopback新加的ip 1.1.1.1已经在local路由表里了：
```bash
###左右滑动
ubuntu@VM-16-3-ubuntu:~$ ip route show table local
local 1.1.1.1 dev lo proto kernel scope host src 1.1.1.1 
local 10.0.16.3 dev eth0 proto kernel scope host src 10.0.16.3 
broadcast 10.0.19.255 dev eth0 proto kernel scope link src 10.0.16.3 
local 10.42.0.0 dev flannel.1 proto kernel scope host src 10.42.0.0 
local 10.42.0.1 dev cni0 proto kernel scope host src 10.42.0.1 
broadcast 10.42.0.255 dev cni0 proto kernel scope link src 10.42.0.1 
local 127.0.0.0/8 dev lo proto kernel scope host src 127.0.0.1 
local 127.0.0.1 dev lo proto kernel scope host src 127.0.0.1 
broadcast 127.255.255.255 dev lo proto kernel scope link src 127.0.0.1 
local 172.17.0.1 dev docker0 proto kernel scope host src 172.17.0.1 
broadcast 172.17.255.255 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
ubuntu@VM-16-3-ubuntu:~$ 

```

查看main路由表里是否有1.1.1.1，对比local路由表，答案是main路由表里没有loopback的地址路由：
```bash
###左右滑动
ubuntu@VM-16-3-ubuntu:~$ ip route | grep 1.1.1.1
ubuntu@VM-16-3-ubuntu:~$ 
ubuntu@VM-16-3-ubuntu:~$ ip route show table local| grep 1.1.1.1
local 1.1.1.1 dev lo proto kernel scope host src 1.1.1.1 
ubuntu@VM-16-3-ubuntu:~$ 
```

通过ip route get ip可以看到其在local路由表：
```bash
###左右滑动
ubuntu@VM-16-3-ubuntu:~$ ip route get 1.1.1.1
local 1.1.1.1 dev lo src 1.1.1.1 uid 1000 
    cache <local> 
ubuntu@VM-16-3-ubuntu:~$ 
```

了解到了Linux系统下的多路由表，我们再来看看`rt_tables`文件，这个文件就是路由表编号和名字的对应关系：
```bash
### 左右滑动
<路由表编号>    <路由表名>
```

理论上`rt_tables`中可以定义多个路由表，但是实际上由系统支持的路由表数量是有限的，取决于操作系统的配置和内核参数，一般可以定义1-255一共255张路由表，剔除预留的三张表之后，实际上可以自由定义1-252即252张路由表。

然后我们再来看看如何在Linux系统中定义Routing Policy。

在配置Routing Policy时，以下是一些基本的元素：

- **策略条件：** 规定何时应用Routing Policy，例如基于源地址、目标地址、服务类型等。
  
- **动作：** 确定数据包应该如何处理，例如转发到特定的下一跳、阻止或允许传输等。

让我们通过一个简单的场景来说明如何配置Routing Policy。

**场景：** 一个Linux服务器上有两个不同的互联网连接，一个是有线高速光纤，另一个是无线便携式4G宽带。我们希望某些类型的流量通过光纤，而其他类型的流量通过4G。

**配置示例：**

首先必须要在rt_tables文件中定义没有冲突编号的路由表:
```bash
### 左右滑动
sudo cat /etc/iproute2/rt_tables | grep -E "^10 custom_table10"
echo "10 custom_table10" | sudo tee -a /etc/iproute2/rt_tables

sudo cat /etc/iproute2/rt_tables | grep -E "^20 custom_table20"
echo "20 custom_table20" | sudo tee -a /etc/iproute2/rt_tables

# 也可以直接使用如下的两条带if判断的小shell
table_id_name="10 custom_table10";if [ `sudo cat /etc/iproute2/rt_tables | grep -E "^${table_id_name}" -c` == 0 ];then echo "${table_id_name}" | sudo tee -a /etc/iproute2/rt_tables;else echo "Table ${table_id_name} existed!";fi

table_id_name="20 custom_table20";if [ `sudo cat /etc/iproute2/rt_tables | grep -E "^${table_id_name}" -c` == 0 ];then echo "${table_id_name}" | sudo tee -a /etc/iproute2/rt_tables;else echo "Table ${table_id_name} existed!";fi
```

然后定义策略条件和动作：
```bash
### 左右滑动
# 定义Routing Policy
ip rule add from 192.168.1.0/24 table 10
ip rule add from 192.168.2.0/24 table 20

# 定义路由表
ip route add default via 192.168.1.1 dev eth0 table 10
ip route add default via 192.168.2.1 dev wlan0 table 20
```
使用编号或者路由表名都是等效的，下边是使用​路由表名字的例子：​
```bash
### 左右滑动
# 定义Routing Policy
ip rule add from 192.168.1.0/24 table custom_table10
ip rule add from 192.168.2.0/24 table custom_table20

# 定义路由表
ip route add default via 192.168.1.1 dev eth0 table custom_table10
ip route add default via 192.168.2.1 dev wlan0 table custom_table20
```

这个例子中，我们通过Routing Policy将来自两个不同子网的流量分别路由到两个不同的路由表：

- 如果数据包的源地址是192.168.1.0/24中的ip，走路由表10，从eth0接口出去。

- 如果数据包的源地址是192.168.2.0/24中的ip，走路由表20，从wlan0接口出去。

配置完成后可以使用以下命令验证配置是否生效：

```bash
### 左右滑动
# 这将显示当前定义的策略规则
ip rule show
```
```bash
### 左右滑动
# 这将显示名为custom_table10和custom_table20的自定义表中的路由规则。
ip route show table custom_table10
ip route show table custom_table20
```

那么路由策略有哪些应用场景呢？Routing policy在Linux系统中的使用场景有很多，其中一些典型的情况包括：

1. **多路径负载均衡：** 如果你有多个网络接口连接到不同的网络，你可能希望在这些接口之间分配流量，以实现负载均衡。通过使用不同的路由表和策略规则，你

2. **VPN和专线冗余：** 在使用VPN或专线连接到不同的网络提供商时，你可能需要设置冗余路径，以确保在一个连接失败时能够切换到另一个连接。

3. **策略路由：** 有时你可能需要根据数据包的特定属性（如源IP地址、目标IP地址、服务类型等）来选择不同的路由表。

4. **QoS优化：** 根据服务类型或数据包优先级，可以通过Routing Policy确保关键服务的低延迟和高带宽。

5. **故障切换：** 当一个网络路径出现故障时，Routing Policy可以自动切换到备用路径，提高网络的可用性。

这些只是一些可能的使用场景和示例。实际上，Routing policy非常强大，可以根据需要进行高度定制，以满足复杂网络环境的需求。

虽然Routing Policy提供了强大的灵活性，但需要小心使用，以避免配置错误导致网络问题；对于复杂的Routing Policy配置，需要始终保持详细的文档记录，比如路由规划、配置文件、实施log，以便未来的维护和故障排除；同时需要注意的是，使用命令行配置的rule和路由是临时生效的，系统重启或者网络服务重启后就会丢失，因此需要根据不同系统的设定将配置写到对应的文件中，以便系统重启或者网络服务重启后路由策略依然存在。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCmSwSyrQEIDQIblccc1dFkTtBBmnl1xVo6vNT25t8E35ZwgIrXu2zp4bicD4qMiazsdb1bN0HMcZRg/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBv1qnv6B9CSWL6ZPAEZib6iaaibgro2R8XJmoZ2bWDoibGrVJibkxMqiaLFYkiaIq7SKkYotGEBTNY195yQ/640?wx_fmt=png&amp;from=appmsg

