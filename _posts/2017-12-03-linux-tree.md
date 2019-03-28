---
layout: post
title: Linux下树状显示文件夹结构的一个脚本
date: 2017-12-03
categories: blog
tags: [Linux, tree]
description: 一行命令，一棵树！
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

转眼已经到了17年的最后一个月份，又一个匆忙的年份就这样来到了尾声，感觉才刚刚步入17年却马上结束了。

长大之后的时间总是过的措手不及，转瞬即逝，也许一年一年重复而过，而对年终也不及小时候那种热烈的期盼，没有期待也就没有了终点，所有走过的每一个时刻都不知道为了什么。

但是转身来看，17年的却是做了太多的事情，而最大的一件莫过于组建了自己的家庭，从此跟妞儿开始了新的征程，兼程相伴。

11月的公号和博客都没有任何的举动，孰不为也，而略感力不从心，月初回国两周匆忙的把房子整理了一下，周末逛家居跑建材，然后回到日本后周末加班，至此才略感周身放松，正好最近写了一个脚本分享一下。

这个脚本其实跟Linux中的tree工具重叠，只是因为tree工具在Linux中如CentOS/Ubuntu等系统中非默认安装，因此使用时需先安装，而在某些环境的限制下不能快速安装，因此萌生了写一个小脚本的念头。

在网上发现了一个同僚写过的脚本，具体可参考[Unix Tree/Linux Tree：http://centerkey.com/tree/](http://centerkey.com/tree/)

但是在阅读代码之后看到了一些弊端，如：

1. 只可以列举子目录，而文件却不在显示的行列。
2. 原脚本没有多输入的参数值进行判断，可能会有异常输出。
3. 如果不看源码便不知道如何使用，也就是没有帮助打印。

鉴于以上的弊端做了改进，代码已经更新到[Github: https://github.com/MinpuKang/tree](https://github.com/MinpuKang/tree)

整个代码主要是调用了**”ls -R“打印出文件目录**，再通过**sed进行正则替换**等过程，从而得到想要的树状图。

修改之后的脚本使用如下：

##### a. 帮助参数-h|--help:
```
[coreuser@HK-CentOS ~]$ tree -h
This is used to list a directory with a dendritical structure
Usage:
   $ tree [directory] [-h|--help]

Examples:
   $ tree
   $ tree /etc/opt
   $ tree ..
```
##### b. 使用举例：
```
[coreuser@HK-CentOS ~]$ tree

/home/coreuser
 .
 |-bin
 |---tree
 |-file1
 |-script
 |---tree.sh
 |-sname
 |-sship
 |---fixed.sh
 |---ip
 |---original.sh

[coreuser@HK-CentOS ~]$ tree ..

/home
ls: 无法打开目录./om_admin: 权限不够
 .
 |-coreuser
 |---bin
 |-----tree
 |---file1
 |---script
 |-----tree.sh
 |---sname
 |---sship
 |-----fixed.sh
 |-----ip
 |-----original.sh
 |-om_admin

[coreuser@HK-CentOS ~]$ tree sship

/home/coreuser/sship
 .
 |-fixed.sh
 |-ip
 |-original.sh

[coreuser@HK-CentOS ~]$

```

##### c. 最终代码：
```
#!/bin/sh
########################################################
#  Linux Tree to list the directories and files        #
#  Version:1.0                                         #
#  Owner: Minpu Kang                                   #
#  Introduction: This is an updated one based on the   #
#                one(centerkey.com/tree) deployed by   #
#                Dem Pilafian                          #
#                                                      #
#  What is updated:                                    #
#       1. Update to list not only sub-directories but #
#          also files                                  #
#       2. Add the help printout with -h|--help        #
#       3. Optimized to judge the input is file or     #
#          directory following the command             #
#                                                      #
#  An example for setup:                               #
#     $ vi tree.sh                                     #
#     $ chmod u+x tree.sh                              #
#     $ ln -s tree.sh ~/bin/tree                       #
#     $ echo "PATH=~/bin:\${PATH}" >> ~/.profile       #
#                                                      #
#  Link: https://github.com/MinpuKang/tree             #
#                                                      #
########################################################

script_name=`basename $0`

if [[ "$1" == "-h" ]] || [[ "$1" == "--help"  ]]
   then
   echo "This is used to list a directory with a dendritical structure"
   echo "Usage:                                                       "
   echo "   $ $script_name [directory] [-h|--help]                    "
   echo "                                                             "
   echo "Examples:                                                    "
   echo "   $ $script_name                                            "
   echo "   $ $script_name /etc/opt                                   "
   echo "   $ $script_name ..                                         "
   exit
fi


echo

if [ "$1" != "" ] && [ -d $1 ]  #if parameter exists, use as base folder
   then cd "$1"
elif [ "$1" != "" ] && [ -f $1 ]
   then echo "   -> $1 is a file";echo;exit

elif [ "$1" != "" ]
   then echo "   -> $1:No such file or directory";echo;exit

fi

pwd

for i in `ls -R`; do if [ `echo $i| grep ":$" -c` == 1 ]; then path=`echo $i | sed -e 's/\/*:$//'`;echo $path;else file="$path/$i";echo $file ;fi ;done | sort | uniq  | sed -e 's/[^-][^\/]*\//--/g' -e 's/^/ /' -e 's/-/|/'
# 1st sed: remove colons: -e 's/\/*:$//'
# 2nd sed: replace higher level folder names with dashes: -e 's/[^-][^\/]*\//--/g'
# 3rd sed: indent graph three spaces: -e 's/^/   /'
# 4th sed: replace first dash with a vertical bar: -e 's/-/|/'
if [ `ls | wc -l` == 0 ]   # check if no files or folders
   then echo "   -> no files or sub-directories"
fi

echo

exit
#finsihed
```

以上就是这个脚本的相关内容，欢迎使用！

------------
<p align="center">欢迎关注公众号，摄影，旅行，瞎聊，等等等：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD1g2UjYu4VfCGHmbhgVqOAnNnJQfN7ZhRVUCopYOsfpPtIEB95VNEqu8trAxJXzGDg01ka6z6wzQ/0?wx_fmt=jpeg" width="30%" />

  [1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCJxdENBAiclBhsWAUGYsQxSvqb8Z1SItcCqgzGGUjgWLG1zEZMkicmTEfiaouUrsIjKnbXWbTIEmJzw/0?wx_fmt=jpeg

