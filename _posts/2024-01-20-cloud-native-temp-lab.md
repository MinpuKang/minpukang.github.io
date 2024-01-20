---
layout:     post
title:      云原生｜一个在线的K8s免费练习平台
date:       2024-01-20
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - CLoud
    - 云原生
    - K8s
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

技术的学习唯手熟尔，必须要在理论的基础上实际操作，才能加深印象，之前分享了使用[VMWare创建VM的方式安装K3s来做实验](https://mp.weixin.qq.com/s/oksViLEXgdNJ6zOKLCwSEQ)，最近发现了一个在线的k8s练习平台叫做Play with Kubernetes，是对于环境受限或者资源受限还想做K8s实验的福音。

## Play with Kubernetes 介绍

Play with Kubernetes是Docker通过使用Docker-in-Docker(DinD)技术模拟了多个虚拟机的效果，从而提供了一个在浏览器中免费使用CentOS Linux虚拟机的接口和体验平台，Play with Kubernetes有如下几个优缺点：

#### 优点
- 免费体验：每次登录后都有4个小时的体验时间，可以做想做的实验;
- K8s环境使用kubeadm直接部署（使用 weave 网络）;
- 平台共提供了5台Centos7设备供我们使用（docker版本为24.0.2），也就是可以创建5个虚拟node；
- K8s也是比较新的版本1.27.2版本；
- 直接联网，需要apply的yaml或者下载的镜像是直接从官网下载的，而且是外网。

#### 缺点
- 只能使用github或者docker账户做oauth验证，所以要首先拥有docker或者github账户；
- 每次4小时的Session过期后，再登录之后又要从0开始搭建k8s；
- 因为是浏览器中使用，所有交互不是很好，比如Windows下的复制粘贴不是正常的Ctrl+C和Ctrl+V了（后边会介绍如何复制粘贴）；
- 最多只能创建5个虚拟机，也就是只能搭建一个master+4 worker的5node的集群，当然对于体验来说也够用了。

总体来说，Play with Kubernetes是一个很好的免费体验平台，对于日常学习、理解和实践基础概念是完全够用了。

下边介绍下如何使用。

## 如何复制粘贴

工欲善其事必先利其器，之前说了使用过程中不能用正常的Ctrl+C和Ctrl+V来复制粘贴了，所以先介绍下如何复制粘贴。

![@七禾页话][2]

#### MAC电脑
登录后，如果是Mac电脑，Setting中可以看到会自动识别出来，如果没有识别出来，可以选中“Mac OSX”，复制粘贴键如下：
- 复制： Command键+C
- 粘贴： Command键+V

#### Windows电脑
如果不是Mac电脑，是Windows系统，那么登录后，Setting中识别的是None，这个时候的复制粘贴如下：

带Fn键的，需要开启Fn功能，也就是Fn键上的小灯是亮的：
- Ctrl键+Fn键+Insert键(F10键)
- Shift键+Fn键+Insert键(F10键)

Fn功能的开启先按fn然后迅速按左侧shift键，即可开启fn（功能）模式：

![@七禾页话][3]

如果不带Fn键的：
- Ctrl键+Insert键
- Shift键+Insert键

## 如何使用Play with Kubernetes

接下来就是正文如何使用Play with Kubernetes了。

打开网站[https://labs.play-with-k8s.com/](https://labs.play-with-k8s.com/)，选择使用`Github`或者`Docker`后跳转进行Oauth验证登录。

![@七禾页话][4]

#### 创建Instance

登录后首先要`Add New Instance`来创建Node，可以根据自己的需要创建多个node，最多5个：

![@七禾页话][5]

![@七禾页话][6]


#### 初始化集群的Master节点

创建完Node后，可以看到Warning提示初始化Master的过程，一共三部曲，其中必做的是头两步：
1. 初始化K8s的Master节点；
2. 初始化K8s的网络；

初始化K8s的Master节点命令如下：
```bash
#### 左右滑动，不需要任何修改
kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16
```

Master节点启动过程如下：
![@七禾页话][7]

需要注意的在上图中已经标注，此时Node的状态是NotReady，而且coreDNS的POD也是pending的状态，原因是没有网络插件，所以需要运行第二步：初始化K8s的网络，命令如下：
```bash
#### 左右滑动，不需要任何修改
kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
```

安装结果如下，安装网络插件后，等一会儿Node就会Ready，所有的POD也都会Ready：
![@七禾页话][8]

#### 注册Worker到Master

如果已经创建了其他的node，直接在master以外的node上运行如下的命令，此命令是安装K8s master节点log中的`join`命令，其中token每次都会变，所以不要直接copy下边的例子：
```bash
#### 左右滑动
kubeadm join 192.168.0.18:6443 --token 1g5gfn.agqd0wf4cbv2hbyf \
        --discovery-token-ca-cert-hash sha256:eac1d72af79f6b3da5e69ee44cd1e3f46c3de52f90f4ef448655daefadf5669d 
```

过程如下：
![@七禾页话][9]

然后回到Master节点运行`kubectl get node`可以看到node2已经加入到集群了：
![@七禾页话][10]

但是会看到worker的role是none，为了很清晰明了的知道各个node的角色，可以打label：
```bash
###左右滑动
kubectl label nodes <node name> kubernetes.io/role=worker

或者for循环
for i in `kubectl get nodes -o wide --no-headers | grep -iv "control-plane" | awk '{print $1}'`;do kubectl label nodes $i kubernetes.io/role=worker;done
```

效果如下：
![@七禾页话][11]

可以继续将第二个worker节点加入到集群中，到此一个Master加两个Worker的小K8s集群创建完成了。

![@七禾页话][12]

#### 实例化一个Nginx

Ngnix作为云原生界的`hello world`常常被用来验证集群是否好用，所以我们也继续用Ngnix来验证Cluster的基础功能是否好用：

```bash
####左右滑动
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/nginx-app.yaml
```

结果如下：
![@七禾页话][13]

查看service之后可以使用`curl cluster-ip`来确认nginx的业务是否好用，如果看到如下的`Thank you for using nginx`代表nginx业务是正常的：
![@七禾页话][14]

以上就是使用`Play with Kubernetes`的过程，总体使用下来除了复制粘贴比较麻烦，偶尔会有卡顿，整体使用下来还是很好的，对于初次接触K8s或者做一些简单验证学习是很好的一个环境，当然一次性4个小时，如果想要验证的没有做完，那就要重来了，需要数秒刷副本！

欢迎留言来一起了解学习ICT的相关知识！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCwTrHQD1lMZ9IJymibAPfZtCEB0B9c4sA2RdXYehyOEzADZfsYezDXziboqH1F7xCX4pDG3EThrvrA/640?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmprUJRsEawibQjydqM2DuhVPj19G73uKAPlmCE5UicFroKibvf4HdeqTfONA/640?wx_fmt=gif&from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmpr1MRUuM0du54IYdUJIjulyBEEsgxzC6V93HcYkSQ2yORznZSoRSH7wQ/640?wx_fmt=png&from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmprLpFxOghIvdTWU8BvWOLOrHBqLwwHOLNQmSGK2eNXwUE3lFQnibXpnQQ/640?wx_fmt=png&from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmprqjuc6KgkvicicUfq0YrBqbhZgVcmLDtXl3JW6BJMnQhjXcetibdGjicOzw/640?wx_fmt=png&from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmprUvMdx3CqXGlpBK8yyIBOVWn16mmNPeowBxfn6O9ckyrhicT6rv5bpmA/640?wx_fmt=gif&from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmpr7hE5iaxnkyJH7PibaafGiakbIu7apOjrBFY7kMY3brNKfNoAI0Z5qUtMg/640?wx_fmt=png&from=appmsg


[8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmprshRnVNtkTxSLYlDqx8YAsKMYnY1pOLsLhN7icXrXW97nj2t3cqBscdw/640?wx_fmt=png&from=appmsg


[9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmprIU1YlarrWIIPzd6qmOHdmAF5dZZHBnQudOQ5pWomAmss1LAvflIgvQ/640?wx_fmt=png&from=appmsg


[10]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmprsJibV2pMIXfBBlCgYFnFWtquCPRMCUgyicp1FzUtj94tkiagBrGfGgNXg/640?wx_fmt=png&from=appmsg


[11]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmprSuEEdLaGYH2BibdiczP3lDiaLOdXuFrEDrX9aiaVZaFVEBdkqF8oy4bsmA/640?wx_fmt=png&from=appmsg


[12]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmprHoelxhKIvbvq9geWNdLF95lXZUO00XgCiaxrNzHuWxUfald3rHs5dmg/640?wx_fmt=png&amp;from=appmsg


[13]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmprF9h96yFhLibEu0Oibrr0KxeUE8uY2PrRDI1eTicCNQR8leG1NqItq2KKA/640?wx_fmt=png&amp;from=appmsg


[14]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXunyiaOkWbkxCPibotyfmprXb0FujJ4AnucK4Qh9TmnEF4phE0M4va0v2mZnYj9ejx2R0qZ8Z0Vvw/640?wx_fmt=png&amp;from=appmsg