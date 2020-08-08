---
layout:     post
title:      应用｜1+1构造地表最强流程图工具
date:       2020-08-08
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 应用
    - 流程图
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

大家知道我是一个“24K资深”通信工程师，所以日常画画拓扑图、信令流程图、产品逻辑图等等都是必备技能，但是苦无Visio久矣，所以每次画图都一个脑袋两个大。

每次不是在Word里就是PPT中一个一个的插入编辑，最终形成庞大复杂的流程图。

但是毕竟业余的工具不能搭配我专业的水准，在多次忍辱负重之后，终于发现了一款免费好用不费资源的地表最强流程图软件。

其实说它是软件，并不准确，更准确的说法是它是站在巨人肩膀上的一盏明灯，这个巨人就是现在非常流行的一款编辑器Visual Studio Codo(VS Code)，而最强流程图工具就是VS Code的一款插件draw.io。
![1][1]

香不香来看看演示效果：
![2][2]

#### 如何安装？
安装异常简单，直接在VS Code插件库中搜索关键字“draw.io”后选择第一款“Draw.io Integration”：
![3][3]

如同他的介绍：一款将Draw.io集成到VS Code的民间扩展，但是高手都在民间，所以这款插件可以匹敌很多正式发布的产品，毕竟人民的力量是无穷的。

### 如何使用？
使用也异常简单，如同演示效果中一样，只需要创建带有下列任何一个扩展名的文件后，使用VS Code打开就会加载draw.io画板。

- .drawio
- .dio
- .drawio.svg
- .drawio.png

然后就可以肆无忌惮的画需要的任何流程图、信令图、拓扑图了。
![4][4]

#### 其他功能
这款插件有很多功能，第一个其实也是默认的功能，可以通过.drawio.svg和.drawio.png格式直接编辑生成svg、png文件。

*.drawio.svg可以用于直接嵌入到Github的README中，而不需要额外的导出.

*.drawio.png不需要额外导出直接存为png文件，但是开发者建议最好用.svg这种可以任意缩放的矢量图文件，这款插件的logo就是使用此插件创建的一个.drawio.png图片。
![5][5]

插件默认只能处理*.drawio.svg格式的文件，但是通过在VS Code的settings.json中关联.svg格式，就可以打开.svg的文件了，配置如下：
````
"workbench.editorAssociations": [
    {
        "viewType": "hediet.vscode-drawio-text",
        "filenamePattern": "*.svg"
    }
]
````
需要注意的是这样不能编辑任何的svg文件，只有通过Draw.io创建的才可以被编辑。

第二个就是Code Link Feature，就是可以通过标签#和Class的名字，直接进入源码，咱也不懂，直接上演示：
![6][6]

另一个强大的功能就是可以通过File: Reopen With...命令使用xml打开，这样可以实时的在draw.io和xml直接同步更新：
![7][7]

#### 配置
插件支持自定义配置，比如修改主题、自动存储、默认字体大小等等：
![8][8]

Github上有其源码，所以大家也可以贡献自己的星星之火填充更多好用的功能。

希望这样的一款插件可以给疲于寻找流程图工具的大家一个灯塔。

不过话说VS Code是微软出品，不知道哪天会不会如同matlab一样，被封印了。

相关链接：

[Github of Draw.io Integration](https://github.com/hediet/vscode-drawio)

[Visual Studio Code](https://code.visualstudio.com/)

[Draw.io Integration插件](https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio)

------------
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)，旅行、摄影。。。偶尔分享技术周边</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[1]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC7et8M6oWpeVWVzW0ricMGQTc0Iib8x8t8975icKQ810R2HumlficAdbLOt0aYUIr9Q0IKeAgptfQceA/0?wx_fmt=png


[2]:https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eC7et8M6oWpeVWVzW0ricMGQTKjApib6Qf9Jdpbvn26ek0GZN1KBSOuuvXcTqUKSFLgg1Z3MNFFvC2A/0?wx_fmt=gif


[3]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC7et8M6oWpeVWVzW0ricMGQxU1ByureMSK7fymxwKicSuTROfRcu87nuvx4sWKUBXSR4Ml8PicJTJ7w/0?wx_fmt=png


[4]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC7et8M6oWpeVWVzW0ricMGQFSCV5dWaKK6KL0xSwAWNIiaibIwT6jmQW8AN6eBslhvzgbZYBWHgu7og/0?wx_fmt=png


[5]:https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eC7et8M6oWpeVWVzW0ricMGQFZppFrfTax1uHU60xdZtPTa3JVrvpnv4adBgUDYfcRRgLydnZPibb8A/0?wx_fmt=gif


[6]:https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eC7et8M6oWpeVWVzW0ricMGQ706Dx0MZjNPFtot98hqdR1ceYtJBSvrUAFED1aRcpvouPok8P3qMlw/0?wx_fmt=gif


[7]:https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eC7et8M6oWpeVWVzW0ricMGQbtse1PMNctEYLH9CBUwlNHl5a9ZWg3rKkdPRly6AujbF1iaRFoJRiafA/0?wx_fmt=gif


[8]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC7et8M6oWpeVWVzW0ricMGQKeHycouNS9TGaUgQgYGia2RrmDk2ichPL1xWBEDhWZJaluE0ib9t1bTSg/0?wx_fmt=png


