---
layout: post
title: Linux来创建一个-rf文件吧(奸笑)
date: 2017-12-06
categories: blog
tags: [Linux, rm]
description: 生活的乐趣就是不断的尝试^_^
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

今天下班的时候看到同事朋友圈发了一个图说创建了个文件删不掉鸟╮(￣▽￣"")╭

仔细一看是一个”-rf *“的文件，也就是以连字符“-”为开头，突然想起了前两天写tree那个shell助参数-h|--help的时候觉得如果创建了这样的两个名字文件和文件夹要怎么办呢？

嗯，然后就从Windows下创建了-h和--help的文件文件夹传到了linux里，开始实验，结果因为脚本里使用了cd命令，所以在运行**tree -h**的时候相当于进行了一下**cd -h**，最后就放弃了。

但是问题来了，怎么删除呢？在进行简单尝试而不得的时候，使用Filezilla sftp到server以Windows的方式进行右键删除(^_−)−☆

#### 1.rm的双连线或者使用路径删除“-文件”
今天看到了同事的这个问题，秉承着虚心学习的精神回来仔细看了下rm的帮助文档**man rm**

然后看到了这样的描述：
```
To remove a file whose name starts with a '-', for example '-foo', use one of these commands:
       rm -- -foo
       rm ./-foo
```
虽然如同发现了新大陆一样，但是也略怅然若失，Linux的世界果然博大精深呀。

那么这样的操作到底可以不可以呢？遂进行了实验：
```
[[coreuser@HK-CentOS delete]$ echo "test" > -rf\ \*
[coreuser@HK-CentOS delete]$ ll
总用量 4
-rw-rw-r--. 1 coreuser coreuser 5 8月   5 03:14 -rf *
[coreuser@HK-CentOS delete]$ rm "-rf *"
rm：无效选项 --
Try 'rm ./'-rf *'' to remove the file "-rf *".
Try 'rm --help' for more information.
[coreuser@HK-CentOS delete]$ rm -- "-rf *"
[coreuser@HK-CentOS delete]$ ll
总用量 0
[coreuser@HK-CentOS delete]$

```
果然是可以删除的。

那么我们再来试试相对路径“./”吧：
```
[coreuser@HK-CentOS delete]$ echo "test" > -rf\ \*
[coreuser@HK-CentOS delete]$ ll
总用量 4
-rw-rw-r--. 1 coreuser coreuser 5 8月   5 03:16 -rf *
[coreuser@HK-CentOS delete]$ rm "./-rf *"
[coreuser@HK-CentOS delete]$ ll
总用量 0
[coreuser@HK-CentOS delete]$
```
太棒了，也是可以的！帮助文档诚不欺我也(^_−)−☆

既然相对路径也可以，那么绝对路径更没问题了吧？
```
[coreuser@HK-CentOS delete]$ echo "test" > -rf\ \*
[coreuser@HK-CentOS delete]$ ll
总用量 4
-rw-rw-r--. 1 coreuser coreuser 5 8月   5 03:20 -rf *
[coreuser@HK-CentOS delete]$ rm "/home/coreuser/delete/-rf *"
[coreuser@HK-CentOS delete]$
[coreuser@HK-CentOS delete]$ ll
总用量 0
[coreuser@HK-CentOS delete]$
```
果然No Problem呀！

然后我又继续了一个实验：
```
[coreuser@HK-CentOS delete]$ ll
总用量 4
-rw-rw-r--. 1 coreuser coreuser 5 8月   5 03:24 -rf
[coreuser@HK-CentOS delete]$ rm -- -rf
rm: 无法删除"-rf": 没有那个文件或目录
[coreuser@HK-CentOS delete]$
```
咦？为什么这个不起作用了呢？

哈哈，其实我没有贴创建文件的命令所以造成了一个混淆以为这个文件名字就只是“-rf”，其实我是通过**“echo "test" > -rf\   ”**，也就是在文件名字中添加了空格，这个时候当我们进行rm命令的时候最好**使用tab键进行补全**，这样就不会错过任何空格字符了：
```
[coreuser@HK-CentOS delete]$ rm -- -rf\  [使用tab补全]
[coreuser@HK-CentOS delete]$ ll
总用量 0
[coreuser@HK-CentOS delete]$
```

#### 2.其他的参数呢？
既然rm参数支持双连字符来处理以**“-”**开头的文件，那么其他的命令可不可以使用呢？

