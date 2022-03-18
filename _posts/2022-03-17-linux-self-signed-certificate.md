---
layout:     post
title:      Linux｜一个创建自签名证书的小脚本
date:       2022-03-17
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - Linux
    - 证书
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

这个草稿马上就要半年了，再不整理，可能就要长毛了。

写这个脚本是因为在实验室安装云平台和虚拟设备，需要证书，就使用openssl自建CA并签发证书了，命令虽然几行，但是输入参数和经常需要来回签发证书也是一件麻烦的事情。
![1][1]

而且在使用的时候经常碰到证书Seria一样的问题，原因是同一个CA签发了多个证书没有考虑Serial冲突的问题。
![2][2]

这个脚本就在此情景下应运而生，整合了openssl制作CA和签发证书的过程，并且使用RANDOM变量来设置证书的Serial。

脚本的输入是一个非常标准的配置文件，包括两个部分，一个是CA信息，一个是证书信息。

两部分都是由Common Name和Subject构成，其中Subject遵循openssl需要创建CSR所需要的标准参数格式。

配置文件详情如下：
```
####### Input host FQDNs here and Prefix of Subject
# Subject_Pre_CN include below items:
# C: Country Name (2 letter code)
# ST: State or Province Name (full name) [Some-State]
# L: Locality Name (eg, city)
# O: Organization Name (eg, company) [Internet Widgits Pty Ltd]
# OU: Organizational Unit Name (eg, section)

# CN: Common Name (e.g. server FQDN or YOUR name)


## Root Subject: CN_ROOTCA and Subject_PreCN_ROOT are fixed parameters, values should be updated, CN_ROOTCA is for the CN of Root CA.
CN_ROOTCA: www.hk314.top
Subject_PreCN_ROOT: C=CN/ST=LN/L=DL/O=HK/OU=Root


## Client Certificate Subject: CN_Cert and Subject_PreCN_Cert are fixed parameters, values should be updated, CN_Cert is for the CN of client.
CN_Cert: self.cert.hk314.top
Subject_PreCN_Cert: C=CN/ST=LN/L=DL/O=HK/OU=Self
````

配置文件准备好之后就可以直接运行脚本制作CA机构并签发证书了，也支持使用已经创好的CA签发证书，帮助如下：

```
[coreuser@HK:ca_self_signed]$ ./cert_self_signed.sh -h
This is used to generate certificate with an existed CA or self-signed certificates based on openssl!
Version: 1.0

Usage:
 cert_self_signed.sh [-h] -c ConfigFile [-ca CACert -key CAKey] 

Options:
 -h        Show the help
 -c        Config File for Certificate Subject
 -ca       An existed CA certificate(with relative path or absolute path)
 -key      The existed CA private key file(with relative path or absolute path)
           Note: -ca and -key must be set in pair.

For Example:
---------------------------------------------------------------------------------
 1. Show Help:
    user@host > cert_self_signed.sh -h
 
 2. Generate a ROOT CA and self-signed certificate:
    user@host > cert_self_signed.sh -c config.cfg

 3. Generate certificate with an existd ROOT CA:
    user@host > cert_self_signed.sh -c config.cfg -ca ca.crt -key ca.key
    OR
    user@host > cert_self_signed.sh -c config.cfg -ca /home/user/ca.crt -key ca.key
 
---------------------------------------------------------------------------------
```

脚本输出不仅仅有签发的证书，也会打印出相应的verify的命令，打印这个是因为之前做证书verify的时候经常忘记参数，所以就直接写到脚本输入里了。

脚本做了很多的容错，比如配置文件缺失，配置文件格式有问题，使用已经制作好的CA签发证书的时候要确认CA的证书和私钥匹配。

这个脚本不仅仅可以快速制作CA并签发证书，同时脚本内容也不复杂，可以顺便熟悉openssl命令和签发证书的流程。

脚本、使用介绍和证书样例已经同步到Github，欢迎点击[**阅读原文**](https://github.com/MinpuKang/generate-self-signed-certificate)查看。

PS：这个Repository的README中超链了一篇介绍数字签名的古老博客(已经十几年之久)，但是对于理解证书、数字签名等等非常之浅显易懂​。

------------
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)，旅行、摄影。。。偶尔分享技术周边</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[1]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCUickUcuIz4aa1Z5KJdsWYqCnZFiankEgPQQ9PVzH0nPV3WUsBTxJmKOTH6Un3pODWPYDfjeJI4u7g/0?wx_fmt=png


[2]:https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCUickUcuIz4aa1Z5KJdsWYqwwiagM1C5rpvQssCSQln095ia4KgAOjYN2U3ZdxJOe1oHEiaNOhLkmxWQ/0?wx_fmt=png

