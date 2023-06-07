---
layout:     post
title:      应用｜递归批量创建文件夹的Python工具
date:       2023-06-07
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - 应用
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

![那天的星海有点儿绚丽][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>


文件管理一直是电脑相关工作中容易被忽视但是却非常重要的事情，就如同线下的档案或者图书管理，有很好的逻辑结构，在索引和辨识度上都可以起到事半功倍的效果。

项目中需要存放各种各样的文件，分门别类，因此一个很好的文件夹结构就可以让这些文件更好的存放，也方便大家可以很快的找到。

最近的一个项目大概涉及到十几个产品，每个产品属于不同的部门，而且每个产品在不同的地方还要建设多个，就导致文件夹结构异常复杂，有的需要按照部门来创建，有的需要具体创建到每个需要建设的产品。
```
#左右滑动
rootFoler
    parentFolder1
        sonFolder1
            grandsonFolder1
        sonFolder2
    parentFolder2
        sonFolder2
        sonFolder3
            grandsonFolder1
            grandsonFolder2
```

因此创建文件夹的时候一个涉及到递归多层级创建。

Windows下需要一个一个创建，就是点点点，Linux下使用mkdir也需要罗列好各个文件夹的路径然后一条条的命令运行。

不管是效率还是耐心都是一种消耗。

所以就创建了这个recursive create folder的python小工具。

这个工具基于yaml，将folder的层级写到一个yaml文件后传递给脚本就可以创建好所有相关的文件夹了。

yaml的一个template如下，只需要定义各个文件夹的名字，然后每一个名字后以冒号“:”结尾，也就是每个文件夹的名字是yaml里的一个key，还有要注意层级缩进：
```
#左右滑动
rootFoler:
    parentFolder1:
        sonFolder1:
            grandsonFolder1:
        sonFolder2:
    parentFolder2:
        sonFolder2:
        sonFolder3:
            grandsonFolder1:
            grandsonFolder2:
```

运行非常简单，如下：
```
#左右滑动
$ ./foldergen.py template.yaml
Current Folder /home/user/
    Folder "rootFoler/parentFolder1/sonFolder1/grandsonFolder1" Create Successfully!
    Folder "rootFoler/parentFolder1/sonFolder2" Create Successfully!
    Folder "rootFoler/parentFolder2/sonFolder2" Create Successfully!
    Folder "rootFoler/parentFolder2/sonFolder3/grandsonFolder1" Create Successfully!
    Folder "rootFoler/parentFolder2/sonFolder3/grandsonFolder2" Create Successfully!
```

最终yaml里定义的文件夹名字都被创建，可以使用tree命令查看：
```
#左右滑动
$ tree ./rootFoler
./rootFoler
├── parentFolder1
│   ├── sonFolder1
│   │   └── grandsonFolder1
│   └── sonFolder2
└── parentFolder2
    ├── sonFolder2
    └── sonFolder3
        ├── grandsonFolder1
        └── grandsonFolder2

9 directories, 0 files
```

有的朋友可能会说，这个运行得在Linux环境下，我用的Windows没办法运行呀。

Windows下推荐开启WSL构建一个内嵌的Linux环境，如果开启WSL？移步[Linux｜二更WSL打造Windows下更顺畅的双系统](https://mp.weixin.qq.com/s/BbXxE0_6uLETds-N1kt-2A)

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBxCPUriab12goWCZnnJmKnxytYxSgTktwmicdpBrL4N2sLUbFc6WzZTUqHWCKUnkhRcrD0JFbYHxmw/640?wx_fmt=jpeg

