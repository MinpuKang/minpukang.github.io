---
layout:     post
title:      Linux｜ssh免密登录的那件小事儿
date:       2025-04-27
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - Linux
    - ssh
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

![@七禾页话，大连的野生梅花鹿已经是明信片的存在了][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

作为一个网络运维工程师，ssh是最最最常用的一个linux命令之一，尤其是现在云化之后，每天都需要维护大量的服务器、VM等，所以每天都会ssh各种服务器、VM，每一次ssh都要使用用户名、IP和密码。

在现在安全至上的前提下，密码设置也需要越来越复杂，而且要不定期修改密码，同时对于某些业务需求下无法输入密码，等等这些问题就需要免密登录。

那么什么是免密登录，如何设置免密登录呢？

聊免密登录前，先分清客户端和服务器，如下图所示，左边是客户端，是当前用户所在的设备，想登录到右边的服务器，在ssh会话中客户端和服务器是可以互相变化的，唯一的判定标准就是当前所在的设备就是客户端，要登录的目标设备就是服务器。

![@七禾页话][2]

从客户端登录服务器的时候需要服务器的如下信息：
```
1. IP or URL（URL需要DNS解析成IP)
2. Port，默认是22，可以在服务器中的/etc/ssh/sshd_config查看
3. 用户名和密码，是服务器上定义的用户名。
```

这里需要注意的是用户名是服务器上定义的用户名，可以和客户端里的用户一样，也可以不一样。

比如下图中，当前所在的设备叫做jumpserver，要登录到一个叫做instance-20250427-033831的设备，所以jumpserver就是当前ssh会话的客户端，而instance-20250427-033831就是服务器：

![@七禾页话][3]

我ssh了两次，第一次直接ssh了instance-20250427-033831的IP，登录进去后可以看到Prompt是“ubuntu@instance-20250427-033831”，也就是说如果ssh中没有指定用户名，那就会用客户端当前的用户作为服务器中的用户登录，但是服务器中不一定有这个用户。

比如下图中，在jumpserver这个客户端上的用户是user1，直接 `ssh 10.128.0.4` 之后，默认就是用user1这个用户登录服务器，但是一直都是失败的，我们在服务器上查看用户的时候可以发现，服务器上并没有user1这个用户，所以失败。

![@七禾页话][4]

那么接下来，我们做从客户端到服务器的免密登录。

### 查看客户端上的ssh key pair

为什么要查看是否已经有ssh ker pair了呢？有的人可能习惯性的直接使用 `ssh-keygen` 生成ssh key pair了，虽然如果已经了ssh key pair也会提示已经存在，是否overwrite，但是有的时候有的人是会直接回车回车回车直到生成完，中间的提示根本不看。

![@七禾页话][5]

但是如果已经有生成的key pair，这组key可能已经有其他的用途了，这个时候一旦被覆盖了之后，其他的用途无法继续可能会导致莫名的问题，排错的时候也很容易忽略这个key pair是否已经被重置。

因此在生成任何东西的前最好要先看看是否已经存在，如果存在直接用即可，命令也非常简单：
```
ls -l ~/.ssh/
```

![@七禾页话][6]

### 生成ssh key pair 

如果已经存在ssh key pair了，可以直接使用，那么如果不想直接使用已有的：


- 备份当前的key pair，然后生成。

- 生成key pair的时候设置自定义的路径，也可以直接使用 `-f` 参数指定，但是这样会导致之后ssh的时候需要使用额外的 `-i` 参数。

![@七禾页话][7]

![@七禾页话][8]

- 换一个算法生成key pair，需要 `-t` 参数指定算法，比如下图中使用了椭圆曲线加密算法ECDSA（521位）生成了密钥。

![@七禾页话][9]


在 SSH 中，常见的密钥类型包括以下几种：

- RSA：这是最早的 SSH 密钥类型之一，使用 RSA 加密算法。RSA 密钥在 SSH 中被广泛使用，并且是许多 SSH 工具和协议的默认密钥类型。
- DSA：这是另一种早期的 SSH 密钥类型，使用 DSA 加密算法。DSA 密钥已被广泛使用，但现在已不建议使用。
- ECDSA：这是一种基于椭圆曲线加密算法的 SSH 密钥类型，通常比 RSA 和 DSA 密钥更安全和高效。
- ed25519：这是一种基于椭圆曲线加密算法的公钥加密方案，它被广泛应用于 SSH 密钥认证。ed25519 密钥具有更高的安全性和更好的性能，因此在许多情况下被认为是最佳选择。
- ECIES：这是一种基于椭圆曲线加密算法的加密方案，可以在 SSH 中使用。ECIES 密钥通常用于加密和解密敏感数据。

### 将公钥放到服务器中

当在客户端当前用户生成完公钥秘钥对之后，需要将公钥放到服务器中的用户中（默认需要放到用户的~/.ssh/authorized_keys文件中），从而完成免密登录。

有两种方法：

直接使用 `ssh-copy-id` 命令即可，如下图中，将客户端中`ubuntu`这个用户的一个公钥复制到服务器`user2`的authorized_keys中，然后就可以免密登录了：

![@七禾页话][10]

Note：在SSH 2协议中，ssh命令默认会读取~/.ssh/id_dsa, ~/.ssh/id_ecdsa, ~/.ssh/id_ed25519 和 ~/.ssh/id_rsa这几个密钥文件。

如果需要指定某一组生成的公钥，需要通过 `-i` 参数指定私钥，那么对应的公钥就会复制到服务器中：

![@七禾页话][11]

另外一种方案就是直接打印出公钥后复制到服务器用户的`~/.ssh/authorized_keys`中：

```
###左右滑动
###在客户端中：
cat <公钥>， 比如：
cat .ssh/id_secure.pub 

###在服务器中，使用vi编辑~/.ssh/authorized_keys，或者使用echo + >>，一定要注意是 >> 追加到文件中：
echo "公钥的内容" >> ~/.ssh/authorized_keys 

```

具体操作如下图：

![@七禾页话][12]

到此就完成了从客户端到服务器的免密登录，需要注意的是客户端使用的必须是生成公钥秘钥对的用户，比如上述例子中客户端我使用的是ubuntu的这个用户，也就是说只有在ubuntu这个用户下登录服务器才可以免密登录，然后就是服务器的用户需要是防止了客户端公钥的用户，比如我的例子中是将客户端ubuntu用户的公钥放到了服务器user2这个用户的`authorized_keys`文件中，那么只有如下的ssh命令才可以免密：

![@七禾页话][13]

到这里通过图例梳理下免密登录的需求和过程：

![@七禾页话][14]

最后就是免密登录中需要额外注意的问题和排错建议：

1. 服务器上.ssh目录的权限必须是700。
2. 服务器上.authorized_keys文件权限必须是600或者644。
3. 服务器上用户家目录文件权限必须是700（用户目录权限也以可为755，750，就是不能是77x。），比如用户名是user2，则/home/user2这个目录权限必须是700，如果遇到以上三个权限在客户端上查看/var/log/secure文件会看到”bad ownershop or modes for ...“的错误。
4. 服务器上SELinux关闭为disabled，可以使用命令修改setenforce 0 ，查看状态的命令为getenforce或者查看/etc/selinux/config 文件中是否是disabled。
5. 免密失败有可能是StrictModes问题，在服务器中编辑/etc/ssh/sshd_config，找到#StrictModes yes改成StrictModes no。
6. 有可能是PubkeyAuthentication问题，在服务器中编辑/etc/ssh/sshd_config，找到PubkeyAuthentication改成yes。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs5Jgia6eXnwgxa3jrl2xFQibMBJDnq3M8XDpMicgSlnwe5jm84Fx7UVzlTA/640?wx_fmt=jpeg&from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs5UDib4NPs8VOl123aopm4BTGzO9rVu5eBMavKEmnTZojFQepVic9CJFsA/640?wx_fmt=png&from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs5W5Q5HOLkHLDeMS4Asiby7UZnv76rNLZCwBR6wpyRGJjKgEBAc8TFA2w/640?wx_fmt=png&from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs54ukGOmvmk4TNavIOdn85IDfMPF3UZO3tibo7GKE1oOpke8S2TiaRE1yA/640?wx_fmt=png&from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs5RBMo7eo5qeRFf5GkkY1IXxs0NcW8ibPpoic1icKAxV2amFSe5CvgPLbBA/640?wx_fmt=png&from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs506doDZrKIL22zfjqJg9vGjgyia6EF4DA3LkgbAu2qFib27sdsbzJLfuQ/640?wx_fmt=png&from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs5v0rpqk9vxEK3m9afJEoMjpQv0ciaeAM5icYlHqsYnM8ibmNiaSUWcWv8Hg/640?wx_fmt=png&from=appmsg


[8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs5kVt8v4Adiak41x0tfy9KVTicOAE41PWDZoS2K9jibicU7fKQibf33npFuOw/640?wx_fmt=png&from=appmsg


[9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs56X39ibuWSkQpc15iavDpohicWpZzPgnwC2BU2FH2woYVhYs1ERI8jIvug/640?wx_fmt=png&from=appmsg


[10]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs5VeKSgRkZjoC9L5EQib4arrgbFOauu7IbfuvaJjsFLFgBtmU1t1y3VmQ/640?wx_fmt=png&from=appmsg


[11]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs54qmGMia4xuWvY8nwJIFVia8HdKHEbLPfIvJ5uiaDlMSj78ic4RD2cOMJDQ/640?wx_fmt=png&from=appmsg


[12]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs51kPm7Tqr7BgDZAf6HCxGgf8GCzeGCh5hWU2lIlcTeylsA8TFsZT5gg/640?wx_fmt=png&from=appmsg


[13]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs5uFbE96Jm0XasEN0Q2U7LUFQavkQX46icaPHlB5rXgiaxtgHdIKYpREKA/640?wx_fmt=png&from=appmsg


[14]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC1DD2u2jD5pe47Ns1AkSs5vokQZZ5djjFicFNscW77UU154gmJvR90HhfOicpjicHvmh23gPSvlrG5A/640?wx_fmt=png&from=appmsg

