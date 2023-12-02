---
layout:     post
title:      通信｜DNS域名中的点儿和通配符
date:       2023-12-01
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 通信
    - DNS
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

DNS中有两个经常使用但是容易被忽视的小东西：
- 一个是尾随点（Trailing dot）
- 一个是通配符（Wildcard）

### DNS 尾随点（Trailing Dot）

尾随点是DNS定义必要的一个知识点，它代表了一个域名的绝对路径。在 DNS 层次结构中，根域（root domain）被表示为一个空字符串。因此，一个完全合格的域名（FQDN）如 `www.example.com` 在技术上应该写作 `www.example.com.`，其中末尾的点代表了根域，每个点儿前边的部分都是域名的一个级别，比如下图中的域名`www.hk314.top.`，根域为空，其他以此降级:
![@七禾页话][2]


- **RFC 1034** 提供了关于 DNS 的基础概念，其中解释了尾随点的作用。它指出，尾随点用于区分相对和绝对域名。在 DNS 查询中，没有尾随点的域名可能会依赖于本地配置（如搜索后缀）进行补充。
- **RFC 1535**讨论了 DNS 解析器处理不完全合格域名的方式，强调了尾随点在避免潜在安全风险中的重要性。

举个例子，如下的一段配置中，当我们使用dig解析域名`test.hk314.top`获取的地址是1.1.1.1，如果想获得2.2.2.2这个IP，那么需要解析的是`test.hk314.top.hk314.top`：
```bash
##左右滑动
;$ORIGIN    hk314.top.

test.hk314.top.    IN A   1.1.1.1
test.hk314.top     IN A   2.2.2.2
```

所以在定义DNS解析的时候，需要特别注意trailing dot的使用，否则会引起错误的解析，从而需要花费时间去排错。

另外需要注意的是，我们在浏览器中输入域名的时候通常不用带trailing dot，主要是因为我们大多数互联网用户不熟悉 DNS 的内部工作机制，包括尾随点的概念，所以为了使网络更加用户友好，浏览器和其他客户端软件被设计成自动处理这些技术细节。

非常有意思的是当我尝试`baidu.com`，网站打开没有问题，当我尝试带末尾的点儿`baidu.com.`，提示`405 not allowed`；但是`google.com`和`google.com.`都是可以正常打开谷歌主页的。

嗯，个中缘由咱们自己瞎猜吧。

### DNS 的通配符（Wildcard）

DNS通配符是一种特殊的域名部分，通常用星号（`*`）表示，用于匹配一组子域名。例如，一个通配符DNS记录 `*.example.com` 可以匹配 `any.example.com`、`subdomain.example.com` 等任何以 `example.com` 结尾的子域名。

- **RFC 1034**提供了 DNS 的基础概念，其中包括了通配符的初始定义。它指出通配符可以用于匹配多个域名，但有特定的使用规则。
- **RFC 4592**更深入地探讨了通配符在 DNS 中的作用，特别是在复杂的域名结构中通配符的行为和限制。

但是通配符使用的时候有一些限制：
1. **位置限制**：通配符必须完整地出现在域名的最左侧，且不能被部分使用。例如，`*.example.com` 是有效的，而 `sub*.example.com` 或 `*sub.example.com` 则无效。
2. **级别限制**：一个通配符只能匹配一个域名级别。例如，`*.example.com` 不会匹配 `sub.subdomain.example.com`。

因此我们可以理解下通配符的使用，是只能在FQDN最左侧第一个子域名中使用，也就是域名第一个点儿之前可以用通配符。

另外需要注意的是如果为特定子域设置了明确的 DNS 记录，则该记录优先于通配符，比如如下的两条DNS定义，当解析test.hk314.top的时候得到的IP必然是2.2.2.2：
```bash 
###左右移动

*.hk314.top.        IN A   1.1.1.1
test.hk314.top.     IN A   2.2.2.2
```

以上两个小概念，trailing dot是域名FQDN定义必须要了解的概念，DNS配置中必须要考虑的知识点；通配符作为域名FQDN中一个特殊的子域，需要了解和掌握使用规则，从而简化DNS中的配置。

​最后[通信｜DNS配置生成工具大进化！](https://mp.weixin.qq.com/s/USgpqysaXcUReoU9SPVy9Q)中的工具可以直接关注本公众号`七禾页话(qiheyehk)`回复`dns`获取下载链接。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD0ibol9T5a6wFeicfnnmh61PBPeUibqcURzJ5ST6aeYjBHh1sctrQw0Lbbfsd4iaB3qu6WLsgtTa8jXA/640?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBx5A8pLy6Cdb7IpiadOYogSG8V7VkNZ3z1AJIS3deoxwl3aUTFzZenoic3ia8aYdwlTR9vPJqYveicag/640?wx_fmt=png&amp;from=appmsg
