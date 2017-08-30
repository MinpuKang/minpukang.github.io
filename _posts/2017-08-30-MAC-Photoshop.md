---
layout: post
title: MAC下破解安装Photoshop CC 2015
date: 2017-08-30
categories: blog
tags: [MAC, Photoshop]
description: 后期也是摄影的一部分！
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

一场秋雨一场寒，自从周日到周一下了一场绵绵秋雨之后的这两天突然就如同进入了深秋，可是我明白这是一场阴谋，因为在开学季来临如果不热的话怎么对的起这即将开始的军训，哈哈^_^……

这两天不知道为什么破解的Photoshop突然出现了试用期提示，在30天试用期之后终于迎来了不能用的局面，没办法，再此破解一下吧，所以为了做一个记录，顺便也许能帮助一些跟我一样因为正版太贵而不得不走破解之路的人一个参考。


##准备阶段
1.两个dmg的安装包是必须的，一个是Photoshop安装包（Photoshop_16_LS20.dmg），一个是破解工具X-Force（Adobe CC 2015 X-Force macOS Sierra 10.12..dmg）。

2.然后需要先联网打开这篇教程，再断网，包括有线网络和无线WiFi网络全部关闭。

3.最后需要确保/etc/hosts下没有任何adobe的解析：

    MacPro$ more /etc/hosts
    ##
    # Host Database
    #
    # localhost is used to configure the loopback interface
    # when the system is booting.  Do not change this entry.
    ##
    127.0.0.1   localhost
    255.255.255.255 broadcasthost
    ::1             localhost

##可能碰到的问题

1.在安装Photoshop填写了序列号之后依然出现如下图的问题：
![提示登录][3]

解决办法就是查看准备阶段的/etc/hosts，保证没有任何的adobe内容解析。

2.在安装完毕激活的时候一直出现如下图所示内容，没有任何脱机激活提示：
![提示登录][4]

解决办法就是检查/Library/Application Support/Adobe/下是否存在SLCache、SLStore、SLStore_v1等文件，如果存在，删除之后再打开Photoshop尝试激活；如果依然不行，删除文件后，重新安装Photoshop：
![删除Abode Crash文件][5]


##教程开始
所谓万事开头难，准备工作做好也就完成了50%以上的工作，接下来就是要一步一步的点点点了：

整个过程大概有如下几步：

    1. 再次检查准备工作是否完备；
    2. 打开X-Force，生成一组PS序列号：需要注意的是X-Force要保证全程开启状态，直到脱机激活PS为止；
    3. 安装Photoshop；
    4. 脱机激活Photoshop；
    5. 修改/etc/hosts文件；
    6. 联网更新ACR（Adobe Camera Raw）。

在检查准备工作完成之后并多次确保**断网**之后就可以开始破解流程啦。


####1、打开X-Force，并生成PS序列号
打开X-Force，Product包含了几乎Adobe所有的产品（所以本教材也适用于Adobe 2015的其他系列，如果需要可以微信公众号下留言或者后台私信是否有相应的软件^_^……小福利）。
我们现在需要选择的产品是**“Adobe Photoshop CC”**，然后点击**“Gen. serial”**生成序列号既Serial，如下：
![X-Force1][6]

**Note：保持X-Force为打开状态一直到脱机激活完毕！**

####2、安装Photoshop
上一步已经拿到了破解的钥匙，现在我们需要慢慢的打开破解的大门，一步一步的开启Photoshop这座堡垒。

现在需要打开Photoshop的安装包，点击 _**Adobe Photoshop CC 2105**_ 之后选择 _**Install**_ ，此时面对了一道选择题，是“安装”还是“试用”？因为我们是要破解，试用算个什么鬼，所以此处果断选择 _**安装**_ ，如下图所示：
![安装选择1][7]

点击“安装”之后出现下图左边的登录提示，然后点击 **“登录”** 尝试联网失败后出现右图提示后选择 **“稍后连接”**：
![安装选择2][8]

