---
layout: post
title: Chrome插件之一键Top
date: 2017-10-28
categories: blog
tags: [应用]
description: 回到Top更迅速！
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

在使用很多网站想回到页面顶部的时候都想为什么不加入一键Top的功能呢，这个用户体验也太不友好了。

所以这几天就想写一个此功能的扩展插件，经过几天的学习和了解，今天终于写了这个谷歌浏览器的插件——Back to Top。

主要通过调用chrome.tabs API的executeScript来执行window.scroll函数回滚到顶部坐标：

- **chrome.tabs.executeScript(integer tabId, object details, function callback)**

具体可参考**[chrome tabs API](https://developer.chrome.com/extensions/tabs#method-executeScript)**以及**[scroll函数介绍](https://developer.mozilla.org/en-US/docs/Web/API/Window/scroll)**。

插件已经发布到Google APP里，欢迎使用：

[Back To Top插件链接：https://chrome.google.com/webstore/detail/back-to-top/mhofnmfmcbeckfemdbfhfbgomilhlaeh](https://chrome.google.com/webstore/detail/back-to-top/mhofnmfmcbeckfemdbfhfbgomilhlaeh)

<img src="https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDEw9gNsp2fPs4HGY15nTgruacAd8w9ibKhhPW0zMzvHfQibQ2ibyt6YgGt3B5MYNRr0oIXNRQ1C04YQ/0?wx_fmt=png" width="70%" />

使用效果如下：
![Chrome扩展BackToTop][4]

什么？国内下载不了？

我一般的操作都是整理好一个批次的需要使用google搜索或者下载的内容清单，然后各种搜索VPN，一般VPN都会有几个小时或者几天的免费注册使用时间，趁着免费的时候就把当前时期所有需要的搞好了。【机智^_^】

------------
<p align="center">欢迎关注公众号，摄影，旅行，瞎聊，等等等：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD1g2UjYu4VfCGHmbhgVqOAnNnJQfN7ZhRVUCopYOsfpPtIEB95VNEqu8trAxJXzGDg01ka6z6wzQ/0?wx_fmt=jpeg" width="30%" />

  [0]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCZ6gG5NJjutfc6ZHJLrS03l9SOZbtcUVZpjg7KpA8mLsSEk8FZjlicsluXXorAoDAKFBIQWDBtr0g/0?wx_fmt=jpeg
  [1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAoGfjsaJt2NQ0a9AKmrIRoR9gKlX1I78Z4AoPtjyEPM56slw9gAQBdAHjHckbw4h93FvVVATBuLQ/0?wx_fmt=jpeg
  [2]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD3anvFetwgNHv3X1AiaXIzWPvazEMIEralm9vs42XsVfoniaXRCSkSpNpz9icsIYFgq84Eic2whLdAfg/0?wx_fmt=jpeg
  [3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDEw9gNsp2fPs4HGY15nTgruacAd8w9ibKhhPW0zMzvHfQibQ2ibyt6YgGt3B5MYNRr0oIXNRQ1C04YQ/0?wx_fmt=png
  [4]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eDEw9gNsp2fPs4HGY15nTgrd41Wm0f3XnLNTtHgd5yYAN13BbD17nD9C7icl1mXxfEElA4zXctSDyw/0?wx_fmt=gif








