---
layout:     post
title:      应用｜Windows11恢复经典的右键菜单：一条命令解决显示更多选项问题
date:       2024-06-30
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 应用
    - 微软
    - Windows
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

从Windows10到Windows11，有些体验很好，但是有些体验很怪，不知道是不是阿三所领导的原因。

比如本文想要解决的这个“显示更多选项”的问题，这个功能初次使用Windows11是真的不习惯，如果记不住快捷键的时候，一定需要多点一次可能看到自己需要的功能，比如复制、剪切等。

![@七禾页话][2]

那么如何丝滑的如同之前的Windows10等丝滑的展示如下图的所有可用选项呢？

![@七禾页话][3]

答案就是修改注册表。

需要以管理员身份运行CMD或者Powershell后，输入如下的命令添加一个注册表信息即可：
```bash
左右滑动

reg add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve

```

![@七禾页话][4]

![@七禾页话][5]

运行完之后需要重启系统才能生效，如果不想重启电脑，可以继续运行如下的命令重启explore（也可以通过任务管理器重启）：
```bash
左右滑动

taskkill /f /im explorer.exe & start explorer.exe

```

![@七禾页话][6]

![@七禾页话][8]

然后在右键点击文件或者文件夹等就可以看到和Windows10一样的效果了：

![@七禾页话][3]

如果想要回退成Windows11默认的效果，就需要在cmd或者PowerShell中删除添加的注册表信息，cmd或者PowerShell依然需要以管理员方式运行：
```bash
左右滑动

reg.exe delete "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /va /f

```

运行后需要重启电脑或者运行如下命令重启explore即可（也可以通过任务管理器重启）：
```bash
左右滑动

taskkill /f /im explorer.exe & start explorer.exe

```

![@七禾页话][8]

以上，就是Windows11系统如何在“显示更多选项”和显示所有选项中的切换。

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAOmJjI73ltnv2EbWVNAzwo7rhkiaQFYs9HLbicEic6haQKjo1XgXCXVsE8OwWDAlI2IIwayYQCibT8wA/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAOmJjI73ltnv2EbWVNAzwoLmMymiaLicqFia68x1eerPuxSRibLHFibBNmHF4H334yUiaXTon2Qib6FzHUg/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAOmJjI73ltnv2EbWVNAzwojBic3ibS1VcvoprBDhcibspj2eicjy3Sh0lmj2Havq3uJPP8mrUciceib1XQ/640?wx_fmt=png&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAOmJjI73ltnv2EbWVNAzwoAhZIqQ5UTcxvexb3TabgK9pWIyMl2CicIFEvzuKibg2R8ssiadtBrohew/640?wx_fmt=png&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAOmJjI73ltnv2EbWVNAzwoBJzib4fViaDaPFpP6ncKl2x4tV8UruR69TVNJ9GLxYb5D36ZQibGcJL7g/640?wx_fmt=png&amp;from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAOmJjI73ltnv2EbWVNAzwoZL0Gzrc9jQI5wuzRcfRVnqgyN6fibgh3YfjQR4xLcQq5MV2ib1PNc3Kg/640?wx_fmt=png&amp;from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAOmJjI73ltnv2EbWVNAzwo46ibnUnjg3XkYkF4frVvicPkompgurpoTFKicNmNLJYnf0zTFpNNCbr0Q/640?wx_fmt=png&amp;from=appmsg


[8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAOmJjI73ltnv2EbWVNAzwosHlYSZdukoGOBibQN9P3gT1udxwOQ5c2UFFriaVAtRNibkHoIT2pZkHUg/640?wx_fmt=png&amp;from=appmsg