之后出现了软件许可协议，通读文章并总结中心思想之后当然是选择相信他呀——**“接受”**，进入到右图填写序列号的窗口：
![安装选择3][9]

此时将步骤一中X-Force生成的序列号复制并粘贴到Photoshop对话框中（小提示：选中X-Force中的序列号右键选择copy，然后到photoshop对话框第一个输入框右键选择粘贴）：
![安装选择4][10]

填充完序列号，点击下一步再次出现了联网失败的记录，不要被他蛊惑，依然保持**断网**状态选择 **“稍后连接”** 进入到安装的后期阶段，选择安装语言和位置之后点击 **“安装”**：
![安装选择5][11]

由于Mac系统的安全性和某些配置，此时需要对用户进入验证，输入当前用户的密码并点击**“好”**确认后就进入到自动安装阶段，一直到安装完毕出现右图提示：
![安装选择6][12]

到此安装阶段结束，点击 **”立即启动”** 开始脱机激活，保持X-Force为打开状态且没有任何修改，同时保持断网！


####3、脱机激活Photoshop
紧接上文，当点击立即启动之后，Photoshop进入到初始化阶段并要求登录，由于网络不通会出现**“连接Internet时是否出现问题？”**，当然是有问题的了，此时切记要点击 **“连接Internet时是否出现问题？”**从而进入到右图**“脱机激活”**状态：
![脱机激活1][13]

点击**“脱机激活”**，进入到请求代码生成页面，并点击**“生成请求代码”**生成请求代码如右图所示：
![脱机激活2][14]

选中生成的请求代码右键选择复制，并将其粘贴到X-Force的第二个对话框**"Request code"**中：
![脱机激活3][15]

此时点击X-Force中的**“Gen.activation”**按钮来生成其第三个对话框中的**“Activation code”**：
![脱机激活4][16]

如同复制Serial一样，选中Activation code后右键选择Copy，将其粘贴到PS脱机激活对话框中的**“响应代码”**中，操作如下：
![脱机激活5][17]

确保在安装和激活过程中X-Force中的序列号不变，同时X-Force使用请求代码生成的响应代码已经完整填充到了脱机激活对话框之后点击神圣的按钮**“激活”**，从而完成了整个脱机激活过程：：
![脱机激活6][18]

![脱机激活6][19]

点击启动按钮来体验Photoshop的神奇之处吧，此时可以通过 **“帮助”** 菜单看到 **“更新”** 按钮说明激活成功：
![脱机激活7][20]

到此可以关闭X-Force，但是依然要保持**“断网”**状态。

####4、修改/etc/hosts文件
关闭Photoshop，依然保持断网状态，我们知道此时一旦连网，如果再次使用photoshop，那么必然要导致其进行官网同步，那么破解大业就烟消云散了，如何解决？那么就要阻止其在联网状态下去跟官网同步，我们的做法是“瞎指挥”。

什么意思呢？我们知道软件跟官网同步都是通过其内置的url进行DNS解析拿到IP地址后发送某些数据到官网，那么我们就可以通过篡改其域名解析从而达到瞎指挥的目的。

然而如果自己搭建一个DNS server费时费力还要解决其他使用的问题，怎么办，在DNS解析过程其实软件第一步是要到一个叫做hosts的文件中查询地址的，那么我们就可以将adobe的一系列url加入到hosts文件中并将其指挥到一个未知的世界从而完美解决官网同步的问题。

在MAC下hosts文件在/etc下，如果熟悉终端的朋友可以直接通过命令**sudo vi /etc/hosts**在终端下添加，但是为了方便更多的朋友，我们这里依然介绍图形界面下的修改方式：

a、在Finder下的**“前往”**中选择**“前往文件夹”**，然后填写**“/etc”**，点击**前往**按钮进入到**etc**文件夹：
![后续问题1][21]

