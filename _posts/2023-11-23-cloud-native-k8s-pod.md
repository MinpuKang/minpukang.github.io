---
layout:     post
title:      云原生｜什么是Kubernetes最小单元Pod？
date:       2023-11-23
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - CLoud Native
    - 云原生
    - K8s
    - POD
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

前一篇`云原生｜关于K8s中Ingress和Egress流量的那些事儿`是一个方案，针对Ingress流量响应的方案，里边涉及到了Linux的知识点和Kubernetes的知识点。

所以打算没事儿也聊聊自己对Kubernetes一些概念的了解，正好就当自己的笔记了。

一切都有从一个豆荚开始说起，豆角（豆荚）是藤蔓上成熟的果实，如果它破裂了，豆子就洒满一地，没有了跟滕蔓衔接的桥梁了，所以豆荚是藤蔓上最小的果实单元。

POD的中文含义就是豆荚，非常形象，如同豆角之于藤蔓一样，POD也是Kubernetes这个藤蔓上的最小单元，而将POD剥开看到的是一个个Container(容器)就如同剥开豆荚看到的一颗颗豆子。
![@七禾页话][2]


要了解POD，那就需要先了解下容器。容器是一个独立的环境，其中打包了应用程序以及其依赖环境。每个容器都有一个IP地址，它可以申请外部存储并且调用CPU和Memory系统资源等。

Kubernetes是一个通过自己的运行方式部署、扩展和管理容器化应用的编排系统，这个运行方式就是POD，所以POD是Kubernetes中的最小单元，一个POD可以包含一个或者多个容器，所以可以将POD看做是一个容纳了一个或者多个容器的盒子，但是这个盒子里至少有一个容器，不能为空。

POD的概念是抽象的，它将一个或者多个容器作为一个单元进行管理，POD所提供的功能完全看它里边的容器，所以可以说POD是一个小家庭，家庭里的各个成员是容器，各自来完成预定义的工作，然后通过家庭这个概念来融入社会这个集群。

一个POD可以从集群中获取唯一的IP，当然这个IP是动态的，当POD存在的时候，它在一个集群内部拥有唯一的IP，一旦POD消失，它所拥有的IP就被集群回收再利用了，之前说容器之间沟通也是依赖于IP，所以在Kubernetes中一个POD内部的容器使用localhost+不同端口来互相沟通，如果POD1中的容器想跟POD2中的容器通信，理论上就是两个POD之间的通信，直接使用POD的IP地址。
![@七禾页话][3]

因此，一个POD内的容器是共享如下内容的：
- Network Namepace（网络命名空间）: 所有容器共享localhost。
- IPC namespace（进程间通信命名空间）: 所有容器共享IPC。
- UTS Namespace（主机名和域名命名空间）: 所有的容器共享一个hostname。

但是一个POD内的容器也必然要互相区分的：
- 默认情况下，PID 命名空间 不共享，但 kubernetes 提供了选项，可使用 在 Pod 内的容器之间启用进程共享shareProcessNamespace 选项。
- 存储不共享，每一个容器都拥有他们自己的文件系统和目录，但是是通过POD统一申请一或多个Volume。


简而言之地概括一下：
- Pod 是 Kubernetes 中最小的可部署单元。
- Pod 本质上是短暂的；它们可以被创建、删除和更新。
- 一个 Pod 可以有多个容器；一个 Pod 内可以运行的容器数量没有限制。
- 每个 Pod 都有唯一的 IP 地址。
- Pod 使用 IP 地址相互通信。
- Pod 内的容器使用localhost+不同的端口进行连接。
- Pod 内运行的容器应具有不同的端口号，以避免端口冲突。
- 可以为 Pod 内运行的每个容器设置 CPU 和内存资源。
- Pod 内的容器共享相同的卷挂载。
- Pod 内的所有容器都必须在同一个节点上；它不能跨越多个节点。
- 如果有多个容器，则在 Pod 启动期间，所有主容器都会并行启动。而 pod 内的 init 容器按顺序运行。

