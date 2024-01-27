---
layout:     post
title:      Linux｜容易迷糊的时间戳事件
date:       2024-01-26
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

分享一个之前没有注意到的知识点——timestamp时间戳。

起因是在排错的时候，同事说log的时间不对，通过解析时间戳怎么是中国的时间巴拉巴拉的，理论上应该是设备所在的当地时间。

然后通过一些工具的解析，终于知道了为什么同事解析的时间戳是中国时间了。

我们先来看看时间戳到底是个什么东西。

时间戳是自一个特定时刻（称为“epoch”）起经过的时间量的表示。它在计算机科学中广泛用于记录事件发生的时间点，常用于各大日志、数据包等等。最常见的epoch就是Unix epoch，即1970年1月1日00:00:00 UTC。

### 为什么Unix epoch是1970年呢？

选择1970年元旦的零时作为Unix epoch有历史原因，也有随机性，随便看了看后总结了如下几点：

1. Unix操作系统首次发布是在1969年，然后需要一个简单而实用的方法来表示时间，因此大佬们休假回来一讨论，就把非常接近Unix 系统开发时间并且有初始化意义的1970年1月1日作为Unix时间戳的起始时间了。

2. 因为是休完圣诞元旦假期回来的，1970年1月1日已经作为过去时了，作为一个过去式的接近操作系统开发时间的日期可以避免在表示当时及之后的日期时出现负数。

3. 当时Unix时间戳还是用32位整数来存储的，这意味着它可以表示的最大值是 2^31-1 秒，这样从1970年往前往后算，可以覆盖1901年到2038奶奶的时间，当时来看基本够用了（32系统需要注意2038年问题）。

4. 1970年1月1日就是一个普通的新年日，没有与现有历法或重要历史事件相关联，这使得它作为一个“中性”的起点非常合适，避免了不同文化上的认同问题。

最后使用 UTC 作为标准，是因为UTC作为“协调世界时”（Coordinated Universal Time），是目前国际上最广泛采用的时间标准。它是一种基于原子时钟的时间尺度，与格林威治平均时（GMT）非常接近，但在技术上更为准确。

总体来说就是UNIX大概这个时间点发布的，过完年就拍脑门子定了。

### 时间戳的精确度如何区分呢？

聊到时间戳，就得聊聊其精确度，平常我们看时间就是时分秒，但是在计算机或者更加高精尖的技术需求中，比如航天，秒已经不是最小单位了，需要更精确的毫秒甚至纳秒的精度。

时间戳可以精确到下边四种不同的级别：

- **秒**：最基本的Unix时间戳是以秒为单位的，表示自Unix epoch以来的秒数，比如`1970年1月1日00:00:01 UTC`距离Unix epoch就差1秒，那么`1970年1月1日00:00:01 UTC`的时间戳就是1。
- **毫秒**：毫秒级时间戳是秒级时间戳的千分之一。
- **微秒**：微秒级进一步细分为秒的百万分之一。
- **纳秒**：纳秒级时间戳提供最高精度，为秒的十亿分之一。

![@七禾页话][2]

区分秒级、毫秒级、微秒级和纳秒级时间戳主要依赖于它们的长度（位数）和数值范围：

1. 秒级时间戳（Second-level Timestamp）的长度通常为10位数字。例如，`1617181723`。
2. 毫秒级时间戳（Millisecond-level Timestamp）通常为13位数字。例如，`1617181723000`。
3. 微秒级时间戳（Microsecond-level Timestamp）通常为16位数字。例如，`1617181723000000`。
4. 纳秒级时间戳（Nanosecond-level Timestamp）可能超过16位，通常在19位左右，具体取决于时间的具体值。例如，`1617181723000000000`。

### ISO-8601标准时间格式

可以看到时间戳都是一串数字，对于人来说非常不好读的，因此需要有一个标准，将时间戳转换成可读的统一时间标准，其中之一就是ISO-8601标准。

ISO-8601是一种国际标准化的日期和时间表示方法。这种格式旨在提供一种清晰、一致的方法来表示时间，易于人类阅读和机器解析。

ISO-8601格式由如下几部分组成：
- 日期部分：按照“YYYY-MM-DD”格式。
- 时间部分：按照“HH:MM:SS”格式。
- 分隔符：日期和时间之间使用`T`。
- 时区：UTC时间用`Z`表示，"Z" 是指 "Zulu time"，这是军事和航空领域中用于指代 UTC 的术语，在 ISO-8601 中，这个 "Z" 代表零时区；其他时区用与UTC的时差表示，如`+HH:MM`或`-HH:MM`。

比如`2024-01-23T13:00:00+00:00Z`表示UTC标准(约等于0时区)的2014年1月23日下午一点，对应我们中国的时间就是晚上9点；而`2024-01-23T13:00:00+08:00`表示东八区的2024年1月23下午1点。

![@七禾页话][3]

在 ISO-8601 标准中，要特别注意分隔符 `T`，它是标准的一部分。这个 `T` 字符是必须的，用于明确区分日期和时间。例如，`2024-03-05T01:30:00` 中的 `T` 就是将日期（`2024-03-05`）和时间（`01:30:00`）明确分开。

此外，`T` 是唯一用于此目的的字符。ISO-8601 标准没有提供其他字符作为日期和时间的分隔符。这种严格的格式规定是为了确保全球范围内的一致性和无歧义性，特别是在跨国界、多语言环境的数据交换中。