b、由于权限问题，先将hosts文件复制到其他地方比如桌面并打开，追加如下内容（注意是追加，也就是从结尾处添加，以下URL基本不会有落网之鱼）：

    127.0.0.1   lmlicenses.wip4.adobe.com
    127.0.0.1   lm.licenses.adobe.com
    127.0.0.1   na1r.services.adobe.com
    127.0.0.1   hlrcv.stage.adobe.com
    127.0.0.1   practivate.adobe.com
    127.0.0.1   activate.adobe.com
    127.0.0.1   ereg.adobe.com
    127.0.0.1   wip3.adobe.com
    127.0.0.1   activate.wip3.adobe.com
    127.0.0.1   3dns-3.adobe.com
    127.0.0.1   3dns-2.adobe.com
    127.0.0.1   adobe-dns.adobe.com
    127.0.0.1   adobe-dns-2.adobe.com
    127.0.0.1   adobe-dns-3.adobe.com
    127.0.0.1   ereg.wip3.adobe.com
    127.0.0.1   activate-sea.adobe.com
    127.0.0.1   wwis-dubc1-vip60.adobe.com
    127.0.0.1   activate-sjc0.adobe.com
    127.0.0.1   hl2rcv.adobe.com

保存关闭，并将修改后的hosts文件复制粘贴回**“/etc”**文件夹，并选择替换老文件，操作如下：
![后续问题2][22]

更新完成后为了万无一失打开**/etc/hosts**文件再次确认已经添加Adobe的系列URL解析：
![后续问题3][23]

到目前为止Photoshop破解已经完成，并且可以联网进行PS的各种操作，但是需要注意的是最好不要进行登录，即使你已经注册了Adobe账号。

然后，为了更好的处理RAW文件，PS CC 2015捆绑的是ACR8.0版本，所以需要对ACR进行在线升级，但是最好不要再对Photoshop升级，保持CC 2015状态，因为一旦升级Photoshop可能会面临被识破的局面，如果升级也不要升级到最新的2017，切记切记。

####5、联网更新ACR
破解完成并且更新了**/etc/hosts**文件后，就可以联网使用了，此时可以更新一下ACR软件，打开Photoshop之后，在帮助菜单栏找到更新按钮进行可更新软件检测，如对Photoshop Camera Raw 8进行升级，选中后点击更新按钮后输入密码，点击**“好”**坐等更新完毕：
![后续问题4][24]

此时因为Photoshop是打开状态，会提示关闭Photoshop CC，在Dock栏中强制关闭PS CC即可。
![后续问题5][25]

到此整个Photoshop CC 2015破解完成，并且将ACR更新到了最新版本。

以上就是本教程的全部内容，也欢迎点击下方二维码关注公众号——七禾页话，了解更多内容。