简单了解了POD之后，那么如何定义和部署一个POD呢？

POD是Kubernetes原生的概念，需要通过yaml对其声明，然后可以直接使用kubectl来创建。

以下是创建NginxWeb服务器Pod的PodYAML的简单示例：
```yaml
##左右滑动
apiVersion: v1
kind: Pod
metadata:
  name: web-server-pod
  labels:
    app: web-server
    environment: production
  annotations:
    description: This pod runs the web server
spec:
  containers:
  - name: web-server
    image: nginx:latest
    ports:
    - containerPort: 80
```


| 参数         | 含义        |
| ----------- | ----------- |
| apiVersion  | 定义POD的api版本，不同k8s版本导致api版本不一 ,可以通过`kubectl api-resources`查看      |
| kind   | 类型，K8s的资源类型，可以通过`kubectl api-resources`查看        |
| metadata | metadata用于唯一标识和描述POD，包括POD的名字、所属的namespace、注释（annotations）以及Label（标签）等|
| spec | 在spec下定义POD所需要状态内容，比如对Container的描述，包括了容器的名字，镜像，对外开放的端口，容器的资源等等内容|

这是非常小的一个例子，一个POD的yaml下有很多可以定义的参数，对于开发者来说需要查看官方资料来定义自己产品需要的参数。

那么如何创建Pod呢？当使用yaml的时候非常简单，直接使用`kubectl apply/create -f <POD的yaml文件名>`即可。

当然也可以将yaml的内容放到kubectl里来创建Pod：
```bash
@@左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod
No resources found in default namespace.
ubuntu@VM-16-3-ubuntu:~$ sudo docker image list
REPOSITORY                           TAG       IMAGE ID       CREATED        SIZE
nginx                                1.25.0    7d3c40f240e1   5 months ago   143MB
ubuntu@VM-16-3-ubuntu:~$ kubectl run web-server-pod \
  --image=nginx:1.25.0 \
  --restart=Never \
  --port=80 \
  --labels=app=web-server,environment=production \
  --annotations description="This pod runs the web server"
pod/web-server-pod created
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod
NAME             READY   STATUS              RESTARTS   AGE
web-server-pod   0/1     ContainerCreating   0          3s
ubuntu@VM-16-3-ubuntu:~$ 
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod web-server-pod -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE             NOMINATED NODE   READINESS GATES
web-server-pod   1/1     Running   0          52s   10.42.0.111   vm-16-3-ubuntu   <none>           <none>
ubuntu@VM-16-3-ubuntu:~$ 
```

如果想了解已经创建的POD的详细信息，可以使用`kubectl describe pod <pod name> -n <pod所在的namespace>`来查看：
![@七禾页话][4]

下图是一张由`describe`命令显示POD重要信息的关系图：

![@七禾页话][5]

如何删除POD呢？

如果是使用`kubectl run`的命令创建的POD，需要用`kubectl delete pod <pod name> -n <namespace>`来删除:
```bash
@@左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
web-server-pod   1/1     Running   0          11m
ubuntu@VM-16-3-ubuntu:~$ kubectl delete pod web-server-pod
pod "web-server-pod" deleted
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod
No resources found in default namespace.
ubuntu@VM-16-3-ubuntu:~$ 
```

如果是使用`kubectl apply/create -f <POD的yaml文件名>`创建的POD，可以使用`kubectl delete pod <pod name> -n <namespace>`或者`kubectl delete -f <POD的yaml文件名>`删除。