然而，在某些非正式的上下文或者为了可读性，在不严格遵循 ISO-8601 标准的情况下，人们可能会省略 `T` 或使用空格代替。但是，在需要严格符合 ISO-8601 标准的场合（例如，编程、数据存储、国际通信等），正确使用 `T` 是必须的。

### 有意思的2038年问题

2038年问题是由32位系统中时间戳表示方法引起的。32位系统中，时间戳以32位有符号(正负号)整数存储，也就是1970年1月1日0点之后的用正数，1970年1月1日0点之前的用负数，因此能表示的最大值为2147483647(2^32-1)，最小值是-2147483647(2^32-1)。当系统时间达到2038年1月19日03:14:07 UTC后，时间戳将溢出，导致时间表示回跳到1901年。

当然现在的计算机大部分都开始向64位过度，已经不存在2038年问题了，但是如果接触到32位系统的设备，一定要注意了。

### 时间戳的转化

一长串时间戳实在看不出来是何年何月何日几时几分几秒，所以我们需要一个工具来将不是人看的时间戳转换成人看的标准时间，但是因为精确度的问题会导致转换出现错误，这个时候最好多用几个工具来对比，或者直接取前十位按照秒级的时间来转换，在日常运维查看log对时间秒级就够用了。

这里我大概找了三个网站，基本可以正确将毫秒级别的时间戳正确转换到秒或者微秒。

使用时间戳1701226329450619（16位微秒级）为例来解析：

- [https://www.unixtimestamp.com/zh/](https://www.unixtimestamp.com/zh/) 可以识别时间戳到纳秒级别，但是不管是什么精确度的时间戳，最终都解析到秒，然后显示GMT(可以认为0时区)和电脑系统时区的两个可读时间：

![@七禾页话][4]

- [https://www.epochconverter.com/](https://www.unixtimestamp.com/zh/) 跟unixtimestamp一样，可以识别到纳秒级别的时间戳，优势是如果是毫秒、微秒、纳秒的时间戳可以在最终转换的GMT和电脑本地时间中追加毫秒数，精度更好一些：

![@七禾页话][5]

- [https://www.epochconverter.io/](https://www.unixtimestamp.com/zh/) 这个网站只能识别到微秒的精确度，纳秒的时间戳会计算错误，但是对于毫秒和微秒的时间戳也可以转换出毫秒数，另外这个网站在GMT和电脑本地时区基础上，可以再选择一个时区，对于我们跨时区项目就非常友好了：

![@七禾页话][6]

这个是我目前找到的几个时间戳转换的网站，公号里不能直接跳转外链，可以后台回复`ts`获取网站超链接；如果你有更好的工具欢迎留言分享或者后台私信。

需要注意的是时间戳可以转换成任何时区的时间，这个就是最开始为什么同事说log里的时间戳是中国的时间，因为大部分网站转换的时候都自动转换成电脑所在时区的时间，如果有搭配GMT时间，可以看到0时区的时间，然后再换算项目设备所在时区的时间，就知道这个log记录的事件是在当地什么时候发生的了。

所以我说上边的第三个网站非常好，因为他可以自由选择一个时区，这样一个时间戳就转换成三个时区的时间了：
- GMT的零时区
- 电脑配置的时区
- 手动选择的时区

但是不管用的什么网址转换，一定要记得时间戳不对应任何时区，它可以转换成任何时区的可读时间，所以转换后的时间一定要搭配时区一起看，然后在脑补转换成其他时区的时间，换算工具可以参考[应用｜外企工作？被时区换算伤到了](https://mp.weixin.qq.com/s/CB1vD1-mxWsBMFflkimB9w)。

在写ppt或者文档的时候，尤其是技术相关的文档，标准来说时间最好要搭配时区，清晰明了防止歧义。

最后贴一张Wireshark视图中设置时间显示格式的配置，可以看到Wireshark对时间戳的解析还是非常强大的，可以转换各种时间，并且可以精确到纳秒，对于分析包看前后顺序是非常有帮助的：

![@七禾页话][7]

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBd1g0OicLb1TwwwficUsaX2yw8OE4qBgpaxP0c4KOSzibicFlXlicLlUptfCVCI0vc48DDBjPuXwI4ZCA/640?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAqEbibyT3NwPUYzUdAOZFTKrNNj09wsjcpYictiakw87ORba7QlkdPluwZwj1cwImPQgiah0RY7OKZsg/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAqEbibyT3NwPUYzUdAOZFTK5jLCwxFkJdu8GOX3LWuS1xDxJoCIUl1Yehe8ZMWxjP421AMoGnstWw/640?wx_fmt=png&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAqEbibyT3NwPUYzUdAOZFTKNNvkZcKr6QicmCmjRpy7kBCbCslOU7wNPHF2xToPDOTahfxQxZwtSpA/640?wx_fmt=png&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAqEbibyT3NwPUYzUdAOZFTK5ib25UnzlbEG3aAMeJ2jGlJHlgvIVy8oSOoY6MiahxrNYRsHwQicPfeTA/640?wx_fmt=png&amp;from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDdP0EDv9IlzfE97NAMJibnbutSPrAnCKRmyxvzQYibt4gaoib5kW54J38c0pdu9icFwBRxqd930udCLw/640?wx_fmt=png&amp;from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAqEbibyT3NwPUYzUdAOZFTKpq93pQMCvfloWNldkRpJibCoQMhPRbicQx3NmkjHLWl3YXKjWd9uulOw/640?wx_fmt=png&amp;from=appmsg
