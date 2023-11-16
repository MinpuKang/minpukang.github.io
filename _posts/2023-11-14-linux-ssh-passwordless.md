---
layout:     post
title:      Linux｜免密登录又又又失败了？
date:       2023-11-14
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
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

![@七禾页话][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

因为某某原因，又到了需要搞免密登录的时候。

虽然自认为免密就是创建一套ssh key pair，然后把public key交给对方放到其authorized_keys里就可以了。

结果免密登录又又又失败了！！！

在ssh debug的模式下看到使用public key登录的时候server回复了type 51的错误：

```
###左右滑动
$ ssh -vvv user@10.10.10.10
.....
debug1: Next authentication method: publickey
debug1: Offering public key: /home/user/.ssh/id_rsa RSA SHA256:6FXZFw9asi0MiG2qHOnraHVYoVWq09zq/FwR+rOtrz4
debug3: send packet: type 50
debug2: we sent a publickey packet, wait for reply
debug3: receive packet: type 51
.....
```

然后在server端sshd_config里开启debug的模式，监控/var/log/messages发现了具体的原因：
```
###左右滑动
# vi /etc/ssh/sshd_config
SyslogFacility AUTH
LogLevel DEBUG

#修改后保存，然后重启sshd服务
# systemctl restart sshd

#在Server上监控log，再从client上进行ssh登录
# tail -f /var/log/messages
....
Authentication refused: bad ownership or modes for directory /home/user
....
```

“Authentication refused: bad ownership or modes for directory /home/user”这个错误就是/home/user这个文件夹是属于错误的用户和用户组，或者某些权限有问题，查看这个文件夹权限，如下：
```
###左右滑动
[user@localhost ~]$ ls -ld /home/user/
drwxrwxr-x 10 user user 322 Nov 13 11:12 /home/user/
```

可以看到文件夹/home/user属于用户user和用户组user，所以所属是没有问题的。

再看权限是rwxrwxr-x，也就是775，问题就在用户组的权限上，Linux下默认的家目录权限应该是755，也就是drwxr-xr-x。

找到了原因，解决方案就很简单了，直接chmod 755修改权限即可：
```
###左右滑动
[root@localhost /]# chmod 755 /home/user
[root@localhost /]# cd /home/
[root@localhost home]# ls -l
total 0
drwxr-xr-x 10 user user 322 Nov 13 11:12 user
[root@localhost home]# 
```

为什么会出现这个问题呢？原因就是创建用户的时候，提前创建了家目录/home/user，然后通过useradd之后没有能修改其家目录的所属以及相应的权限。

通过上述修改后，ssh成功免密登录到了server，使用debug可以看到"debug3: receive packet: type 60"的log：
```
###左右滑动
debug1: Next authentication method: publickey
debug1: Offering public key: /home/user/.ssh/id_rsa RSA SHA256:6FXZFw9asi0MiG2qHOnraHVYoVWq09zq/FwR+rOtrz4
debug3: send packet: type 50
debug2: we sent a publickey packet, wait for reply
debug3: receive packet: type 60
debug1: Server accepts key: /home/user/.ssh/id_rsa RSA SHA256:6FXZFw9asi0MiG2qHOnraHVYoVWq09zq/FwR+rOtrz4
debug3: sign_and_send_pubkey: RSA SHA256:6FXZFw9asi0MiG2qHOnraHVYoVWq09zq/FwR+rOtrz4
debug3: sign_and_send_pubkey: signing using rsa-sha2-256
```

综上，在免密登录的设定下，server上各个文件夹和文件有相对严格的权限和所属要求，罗列如下：
```
###左右滑动
# /home文件夹的权限是755，所属于root的用户和用户组
[user@localhost ~]$ ls -ld /home/
drwxr-xr-x 3 root root 28 Nov  1 07:48 /home/

# 用户家目录的权限是755，所属于用户自己和用户所在的组
[user@localhost ~]$ ls -ld /home/user/
drwxr-xr-x 10 user user 322 Nov 13 11:12 /home/user/

# .ssh这个文件夹权限是700，所属于用户自己和用户所在的组
[user@localhost ~]$ ls -ld /home/user/.ssh/
drwx------ 2 user user 100 Nov 13 11:13 /home/user/.ssh/

# .ssh文件夹下的文件所属于用户自己和用户所在的组，具体权限是authorized_keys和id_rsa是600，id_rsa.pub和known_hosts是640
[user@localhost ~]$ ls -l /home/user/.ssh/
total 16
-rw------- 1 user user  582 Nov 13 11:13 authorized_keys
-rw------- 1 user user 2622 Nov 13 11:13 id_rsa
-rw-r----- 1 user user  582 Nov 13 11:13 id_rsa.pub
-rw-r----- 1 user user  172 Nov 13 11:13 known_hosts
[user@localhost ~]$
```

权限数字是怎么算的？移步[Linux｜关于chmod的那些事儿](https://mp.weixin.qq.com/s/kjGmwoJkE6SKI6Ab_NMoCA)

那么如何做免密登录呢？

首先区分一下ssh的client和server，如图所示，目前用户所在的主机就是client，ssh的IP所属的服务器就是Server，对于Server的要求就是要开启sshd的服务，ssh所使用的用户user1需要在Server上创建好:
![@七禾页话][2]

然后需要在client查看用户下是否已经有ssh的公钥私钥对了，查看的原因是如果已经创建过了，那么就可以直接用，否则再次创建可能导致使用之前的公钥私钥对的任务失败，所以一定要先查看确定没有之后才使用ssh-keygen的命令创建一组ssh的公钥私钥对：
```
###左右滑动
## 切换到家目录
<client userabc>@hostname ~$ cd ~

## 查看是否生成过公钥私钥对
<client userabc>@hostname ~$ ls -l .ssh/

## 如果没有才可以使用ssh-keygen创建公钥私钥对
<client userabc>@hostname ~$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/<client userabc>/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/<client userabc>/.ssh/id_rsa
Your public key has been saved in /home/<client userabc>/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:52dVJoqzNGM7F85518gzHUcTKFugsBTU8wwMruholm8 <client userabc>@hostname
The key's randomart image is:
+---[RSA 3072]----+
|      .*=  .. .. |
|      o o=.. o  .|
|       o .= + ..+|
|    . .    = . =.|
|   . .  S O o ...|
|  +      = O = o+|
| = .      = B * +|
|o .E       = . + |
|  ..             |
+----[SHA256]-----+
```

最后将client用户下生产的ssh公钥内容传到server用于ssh登录的用户家目录下的.ssh/authorized_keys文件中，可以使用ssh-copy-id的命令：
```
###左右滑动
<client userabc>@hostname ~$ ssh-copy-id <server user>@<server ip>
Password: <password for server user>

# 比如上图中将client用户的public key放到server中：
<client userabc>@hostname ~$ ssh-copy-id user1@10.10.10.10
Password: <password of server user1>
```

公钥放到Server后，在当前client生成ssh公钥密钥对的用户session下就可以免密登录到Server了，如果不行的话，一个是看sever上各种文件夹和文件的权限，另外就要看server上/etc/ssh/sshd_config的配置，比如PubkeyAuthentication等参数。

还不行的话，哈哈，关注我留言来一起看看吧！

思考一下，自己对自己能不能免密呢？

以上！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eD0NDokyhCiaFfW0pLpojUx2bysicepcLzyVBHYwsn4WUA5JTNxGLMVdP60W3yLovOuFX1iczT9YAxPg/640?wx_fmt=jpeg

[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDQN5KcQUmEU9wQWeOAdJcI9zEHEHzQ6WqRVsHoHIAlbnIeljVqVECdyDibibBvA5JbgT8libWtNPlbQ/640?wx_fmt=png