在使用yaml创建POD的时候很难记住所有的参数，查阅官网很多时候也是大海捞针，那么怎么办呢？Kubernetes已经想到并给出了解决方案，就是使用`–dry-run`参数来创建yaml：
```bash
@@左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl run nginx-pod --image=nginx:1.25.0 --dry-run=client -o yaml > nginx-pod.yaml
ubuntu@VM-16-3-ubuntu:~$ ls -l nginx-pod.yaml 
-rw-rw-r-- 1 ubuntu ubuntu 251 Nov 19 21:06 nginx-pod.yaml
ubuntu@VM-16-3-ubuntu:~$ cat nginx-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-pod
  name: nginx-pod
spec:
  containers:
  - image: nginx:1.25.0
    name: nginx-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

关于POD，还需要了解一个重要的概念就是它的生命周期，一个POD通常都有控制管理器，比如ReplicaSet、Deployment、DaemonSets等，单独创建一个Pod的时候是不受任何管理器管理的，不管是哪种情况，POD都要经历不同的生命周期阶段：
| **状态**               | **解释**                                                                                                   |
|-----------------------|------------------------------------------------------------------------------------------------------------------------------------|
| Pending(挂起)         | 在执行创建 Pod 过程中，命令行已经执行，Pod 已经被 Kubernetes 系统接受，但仍有一个或多个容器未被创建。可以通过 `kubectl describe` 查看处于 Pending 状态的原因。                                      |
| Running(运行中)       | Pod 已经被绑定到一个节点上，并且所有的容器都已经被创建，至少有一个是运行状态，或者是正在启动或者重启。可以通过 `kubectl logs` 查看 Pod 的日志。                                            |
| Succeeded(成功)       | 所有容器执行成功并终止，并且不会再次重启。可以通过 `kubectl logs` 查看 Pod 的日志。                                              |
| Failed(失败)          | 至少有一个容器没有正常退出，以失败告终。在 Linux 上每个命令都有状态值和信号值，状态值正常是 0-255 之间，正常状态值为 0。容器的创建状态只要是非 0 就是异常的。可以通过 `kubectl logs` 查看具体原因。                  |
| Unknown(未知)         | 通常是通信出问题，不知道状态是什么。通常是 Unknown。                                                                              |
| ImagePullBackOffErr   | 镜像拉取失败，一般是由于镜像不存在、网络不通或者需要登录认证引起的。可以使用 `kubectl describe` 命令查看具体原因。                                                |
| CrashLoopBackOff      | 容器启动失败，有可能是镜像文件本身就有问题，不能正常启动。可以通过 `kubectl logs` 命令查看具体原因，一般为启动命令不正确，健康检查不通过等。                                             |
| OOMKilled             | 内存溢出，运行的容器本身出现内存溢出。一旦出现这种错误容器或者程序本身会自动 kill 掉。通常是内存 limit 设置太小。或者程序本身有问题，JVM 和容器内存限制都够用，还是内存溢出了。                                   |
| Terminating           | Pod 正在被删除，可以通过 `kubectl describe` 查看状态。                                                                           |
| SysctlForbidden       | 内核启动失败，和 Linux 内核相关。在启动 Pod 的时候加了一些内核的需求，但是没有开放需求，就会造成内核启动失败。                                                                                  |
| Completed(主进程退出) | 容器内部主进程退出，一般计划任务执行结束会显示该状态。                                                                          |
| ContainerCreating     | Pod 正在创建，一般为正在下载镜像，或者有配置不当的地方。可以通过 `kubectl describe` 查看具体原因。                                 |
| Initializing(初始化)    | Pod 中包含了初始化容器，这些容器正在执行初始化任务。                                                        |


既然POD有状态，那么POD内的容器也有它的状态：
在 Kubernetes 中，Pod 内的容器有不同的状态，这些状态反映了容器的生命周期和运行状况。以下是一些常见的容器状态及其解释，制作成表格形式：

| **状态**       | **解释**                                                                                                   |
|---------------|------------------------------------------------------------------------------------------------------------|
| **Running**    | 容器正在正常运行中。                                                                                       |
| **Terminated** | 容器已经退出，并且可能处于成功或失败的状态。                                                              |
| **Waiting**    | 容器正在等待某些条件满足，例如依赖的容器尚未启动，或者容器正在等待调度资源。                             |
| **Pending**    | Pod 已经被创建，但容器的镜像正在被拉取，或者容器正在等待被调度到节点上运行。                              |
| **ImagePullBackOff** | 容器尝试拉取镜像失败，并且 Kubernetes 将在一段时间后进行重试。                                        |
| **ErrImagePull**    | 容器无法拉取指定的镜像。通常是由于镜像不存在或者拉取时发生错误导致的。                                  |
| **CrashLoopBackOff** | 容器已经崩溃，并且 Kubernetes 将在一段时间后进行重试。通常是由于容器崩溃导致的，然后容器被重新启动。      |
| **Init:Error**       | Init 容器初始化失败。这是在使用 Init 容器时，Init 容器未能成功执行导致的状态。                              |
| **Init:CrashLoopBackOff** | Init 容器已经崩溃，并且 Kubernetes 将在一段时间后进行重试。通常是由于 Init 容器崩溃导致的，然后容器被重新启动。 |

如何查看POD或者容器的状态呢？依然是使用`kubectl`命令中的`get`和`describe`：

`kubectl get pod`打印中的STATUS是POD的状态，READY标识了POD中已经Ready的容器个数和容器的个数的比例，即`已经Ready的容器个数/容器的个数`，Ready的容器表示其可以对外提供服务了，所以Ready的容器肯定是Running的状态了，但是Running状态的容器不一定是Ready的。
```bash
@@左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod 
NAME             READY   STATUS    RESTARTS   AGE
web-server-pod   1/1     Running   0          6s
ubuntu@VM-16-3-ubuntu:~$ 
```

通过`kubectl describe pod`可以详细获取POD的状态和各个容器的状态：
![@七禾页话][6]

那么针对POD和容器的不同状态如何排错呢？如果一个POD没有Running，用`kubectl describe pod`来查看POD的Event进行排错；如果一个POD已经Running了，但是有容器没有Ready，就需要使用`kubectl logs -n <namespace> <pod name> <container name in this pod>`检查容器启动中的log来拍错。

随着时间的推移，POD的Event和容器的log是会被覆盖掉的，因此在Event和log里没有有用的信息下，可以通过`kubectl delete pod -n <namespace> <pod name>`删除POD以触发POD重建（使用yaml创建的前提下）来获取最初的Event和log排错。

如何访问POD中的应用程序呢？`kubectl`提供了`port-forward`命令，从而实现对集群内部正在运行中的POD的访问。
```bash
@@左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl port-forward pod/web-server-pod 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080     ---->运行curl所打印的log

