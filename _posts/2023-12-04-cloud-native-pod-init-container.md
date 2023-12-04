---
layout:     post
title:      云原生｜别拿Init Container不当前菜
date:       2023-12-04
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - CLoud Native
    - 云原生
    - K8s
    - Kubernetes
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

在 Kubernetes 的Pod中，有一个小物件儿经常容易被忽略，因为它看起来非常不起眼，也不参与日常的对外服务工作，只是默默地在最初的时候闪光，然后就永远的沉睡下去直到Pod需要再次创建。

这个小物件儿，虽然不起眼，但是确在Pod的启动过程中有着决定性的作用，它就是Init Container。

从名字就可以看出它的作用，往大了说如同如盘古开天辟地一般开启了Pod的生命周期，往小了说如同安装系统中PXE Boot的引导，如果一个Pod需要预设某些东西，就必然要有Init Container的设定，从而使得在主容器启动之前执行额外的初始化逻辑成为可能。

## 什么是 Init Containers？

Init Containers 是 Pod 中一种特殊类型的容器，它的目的是在主容器启动之前执行一些初始化任务。Init Containers 的生命周期是独立于主容器的，只有在 Init Containers 完成并成功退出后，主容器才会启动。

在 Pod 的生命周期中，Init Containers 是按照从上到下的顺序逐个执行的。每个 Init Container 都必须成功完成，即退出状态码为 0，才能继续执行下一个 Init Container 或启动主容器。

## 为什么使用 Init Containers？

### 1. 依赖解决

有时，主容器启动之前可能需要一些额外的资源或依赖项。比如等待外部服务准备就绪、下载数据或配置文件等各种准备工作。使用 Init Containers 可以确保这些依赖项在主容器启动之前已经就绪。

### 2. 数据预处理

Init Containers 还可以用于预处理数据。例如，可能需要在主容器运行之前初始化数据库、解压缩文件等操作。Init Containers 提供了在主容器启动前执行这些任务的机制。

### 3. 并行初始化

由于 Init Containers 是按顺序执行的，你可以为不同的初始化任务使用不同的容器，使这些任务能够以并行方式执行。这可以加速整体启动过程。

## 示例演示

比如启动一个 Web 应用程序时，它依赖于一个后端数据库，因此可以使用 Init Containers 来确保数据库初始化完毕后，Web 应用程序才开始启动。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: my-webapp-image
  initContainers:
    - name: init-db
      image: database-initializer
      # 添加数据库初始化逻辑
  # 其他 Pod 配置...
```

这个yaml中的 `webapp` Pod 包含一个名为 `webapp` 的主容器和一个名为 `init-db` 的 Init Container。`init-db` 容器负责执行数据库初始化逻辑。只有在 `init-db` 完成初始化并成功退出后，`webapp` 主容器才会启动。

## 总结

Init Container提供了一种非常灵活的预设机制，在业务主容器启动前初始化了环境和准备了必要的前提条件，这在解决依赖关系、预处理数据和实现并行初始化方面都非常有用。

在troubleshooting的时候，如果一个POD一直没有Running，可以查看此Pod是否有Init Container并查看其状态，如果状态不是Ready的可以尝试通过`kubectl logs pod -n <namesapce> <pod name> -c <init container name>`来查看其启动的log从而定位问题。

别拿豆包不当干粮，小小的Init Container也有着大作用，千万不要忘记它！

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAYJ9EJ3fj2uXNCliamgQMe6lbKiczXicUAE3oFdaTk1JxRr5lqLCdON7ebX9jib7qxu9uWibiaMqcuNJkg/640?wx_fmt=jpeg&amp;from=appmsg


