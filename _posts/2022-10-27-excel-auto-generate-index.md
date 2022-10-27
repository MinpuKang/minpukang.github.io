---
layout:     post
title:      应用｜让Excel的目录超级自动化
date:       2022-10-27
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 应用
    - excel
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

![][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

Excel是各种数据统计维护使用的工具。

不管是做网络规划，还是做财务报表，或者是工程统计，都会将数据分门别类地定义在各种工作表里。

然后在一连串的工作表中来回跳转会异常头痛，所以必然想做一个目录索引以方便跳转，就如同Word里的目录索引一样。

​那么你有没有为了生成Excel的目录而痛苦？在一次次右键选择超链，再选择一个工作表，最后修改下显示文本，循环往复直到所有的工作表都定义到目录中。

然后一旦有工作表的名字被修改，超链接也必须修改后才可以使用，否则就是一个查无此表；再或者新加入的工作表后，再次需要右键选择编辑等等操作后才能更新到目录中。

​那么有没有一个好办法能让这一切都自动化起来呢？

答案当然是：有。

比如像这样，目录在工作表切换后自动生成。

![][2]

比如当工作表的位置发生变动后，目录也自动跟随调整位置。

![][3]

比如添加新的工作表后，目录在对应的位置就自动添加了超链索引。

![][4]

比如工作表的名字更改后，目录里的名字和超链也自动修改。

![][5]

比如工作表被删除后，目录也自动删除其对应的索引。

![][6]

这样的自动化目录是不是看起来就丝滑了不少，富裕的时间至少可以去喝一杯82年的咖啡。

那么是如何实现的呢？

其实就是在目录工作表中加载了几行VBA代码，具体的代码如下（左右滑动看完整代码）：
```
Private Sub Worksheet_Activate()
Application.ScreenUpdating = False

Range("A1") = "Table of Content"
If Range("A5000").End(xlUp).Row >= 2 Then
    Range("A2", "A" & Range("A5000").End(xlUp).Row).ClearContents
End If
For i = 3 To Sheets.Count
    j = i - 1
    Range("A" & j) = Sheets(i).Name
    name_cell = Range("A" & j)
    Range("A" & j).Select
    ActiveSheet.Hyperlinks.Add Anchor:=Selection, Address:="", SubAddress:= _
        "'" & name_cell & "'" & "!A1", TextToDisplay:=name_cell
    Next
    Application.ScreenUpdating = True
End Sub
```

其中有两处可以适配修改的地方：

1. 在目录工作表的第一个单元格A1中定义表头：Table of Content
```
Range("A1") = "Table of Content"
```

2. for循环里i和j的含义：在目录工作表中A列的第j个单元格中生成第i个工作表的索引链接，所以要很好的把握它们俩的关系。

- 比如如下代码是因为我习惯定义第一个工作表作为此Excel的描述和修订版本的追踪，第二个工作表是目录，所以需要在目录里生成第三个工作表和之后的工作表的索引：
```
For i = 3 To Sheets.Count
    j = i - 1
```

- 如果你的习惯是第一个工作表是目录，然后要生成第二个和之后的工作表的索引，那么代码需要修改成如下即可：
```
For i = 2 To Sheets.Count
    j = i
```

哪里添加代码和让其永久生效呢？

选择Developer中的Visual Basic，然后点击工作表后添加代码保存即可。

![][8]

​然后需要将Excel存储为支持宏(Macro-Enabled)的格式即可永久生效，比如office2007开始的xlsm：

![][9]

可能你会说excel里没有Developer选项，这就需要在选项设置中勾选Developer使其显示：

![][10]


可能会遇到的问题。

一个是目录工作表后移导致目录混乱，所以要严格控制For循环中i和j的关系，并且保证目录工作表的位置不变：

![][11]

第二个可能遇到的问题是再次打开Excel后VBA不工作的问题，主要原因是宏被禁止了。

方案一是打开消息提示窗口，然后在每次打开excel的时候就会有安全问题的提示，直接允许即可：

![][12]
![][13]

​方案二就是直接允许运行VBA的宏，一劳永逸，但是会有安全方面的风险，比如我司就直接不允许修改宏配置：

![][14]

最后还有一个小bug，就是工作表的名字不能是数字，否则会提示bug，这个时候点击End然后修改工作表名字即可：

![][15]

到这里，Excel使用VBA生成自动化的目录索引就告一段落了，或者你有更好的方案也欢迎留言私信分享！

最后，如果你觉得还不错，欢迎转发点赞在看收藏四连！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYxIZNKpj5GMzclAI6wrO123XP9hY1csWvGytssBWSzdVKhKjIo1drKA/0?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYhggAeAJdHE8BYHEbXVRw32UoXiatJIViaRsvjtU6qHgZUlmH5nlqfiazA/0?wx_fmt=gif


[3]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYPUESIrfdfiaItByxNMjgrZkYMdexPPfkhn02fNrGjcQ3RhW2GkmbZzA/0?wx_fmt=gif


[4]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYDxLTcSHfIjDCkia0BVG6iaOWP6dpH9OoG3n0co8zstxaxLic9z2INibDyw/0?wx_fmt=gif


[5]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYRUvqStZ916bAW9uGnoTqcq2EfvE1vwXn7u7jS5wGphvzH9cdggHLkg/0?wx_fmt=gif


[6]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYsZkV8XeH1xhz1Z5zPZM74JJWQ98hKQgCoslLE5ztEdyX75uwAIxu0Q/0?wx_fmt=gif


[7]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYk6NVJmibZyknJxOicRfwSLUiarw05tgTISlicv9AsDYcLYF3yl68bAvrtA/0?wx_fmt=gif


[8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYBhfu7SWxK0ibuolAGpc2eNGORBupQoz69Ucn5ZAOSeVZbhdOJThj7iaw/0?wx_fmt=png


[9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYSJiaM5AQydXynhAalRHsicFySNrS8qwZRFBlAog4eIkvo9VrkpyqV92Q/0?wx_fmt=png


[10]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYkUmeGyofibVdWrNGRF5SicpmQ1KREQ3yhkbbN8ePoFO699kue1B4b47w/0?wx_fmt=png


[11]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYiarWqNicXMMbkS2mWCicUib4qmUytk3xLXnF8wFD2cm7zWdEuG8OT4xBSA/0?wx_fmt=png


[12]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYicQtr0ic5x5LGgAQH5diaoziaeP0qc7fJOQ8V3OlHl8ZlibzIrCzzL1I85Q/0?wx_fmt=png


[13]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYmIfzvXnmxud7SV8JX2j04TZCuU3gJdfuDleSxI99bVc7xynnLJKf7A/0?wx_fmt=png


[14]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYNXv89SmASRscMC5icBGZenHolmbHvaOkMPrZ4FYPcw0VjTuMucvtNJA/0?wx_fmt=png


[15]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCWR0fM2otXLIOf01GjluIYBlAQ12MIox2u2L3gibG8TYibulOLqibD9wlNupOJahrf2WcEF7ZSzKmyA/0?wx_fmt=png