@@再开一个session，通过curl可以访问POD内部的Nginx服务：
ubuntu@VM-16-3-ubuntu:~$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
ubuntu@VM-16-3-ubuntu:~$ 
```

某些POD中的容器也开放了shell的访问权限，因此可以通过`kubectl exec`的命令进入POD中的某个容器，格式是`kubectl exec -it -n <namespace> <pod name> -c <container name> -- <shell in container>`，如果POD只有一个容器可以不用制定容器名；如果是多个容器，在不指定容器名的时候，访问的就是Annotation中kubectl.kubernetes.io/default-container的容器或者POD中的第一个容器：
```bash
@@左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
web-server-pod   1/1     Running   0          36m
ubuntu@VM-16-3-ubuntu:~$ kubectl exec -it web-server-pod web-server-pod -- bash
root@web-server-pod:/# hostname
web-server-pod
root@web-server-pod:/# exit
exit
ubuntu@VM-16-3-ubuntu:~$ 
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod -n harbor harbor-registry-5ff45f8998-5gh99 
NAME                               READY   STATUS    RESTARTS   AGE
harbor-registry-5ff45f8998-5gh99   2/2     Running   0          7d1h
ubuntu@VM-16-3-ubuntu:~$ kubectl exec -it -n harbor harbor-registry-5ff45f8998-5gh99 -- bash
Defaulted container "registry" out of: registry, registryctl
harbor [ / ]$ hostname
harbor-registry-5ff45f8998-5gh99
harbor [ / ]$ exit
exit
ubuntu@VM-16-3-ubuntu:~$ 
ubuntu@VM-16-3-ubuntu:~$ kubectl exec -it -n harbor harbor-registry-5ff45f8998-5gh99 -c registryctl -- bash
harbor [ / ]$ hostname
harbor-registry-5ff45f8998-5gh99
harbor [ / ]$ exit
exit
ubuntu@VM-16-3-ubuntu:~$ 
```

小知识点，如何定义Annotation中kubectl.kubernetes.io/default-container？首先需要知道POD下路由的容器名，然后通过`kubectl annotate`命令定义，例子如下：
```bash
@@左右滑动
ubuntu@VM-16-3-ubuntu:~$ kubectl annotate pods -n harbor harbor-registry-5ff45f8998-5gh99  kubectl.kubernetes.io/default-container=registryctl
pod/harbor-registry-5ff45f8998-5gh99 annotate
ubuntu@VM-16-3-ubuntu:~$ kubectl describe pod -n harbor harbor-registry-5ff45f8998-5gh99 | grep -i default-container
                  kubectl.kubernetes.io/default-container: registryctl
