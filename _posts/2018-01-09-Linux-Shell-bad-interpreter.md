---
layout: post
title: Linux关于xxx^M导致Shell程序编译错误
date: 2018-01-09
categories: blog
tags: [Linux, Shell]
description: 任何一种错误都应该可以找到原因，但是深究总会触及到一些不可思议的东西
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

在从Windows下移植某些文件尤其是脚本文件到Linux环境之后会出现无法编译的情况，遇到类似如下的错误提示：

**/bin/sh^M: 坏的解释器: 没有那个文件或目录(bad interpreter: No such file or directory)**

如：
```
[coreuser@HK-CentOS ~]$ ./shell.txt
-bash: ./shell.txt: /bin/sh^M: 坏的解释器: 没有那个文件或目录
[coreuser@HK-CentOS ~]$
```

那么这是因为什么导致，又如何解决呢？

#### 1、原因
这个是因为Windows下和Linux的换行符不同导致：

1. Windows中默认的换行符是\r\n；
2. Linux下的换行符是\n。

因此当文件在Windows下编辑之后就会携带\r\n的换行符导致在Linux环境下无法编译，那么如何查看和解决呢？

#### 2、查看
可以是用vi查看文件属性来判断，也可以使用cat命令来直接查看特殊字符。

#### 2.1、使用vi查看
在vi下可以通过使用**set ff**活着全称**set fileformat**查看文件格式来确认，如果显示为dos，那么基本就含有windows下的换行符了：
```
[coreuser@HK-CentOS ~]$ vi shell.txt
#!/bin/sh

whoami
pwd
~
:set ff
  fileformat=dos
```

#### 2.2、 cat命令查看隐藏字符：
下来看一下cat命令可以使用的参数：
```
CAT(1)                                                                            User Commands                                                                           CAT(1)
NAME
       cat - concatenate files and print on the standard output
SYNOPSIS
       cat [OPTION]... [FILE]...
DESCRIPTION
       Concatenate FILE(s), or standard input, to standard output.
       -A, --show-all
              equivalent to -vET
       -b, --number-nonblank
              number nonempty output lines, overrides -n
       -e     equivalent to -vE
       -E, --show-ends
              display $ at end of each line
       -n, --number
              number all output lines
       -s, --squeeze-blank
              suppress repeated empty output lines
       -t     equivalent to -vT
       -T, --show-tabs
              display TAB characters as ^I
       -u     (ignored)
       -v, --show-nonprinting
              use ^ and M- notation, except for LFD and TAB
```

通过帮助文档可以看到-v参数可以用来显示默认不显示的参数，那么其实只要使用**cat -v**就可以查看了：
```
[coreuser@HK-CentOS ~]$ cat -v shell.txt
#!/bin/sh^M
^M
whoami^M
pwd^M
[coreuser@HK-CentOS ~]$
```

然后就是使用-vET这样的参数组合来查看，但是最好用的依然是**-A**参数，首先不用记忆太多的参数，然后就是show-all比较齐全：
```
[coreuser@HK-CentOS ~]$ cat -A shell.txt
#!/bin/sh^M$
^M$
whoami^M$
pwd^M$
[coreuser@HK-CentOS ~]$ cat -vET shell.txt
#!/bin/sh^M$
^M$
whoami^M$
pwd^M$
[coreuser@HK-CentOS ~]$
```


#### 3、修改
修改可以通过vi修改文件格式达到目的，也可以使用sed命令进行直接修改：

#### 3.1、vi模式下的修改办法
vi下的可以在ex转义方式中直接使用**set ff=unix**修改文件格式来进行全文修改，然后**wq**保存退出即可。
```
[coreuser@HK-CentOS ~]$ vi shell.txt
#!/bin/sh

whoami
pwd
~
:set ff=unix
:wq
```

如果Linux下安装了dos2unix的命令，可以直接使用此命令来修改文件格式，效果同上。

#### 3.2、使用sed命令
使用sed命令来直接替换换行符：
```
sed 's/\r//g' filename > filename_new #不在原文中替换，而是保存到新文件中

    OR

sed -i 's/\r//g' filename #直接在原文中替换
```

显然sed命令更直接和方便，而且在shell编程中也更加实用：
比如遇到字符串中使用了\r\n的换行符，导致字符串无法正确调用，就可以使用** echo string | sed 's/\r//g' **这样的组合来修改字符串中的特殊换行符。

以上。

------------
<p align="center">欢迎关注公众号：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD1g2UjYu4VfCGHmbhgVqOAnNnJQfN7ZhRVUCopYOsfpPtIEB95VNEqu8trAxJXzGDg01ka6z6wzQ/0?wx_fmt=jpeg" width="40%" />



