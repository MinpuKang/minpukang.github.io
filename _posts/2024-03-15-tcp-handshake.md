---
layout:     post
title:      通信｜TCP协议为什么要进行三次握手？为什么又有四次握手？
date:       2024-03-15
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 通信
    - TCP
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

![@七禾页话，拍摄于大连皮口港火箭 @魔方摄影][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

整理一篇知乎上的回答。关于TCP/IP协议为什么要进行三次连接?

## “三”是个神奇的数字

三这个数字在ICT或者各个行业都是非常有意思的存在。
![@七禾页话][2]

比如”三人行必有我师“、”三人成虎“、”一鼓作气、再而衰三而竭”等。

文学中的排比句大部分都是三句，两句没气势，四句又显得臃肿没有气势了。

回到ICT，最近这些年比较火的就是云和云原生了，其中有一个很重要的概念是controller，默认都要启动三个，一个controller没有冗余备份，两个controller会发生脑裂，所以三个是最合理的，即保证了冗余，又防止了脑裂，在资源最大化的使用下，三个controller是最好的选择。

什么是脑裂呢？

脑裂就是两个controller正常情况下是一直有沟通的，互相通信，谁在某一个阶段是主的，另一个就是备的，但是如果在某些特殊情况，两个controller彼此失去了联系，那么都会认为自己是主的，就会给小弟们一起发命令，导致一个小弟收到了两个指令，小弟就蒙圈了：我到底要听谁的？然后系统就进入脑裂的状态了，即两个控制器彼此失去联系后都争夺老大的地位。

如果是三个controller就在最大程度上防止了脑裂的发生，三个controller之间是互相通信的，举个例子，比如A、B、C三个Controller，A和B、C失去了通信，但是B和C之间是相通的，它俩就会将A刨除，告诉小弟们，不要听A的了，听我俩的就行，防止了脑裂的发生。当然在非常极端特殊的情况下，三个都互相失联了，极端特殊情况可能是不可抗力了。

## TCP三次握手建立连接

回到题主的问题，问题有歧义的，TCP/IP是一个协议栈，其中传输层的TCP协议在建立连接的时候是三次握手，但是如果传输层使用UDP协议并没有三次握手。

那么TCP为什么建立的时候要进行三次握手呢？
![@七禾页话][3]

类比一下打电话：
```
A：喂？（意思是：你好，能听到吗？）
B：喂喂！（意思是：可以听到，你可以听到我说话吗？）
A：喂！（意思是：我也可以听到）
然后开始说正事儿。
```

TCP三次握手就是这样的一个过程：
```
Client：syn-你好，可以听到吗？
Server：ack-可以听到，syn-你可以听到吗？
Client：ack-我也可以听到
开始传具体的数据。
```

非常自然的一个流程，而且保证了连接的建立，所以说TCP是可靠性传输的协议：在确认双方都知道了对方即建立了连接后才开始传输数据。

三次握手少一次，无法做到双方互相的确认，多一次呢，又浪费了时间和带宽，所以最优的选择就是三次握手了。

## TCP四次握手关闭连接

那么TCP连接关闭的时候为什么就是四次了呢？
![@七禾页话][4]

依然是类比打电话：
```
A：我要说的说完了，挂了哈？
B：好的呀。
B：我要说的也说完了，挂了吧。
A：好的，拜拜。
```

TCP四次握手就是：
```
Client: fin-数据发完了，关闭连接
Server: ack-确认关闭
Server: fin-我这边数据也发完了，关闭连接吧
Client: ack-好的，确认关闭
```

也是一个自然且安全的过程，保证了双方维护的连接都关闭了。

## 为什么连接的时候是三次握手，关闭的时候却是四次握手？

建立连接时因为当Server收到Client的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的，所以建立连接只需要三次握手。

由于TCP协议是一种面向连接的、可靠的、基于字节流的运输层通信协议，TCP是全双工模式。
这就意味着，关闭连接时，当Client端发出FIN报文段时，只是表示Client端告诉Server端数据已经发送完毕了。当Server端收到FIN报文并返回ACK报文段，表示它已经知道Client端没有数据发送了，但是Server端还是可以发送数据到Client端的，所以Server很可能并不会立即关闭SOCKET，直到Server端把数据也发送完毕。

当Server端也发送了FIN报文段时，这个时候就表示Server端也没有数据要发送了，就会告诉Client端，我也没有数据要发送了，之后彼此就会愉快的中断这次TCP连接。

以上就是TCP的三次握手建立连接和四次握手关闭连接简单介绍。

## 看看trace

Linux里可以使用`netcat`命令尝试建立tcp连接的测试，格式如下：
```
###左右滑动
netcat <host> <port>
如：例子中是尝试跟百度的一个IP进行三次握手建立连接，然后使用ctrl+C退出的时候就会触发拆连接的四次握手过程。
ubuntu@VM-16-3-ubuntu:~$ netcat 180.101.50.188 80
^C
ubuntu@VM-16-3-ubuntu:~$ 
```

测试流程是:
1. 开启抓包，比如直接开启Wireshark，或者开启tcpdump；
2. 运行netcat命令进行tcp连接的建立；
3. ctrl+c取消netcat的tcp连接；
4. 停止Wireshark抓包或者ctrl+c终止tcpdump。

抓包结果如下：
![@七禾页话][5]

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCIPabt0CnOZTSMZic7CBFkrCA3XA5NDS6Dnf3mMbicd05IxvOOxAkj1y4h8M1Bo8vGbBmbk3MrGbdg/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCIPabt0CnOZTSMZic7CBFkrgICb93gsaF2nL75JzU1OSUfSFibpDwxLbS0pqhCcj8ddJFiaAJhWD88g/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCIPabt0CnOZTSMZic7CBFkrTrtFaIeUjMiaOynN4OXbg0KIekNaFVpK33zmpdeqD2K2jm1icvSuCWKQ/640?wx_fmt=jpeg&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCIPabt0CnOZTSMZic7CBFkrBjmpmjwoL1qdjgHctYq9zHPWEUDmI5JAvksxw3SzcURRXfl7ZCDVRg/640?wx_fmt=jpeg&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCIPabt0CnOZTSMZic7CBFkrgFLexFrTheuRLoCxwb6Q9EPeRjPvdgxRPc3s7VZD3ZQroRcuPBwldw/640?wx_fmt=png&amp;from=appmsg