ubuntu@VM-16-3-ubuntu:~$ kubectl exec -it -n harbor harbor-registry-5ff45f8998-5gh99  -- bash
harbor [ / ]$ cd ~
harbor [ ~ ]$ ls
ca-bundle.crt.original  harbor_registryctl  install_cert.sh  start.sh
harbor [ ~ ]$ exit
exit
ubuntu@VM-16-3-ubuntu:~$ 
```

上述例子中的nginx POD配置非常上，但是POD的功能确实非常多的，用于资源管理、配置、安全定义等等，同时POD相关的还有许多功能，列举如下：

以下是与 Pod 相关的关键功能。

| 功能                           | 描述                                               |
| ----------------------------- | --------------------------------------------------- |
| Resource Requests and Limits  | Pod CPU/Memory 分配                                 |
| Labels                        | 附加到 Pod 的键值对，用于对资源进行分类                |
| Selectors                     | 基于标签对资源进行分组                                |
| Liveness, Readiness, and Startup Probes | 容器健康检查                                   |
| ConfigMaps                    | 用于配置管理                                          |
| Secrets                       | 用于管理一些如证书、用户密码等安全信息                                          |
| Volumes                       | 永久性的数据存储                                        |
| Init Containers               | 在主容器之前运行的启动容器                                 |
| Ephemeral Containers          | 用于调试或故障排除目的添加到 Pod 的临时容器            |
| Service Account               | 限制对 Kubernetes 对象和资源的访问                      |
| SecurityContext               | 主机权限和特权                                      |
| Affinity and Anti-Affinity Rules | 在节点之间控制 Pod 的亲和反亲和规则                             |
| Pod Preemption & Priority     | 设置 Pod 调度的优先级                            |
| Pod Disruption Budget         | 在集群维护期间需要运行的最小Pod副本数，常用于集群维护和升级时                   |
| Container Life Cycle Hooks    | 根据 Pod 生命周期阶段更改执行自定义脚本                   |

这些功能涵盖了与 Kubernetes Pod 相关的一些关键方面，包括资源管理、标签和选择器、健康检查、配置管理、安全性等。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG1zp4MQgiaU7icMS2HgMONe5JA0R1kHoaEM9p1DwsmYQSKd9jBpBBRmrDA/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG1NuEA5wRmib3JCM3Y8ecB4vMCMdfziaTmc6ibiaXukJXZSAFNz57ME18z0g/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG1UiaqsEHnfFD3ibYL40f5IQJfjCdyJDq42SZCbmDhDoR2fX9bzRWo71JQ/640?wx_fmt=gif&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG139kYHibsJsgciaxjiaIvV43LeEmPB703DoPIsqX5ROgOS3grgT9k0UxJw/640?wx_fmt=png&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG1WCA4J46FsAQaSMWD53R6KdJ3Ft5ekzMic9dsUFRtfm2zEWicaABphLng/640?wx_fmt=gif&amp;from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eC2BiaJh04kZ7AZpMygaoWG1fXlKGUibiaSeURibGO4erS1np9m7sPoaoHpNxMrwNZpdIdpZUOJqSrLNA/640?wx_fmt=png&amp;from=appmsg

