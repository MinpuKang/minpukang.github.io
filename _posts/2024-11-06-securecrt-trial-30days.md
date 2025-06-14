---
layout:     post
title:      软件库｜SecureCRT无限续期30天失效了怎么办？附CRT激活工具，MAC和Win都有
date:       2024-11-06
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 软件
    - SecureCRT
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

![@七禾页话，拍摄于十月的长白山][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

本来每天删掉SecureCRT的license注册表就可以很好地重置倒计时，结果最近一个月突然不行了，每天依然会删掉其注册表，但是倒计时确不再重置。

后台有朋友留言也说遇到了这个问题。

就在前天30天倒计时彻底结束，然后就不能白嫖SecureCRT的Trial了。

那么怎么办呢？

正所谓山穷水尽疑无路，柳暗花明好白嫖。

方法也很简单，只需要彻底卸载SecureCRT再安装就可以继续使用Trial了。

首先需要备份你的SecureCRT的Config文件夹，防止卸载的时候丢失。

然后卸载掉SecureCRT。

之后使用管理员权限打开注册表：

![@七禾页话][2]

删除掉Computer\HKEY_CURRENT_USER\SOFTWARE和Computer\HKEY_LOCAL_MACHINE\SOFTWARE这两个路径下的VanDyke(针对只安装了SecureCRT)：

![@七禾页话][3]

![@七禾页话][4]

最后再安装上SecureCRT（官网直接下载即可）就可以重启Trial倒计时。

这样重装后也可以每天删掉注册表里的注册信息来重置倒计时，具体可参考[应用｜SecureCRT无限续期30天！附CRT激活工具！](https://mp.weixin.qq.com/s/4l2l_Hv6oz64ntuP24demg)

等到下一次倒计时不能重置，我想依然可以往复上述的步骤白嫖到永远吧！

最后就是如果不想使用试用版，而且也没有公司规定需要使用正版要求，可以公众号后台回复 **crt** 获取安装包和激活工具。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBcriawLODBQLUpOiaOrk3icnSQicYEvFBqesGqUEt6HWWrCUP3KCEfYucsVe1Rpst0lOAQtL7yoaq7Og/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBcriawLODBQLUpOiaOrk3icnSFicBfTcJrVAeQfXu9sWnAMn3icmpqGYnbpBJ3LmN3GlSBOGLBKp4WDnQ/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBcriawLODBQLUpOiaOrk3icnS5b3g9Es0qCNQd7M6ddT0PotbxxyDhgTDhaMywic6h2kNJzTR0qSPvBw/640?wx_fmt=png&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBcriawLODBQLUpOiaOrk3icnS7LGuE0qwPxW1UCum9ouicrMZ2OBcgSicRkxgxMAxGfawwJ8ibzf7PUZbQ/640?wx_fmt=png&amp;from=appmsg

