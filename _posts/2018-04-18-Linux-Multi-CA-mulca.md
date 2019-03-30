---
layout:     post
title:      当一个文件中有个证书链
date:       2018-04-18
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - Linux
    - Openssl
---

<style>
img{
  display:block;
  margin:0
  auto;
}
</style>

<meta name="referrer" content="never">

公众号是真的好久没更新了，实在抱歉，过年之后到现在终于觉得开始清闲了一下，最近一直在整理在日本时候的照片，主要最后一个月末赶上了樱花季，得以观赏了樱花盛开的画面。

这两天遇到了一个问题，就是在查看一个设备证书的时候，证书文件中包含了三个证书，分别是一个根证书和两个子证书，也就形成了一个从根到子证书再到孙证书的证书链：

```
[coreuser@HK-CentOS ca]$ cat ca.crt | grep -w "CERTIFICATE"
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
[coreuser@HK-CentOS ca]$
```

通过openssl命令来查看一下证书的x509标准输出：
```
[coreuser@HK-CentOS ca]$ openssl x509 -in ca.crt -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            76:29:aa:20:fa:8a:8e:76:24:a2:19:36:f4:ad:1a:aa
    Signature Algorithm: sha1WithRSAEncryption
        Issuer: C=US, O=VeriSign, Inc., OU=VeriSign Trust Network, OU=Terms of use at https://www.verisign.com/rpa (c)10, CN=VeriSign Class 3 International Server CA - G3
        Validity
            Not Before: Sep 17 00:00:00 2015 GMT
            Not After : Aug 31 23:59:59 2016 GMT
        Subject: C=CN, ST=Beijing, L=Beijing, O=Beijing Baidu Netcom Science Technology Co., Ltd., OU=service operation department, CN=baidu.com
        Subject Public Key Info:
         ..............
         e1:44:28:42:c5:dd:13:a4:51:a8:bf:fe:30:da:93:36:c5:1e:
         76:e0:c6:cd
[coreuser@HK-CentOS ca]$
```

得到的结果永远都是一个证书的x509标准，如果对此文件熟悉的操作员可以知道此文件中是有三个证书的，但是如果第一次接触就可能被openssl的输出误导。

所以基于openssl的基础上写了一个mulca的脚本来查看这种一个文件中包含多个证书的情况，当然一文件一证书的情况也是可以的。

理论上就是通过判断将各个证书分批调用openssl来进行解析，具体代码如下：
```
#!/bin/sh

sub_ca_file="certificatexx.crt"
ca_number=1
if [ -f  $sub_ca_file ];then
    rm -rf $sub_ca_file
fi

if [ -z $1 ];then
    echo "ERROR: missing the file name";exit
fi

if [ -f $1 ];then
    ca_total_number=`cat $1 | grep -w "\-----BEGIN\ CERTIFICATE-----" -c`
    if [ $ca_total_number != "0" ];then
        echo "Total Certificate found: $ca_total_number";echo
        while read line || [ -n "$line" ]
        do
            if [[ $line == "-----END CERTIFICATE-----" ]];then
                echo $line >> $sub_ca_file 2>&1
                echo "########Certificate $ca_number#######"
                openssl x509 -in $sub_ca_file -text -noout
                rm -rf $sub_ca_file 2>&1
                echo;echo;ca_number=$(($ca_number+1))
            else
                echo $line >> $sub_ca_file 2>&1
            fi
        done < $1
    else
        echo;echo "An incorrecte certificate file.";echo;echo
    fi
else
    echo "File \"$1\" is not existed!"
fi
```

脚本运行结果如下：
```
[coreuser@HK-CentOS ca]$ ./mulca ca.crt
Total Certificate found: 3

########Certificate 1#######
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            76:29:aa:20:fa:8a:8e:76:24:a2:19:36:f4:ad:1a:aa
    Signature Algorithm: sha1WithRSAEncryption
        Issuer: C=US, O=VeriSign, Inc., OU=VeriSign Trust Network, OU=Terms of use at https://www.verisign.com/rpa (c)10, CN=VeriSign Class 3 International Server CA - G3
        Validity
            Not Before: Sep 17 00:00:00 2015 GMT
            Not After : Aug 31 23:59:59 2016 GMT
        Subject: C=CN, ST=Beijing, L=Beijing, O=Beijing Baidu Netcom Science Technology Co., Ltd., OU=service operation department, CN=baidu.com
        Subject Public Key Info:
            ...............
    Signature Algorithm: sha1WithRSAEncryption
         ...............
         76:e0:c6:cd


########Certificate 2#######
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            64:1b:e8:20:ce:02:08:13:f3:2d:4d:2d:95:d6:7e:67
    Signature Algorithm: sha1WithRSAEncryption
        Issuer: C=US, O=VeriSign, Inc., OU=VeriSign Trust Network, OU=(c) 2006 VeriSign, Inc. - For authorized use only, CN=VeriSign Class 3 Public Primary Certification Authority - G5
        Validity
            Not Before: Feb  8 00:00:00 2010 GMT
            Not After : Feb  7 23:59:59 2020 GMT
        Subject: C=US, O=VeriSign, Inc., OU=VeriSign Trust Network, OU=Terms of use at https://www.verisign.com/rpa (c)10, CN=VeriSign Class 3 International Server CA - G3
        Subject Public Key Info:
            ...............
    Signature Algorithm: sha1WithRSAEncryption
         ...............
         78:43:99:a8


########Certificate 3#######
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            18:da:d1:9e:26:7d:e8:bb:4a:21:58:cd:cc:6b:3b:4a
    Signature Algorithm: sha1WithRSAEncryption
        Issuer: C=US, O=VeriSign, Inc., OU=VeriSign Trust Network, OU=(c) 2006 VeriSign, Inc. - For authorized use only, CN=VeriSign Class 3 Public Primary Certification Authority - G5
        Validity
            Not Before: Nov  8 00:00:00 2006 GMT
            Not After : Jul 16 23:59:59 2036 GMT
        Subject: C=US, O=VeriSign, Inc., OU=VeriSign Trust Network, OU=(c) 2006 VeriSign, Inc. - For authorized use only, CN=VeriSign Class 3 Public Primary Certification Authority - G5
        Subject Public Key Info:
            ...............
    Signature Algorithm: sha1WithRSAEncryption
         ...............
         a8:ed:63:6a


[coreuser@HK-CentOS ca]$
```


如果想修改输出内容，可以通过修改脚本中的openssl命令参数来进行控制：
```
openssl x509 -in $sub_ca_file -text -noout
```

后续有时间可能会修改成继承openssl的参数从而通过参数传递来控制输出。

代码已经更新到[Github mulca: https://github.com/MinpuKang/mulca](https://github.com/MinpuKang/mulca)，欢迎使用，如果有兴趣的朋友可以同步完善，在此谢过！

另外在openssl的提问中开发人员提到了在新的1.1.1版本中，将会有一个新的叫做STORE的功能来解决这个问题并给了输出示例，感觉非常完美，敬请期待：[Multi-CAs in one file cannot be listed out #5962](https://github.com/openssl/openssl/issues/5962#issuecomment-382290344)

------------
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)，旅行、摄影。。。</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD1g2UjYu4VfCGHmbhgVqOAnNnJQfN7ZhRVUCopYOsfpPtIEB95VNEqu8trAxJXzGDg01ka6z6wzQ/0?wx_fmt=jpeg" width="30%" />