破解软件下载链接：[MAC-Photoshop，密码:nz9r，链接：http://pan.baidu.com/s/1cLawhC ](http://pan.baidu.com/s/1cLawhC )

------------
<p align="center">欢迎关注公众号：</p>
![公众号][1]

<p align="center">感觉内容不错，读后有收获？欢迎小额赞助：</p>
![赞赏][2]

  [1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eADRGLg8pP0UTYFgxf7ic1ADxuF7ibzZVd1kb3zEeZpvrZQap8waaAzYialn7pFl3HMs7RwUcibPicWz2A/0?wx_fmt=jpeg
  [2]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD3anvFetwgNHv3X1AiaXIzWPvazEMIEralm9vs42XsVfoniaXRCSkSpNpz9icsIYFgq84Eic2whLdAfg/0?wx_fmt=jpeg
  [3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhkQ62CC4l5ibXKUM87O4CphpfArIAEDmqAuxbymDDXs1Mzf9OBvLtZMA/0?wx_fmt=png
  [4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhdGcB3u9nCgWff8osyFrWsZic2sNhnmGoW5P5OPOmVXfpyZlGIyIXZGw/0?wx_fmt=png
  [5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhQaLsxiaQ1wrqiahWia1libspumVNLCs85nzgLibT5fgljckLZPac7HB6l6w/0?wx_fmt=png
  [6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhNGoOJ9XomFB7jqIKujjlLibdhP3AcLuPCiaqEcia2uaiaPg9JycUtlnTAw/0?wx_fmt=png
  [7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhTGz71BibcIPN8jWZiaU21GeCibm78aicDSia3XiaNcI1rKyVrv2icqG3Z7Hmg/0?wx_fmt=png
  [8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhWsjSaHoPLnc8fdEJDwAYR0ue7ib6sZDeCl0rLR6Lbj6zlJejy7Lcumw/0?wx_fmt=png
  [9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhNhcvdcrVKxLQibjNgjuNXAQ485n0BbuxCFIvkK9ZW3ZevWN0GMHmLiaQ/0?wx_fmt=png
  [10]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhFIHbr6GmzziaAic97NZibZiaxfGNPtogiasljCW9Mp78TsgcLM7fgaP6Y1A/0?wx_fmt=png
  [11]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhVbSDWPCwLNic9vZYCjVLalabhK6wemMM5wzyleaIEOJb3NsLtFvjCcQ/0?wx_fmt=png
  [12]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhWVVvBxsMcE5jmRZmUn1IfWUpQ1GffxoUaaa6TQjOAQANdpSrA3XX1w/0?wx_fmt=png
  [13]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLh6YqsY7I7VdDSkvicQrjrSvuq7AvWUYYA2cYRyKm1kB8aCdoHbhHIiaTg/0?wx_fmt=png
  [14]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhv3ANZic741OpgPbs3qAEhAYNyKOlQduzkQo0mw3KLjvWSIcOHYGRW4A/0?wx_fmt=png
  [15]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhDzXXUKrhojibZKQQts2uCM5vwrqmq5HVHLOuAusG8qWx0W8IUUiaU6bg/0?wx_fmt=png
  [16]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLh5JlmpcRQYKVF4NX1Utf75ZL9KnLBKMewaZy0aWlBYHicASfDMCUINPw/0?wx_fmt=png
  [17]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLh63EMkVAJEib6WTDVDQaUw1cn6KI4gflfa5ic1NhzXA3AJib1FlU9FecdA/0?wx_fmt=png
  [18]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLh12XxBdy96yFW94Ddy8vceHMe01mMLQ69oDaVZaZvpka8AYLGmvCHNg/0?wx_fmt=png
  [19]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhg8oRCdnmJer6g4DgkaxibfDd10RibUAMWRZCygibxKkyDulcFma3liaw2w/0?wx_fmt=png
  [20]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhUAfgbsusNga9m7MLAiaZjQjYRpAYDaHfFYmeB89YyPtvibQJpgcj835A/0?wx_fmt=png
  [21]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhC3GCLA252s7QofKibibCMVIqa1ETJ2VuhrJFvSyHzCK2ic2OUFXjq8dkQ/0?wx_fmt=png
  [22]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhXs0Zib051IhBWFHiaFia9oIgEULI2m8ofcQTSn13Eu9BzuqEVlwsrrxibg/0?wx_fmt=png
  [23]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLh0ZB5BoH5Y0GvWsuRPKtica6QPMvBQV8g2IPKnWhKWe6OuoUicxqhEZaA/0?wx_fmt=png
  [24]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhvWJeydRIHDLTkXnoXAicngI19WJZ1kpUdNTeprgTDdtV7dfmpD6tRCg/0?wx_fmt=png
  [25]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDxJVLIQErsDA2WubXblOLhkTiaBIJc9JT0rIYu11eK7TolyvqPQKOgZw1L5yibJ6pFJ84MV0W5ZFqg/0?wx_fmt=png