**touch命令**
```
[coreuser@HK-CentOS delete]$ touch --help
用法：touch [选项]... 文件...
...........
要获取完整文档，请运行：info coreutils 'touch invocation'
[coreuser@HK-CentOS delete]$ touch -- --help
[coreuser@HK-CentOS delete]$ ll
总用量 0
-rw-rw-r--. 1 coreuser coreuser 0 8月   5 03:45 --help
[coreuser@HK-CentOS delete]$ rm -rf -- *
[coreuser@HK-CentOS delete]$ touch ./--help
[coreuser@HK-CentOS delete]$ ll
总用量 0
-rw-rw-r--. 1 coreuser coreuser 0 8月   5 03:45 --help
[coreuser@HK-CentOS delete]$
```

**ls命令**
```
[coreuser@HK-CentOS delete]$ ls
--help
[coreuser@HK-CentOS delete]$ ls --help
用法：ls [选项]... [文件]...
..........
要获取完整文档，请运行：info coreutils 'ls invocation'
[coreuser@HK-CentOS delete]$
[coreuser@HK-CentOS delete]$
[coreuser@HK-CentOS delete]$ ls -- --help
--help
[coreuser@HK-CentOS delete]$
```

**mkdir命令**
```
[coreuser@HK-CentOS delete]$ mkdir -h
mkdir：无效选项 -- h
Try 'mkdir --help' for more information.
[coreuser@HK-CentOS delete]$ mkdir ./-h
[coreuser@HK-CentOS delete]$ ll
总用量 0
drwxrwxr-x. 2 coreuser coreuser 6 8月   5 03:47 -h
-rw-rw-r--. 1 coreuser coreuser 0 8月   5 03:45 --help
[coreuser@HK-CentOS delete]$ rm -- -h
rm: 无法删除"-h": 是一个目录
[coreuser@HK-CentOS delete]$ rm -rf ./-h
[coreuser@HK-CentOS delete]$ mkdir -- -h
[coreuser@HK-CentOS delete]$ ll
总用量 0
drwxrwxr-x. 2 coreuser coreuser 6 8月   5 03:47 -h
-rw-rw-r--. 1 coreuser coreuser 0 8月   5 03:45 --help
[coreuser@HK-CentOS delete]$
```

**cd命令**
```
[coreuser@HK-CentOS delete]$ cd -h
-bash: cd: -h: 无效选项
cd: 用法:cd [-L|[-P [-e]]] [dir]
[coreuser@HK-CentOS delete]$ cd -- -h
[coreuser@HK-CentOS -h]$ cd ..
[coreuser@HK-CentOS delete]$ cd ./-h/
[coreuser@HK-CentOS -h]$
```

**more命令**
```
[coreuser@HK-CentOS -h]$ echo test> -test
[coreuser@HK-CentOS -h]$ ll
总用量 4
-rw-rw-r--. 1 coreuser coreuser 5 8月   5 03:50 -test
[coreuser@HK-CentOS -h]$ more -test
more: 未知选项 -test
.........
  -V        输出版本信息并退出
[coreuser@HK-CentOS -h]$ more ./-test
test
[coreuser@HK-CentOS -h]$ more -- -test
more: 未知选项 -test
.........
  -V        输出版本信息并退出
[coreuser@HK-CentOS -h]$ more -- "-test"
more: 未知选项 -test
用法：more [选项] 文件...
.........
  -V        输出版本信息并退出
[coreuser@HK-CentOS -h]$ more -- "./-test"
test
[coreuser@HK-CentOS -h]$
```

尝试了几个命令之后发现基本所有的命令都可以使用双连字符（--）或者路径的方式来完成针对以连字符（-）开头的文件/文件夹操作。

以上就是今天的实验内容，试一试是一切知识最好来源。
实验环境CentOS：
```
[coreuser@HK-CentOS -h]$ uname -a
Linux HK-CentOS.localdomain 3.10.0-327.el7.x86_64 #1 SMP Thu Nov 19 22:10:57 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
[coreuser@HK-CentOS -h]$ more /proc/version
Linux version 3.10.0-327.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.3 20140911 (Red Hat 4.8.3-9) (GCC) ) #1 SMP Thu Nov 19 22:10:57 UTC 2015
[coreuser@HK-CentOS -h]$
```

------------
<p align="center">欢迎关注公众号：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAoGfjsaJt2NQ0a9AKmrIRoR9gKlX1I78Z4AoPtjyEPM56slw9gAQBdAHjHckbw4h93FvVVATBuLQ/0?wx_fmt=jpeg" width="40%" />

<p align="center">感觉内容不错，读后有收获？欢迎小额赞助：</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAzA577Ce49rCLiby9EtT195GRiaqKCT6QCQ5Weia9OZD72MJz4ABlqAy1gbHepk5hHM464hCiarQRI7w/0?wx_fmt=jpeg" width="30%" />

  [1]:


