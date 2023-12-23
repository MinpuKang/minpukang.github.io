---
layout:     post
title:      云原生｜攒几个好用的kubectl命令集
date:       2023-12-13
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

干活实用篇！

当云和云原生开始之后，我们需要查询各种各样的状态、log，尤其是云原生之后，一个NF中包含了若干的微服务，每个微服务又有若干个Pod，使用单一的`kubectl`指令一个一个的检索查看费时费力，但是当搭配Linux命令或者Python的一些模块后，效率会相对加快不好。

这里整理了一下我经常使用的一些命令合集，其中`for`循环、`grep`、`awk`使用的非常多。

#### 查看一个Namespace下的POD是否都Ready

在创建一个CNF的过程中，POD的状态不尽相同，那么如何在众多POD中快速找到没有Ready的呢？当然可以在`kubectl get pod -n <namemspace>`中肉眼搜索，但是如果可以直接打印出没有Ready的POD岂不美哉？这就是如下命令的使命：
```bash
###左右滑动
kubectl -n <namespace> get pod | grep -iv completed | awk -F"[ /]+" 'BEGIN{found=0} !/NAME/ {if (($2!=$3)||($4!="Running")) { found=1; print $0}} END { if (!found) print "All pods are ready"}' 

例子：不带namespace默认是default
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod | awk -F"[ /]+" 'BEGIN{found=0} !/NAME/ {if (($2!=$3)||($4!="Running")) { found=1; print $0}} END { if (!found) print "All pods are ready"}' 
web-0   0/1     ContainerCreating   0          14s
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod | awk -F"[ /]+" 'BEGIN{found=0} !/NAME/ {if (($2!=$3)||($4!="Running")) { found=1; print $0}} END { if (!found) print "All pods are ready"}' 
All pods are ready
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod 
NAME    READY   STATUS    RESTARTS   AGE
web-1   1/1     Running   0          9d
web-2   1/1     Running   0          9d
web-0   1/1     Running   0          34s
ubuntu@VM-16-3-ubuntu:~$ 
```

这个命令还可以搭配`for`循环来查询一个集群下各个Namespace：
```bash
###左右滑动
for i in `kubectl get ns --no-headers | awk '{print $1}'`;do echo "# Namespace "$namespace;kubectl -n $i get pod | grep -iv completed | awk -F"[ /]+" 'BEGIN{found=0} !/NAME/ {if (($2!=$3)||($4!="Running")) { found=1; print $0}} END { if (!found) print "All pods are ready"}' ;done 
```

如果想使用`-A`来查看集群中不Ready的POD，就需要稍微的变化一下`awk`中判断参数：
```bash
###左右滑动
kubectl get pod -A | grep -iv completed | awk -F"[ /]+" 'BEGIN{found=0} !/NAME/ {if (($3!=$4)||($5!="Running")) { found=1; print $0}} END { if (!found) print "All pods are ready"}' 
```

#### 查看POD中所有的Container的状态

除了查看POD以外，有的时候还需要查看POD中Container的状态，`kubectl describe pod -n <namespace> <pod name>`搭配人眼识别系统可以很浪费时间和眼力的看到容器状态，因此又攒了一个命令来快速直观的打印(同事分享，后期优化)：
```bash
###左右滑动
kubectl get pod -n <namespace> <pod name> -o jsonpath='{"###InitContainerStatus\n"}{range .status.initContainerStatuses[*]}{.name}{":\t READY: "}{.ready}{"\n"}{end}{"###ContainerStatus\n"}{range .status.containerStatuses[*]}{.name}{":\t READY: "}{.ready}{"\t STARTED: "}{.started}{"\n"}{end}'  | column -t 


例子：
ubuntu@VM-16-3-ubuntu:~$ kubectl get pod -n kube-system svclb-traefik-1b6c5f6f-8rgf9  -o jsonpath='{"###InitContainerStatus\n"}{range .status.initContainerStatuses[*]}{.name}{":\t READY: "}{.ready}{"\n"}{end}{"###ContainerStatus\n"}{range .status.containerStatuses[*]}{.name}{":\t READY: "}{.ready}{"\t STARTED: "}{.started}{"\n"}{end}'  | column -t 
###InitContainerStatus                          
###ContainerStatus                              
lb-tcp-443:             READY:  true  STARTED:  true
lb-tcp-80:              READY:  true  STARTED:  true
ubuntu@VM-16-3-ubuntu:~$ 
```

这个命令可以快速的打印出POD中Init Container和业务Container的名字以及状态，然后可以通过`kubectl logs -n <namespace> <pod name> -c <container name>`来看Container的详细log。

还可以搭配`for`循环来查看更多的POD的容器状态，比如没有Ready的POD中container的状态：
```bash
###左右滑动
namesapce=<namespace>
for i in `kubectl -n ${namesapce} get pod | grep -iv completed | awk -F"[ /]+" 'BEGIN{found=0} !/NAME/ {if (($2!=$3)||($4!="Running")) { found=1; print $0}}' | awk '{print $1}'`;do kubectl get pod -n ${namesapce} $i -o jsonpath='{"###InitContainerStatus\n"}{range .status.initContainerStatuses[*]}{.name}{":\t READY: "}{.ready}{"\n"}{end}{"###ContainerStatus\n"}{range .status.containerStatuses[*]}{.name}{":\t READY: "}{.ready}{"\t STARTED: "}{.started}{"\n"}{end}'  | column -t ;done

例子：
ubuntu@VM-16-3-ubuntu:~$ namesapce=default
for i in `kubectl -n ${namesapce} get pod | grep -iv completed | awk -F"[ /]+" 'BEGIN{found=0} !/NAME/ {if (($2!=$3)||($4!="Running")) { found=1; print $0}}' | awk '{print $1}'`;do kubectl get pod -n ${namesapce} $i -o jsonpath='{"###InitContainerStatus\n"}{range .status.initContainerStatuses[*]}{.name}{":\t READY: "}{.ready}{"\n"}{end}{"###ContainerStatus\n"}{range .status.containerStatuses[*]}{.name}{":\t READY: "}{.ready}{"\t STARTED: "}{.started}{"\n"}{end}'  | column -t ;done
###InitContainerStatus                           
###ContainerStatus                               
nginx:                  READY:  false  STARTED:  false
ubuntu@VM-16-3-ubuntu:~$ 
```

#### 其他

工作中因为种种需求还攒了很多类似的命令：

比如统计各个namespace下的POD的数量，并计算总和：
```bash
###左右滑动
pod_count=0;for i in `kubectl get ns --no-headers | awk '{print $1}'`;do echo "#Namespace $i";kubectl get pod -n $i --no-headers | grep -i running| wc -l;pod_count=$(( pod_count+`kubectl get pod -n $i --no-headers | grep -i running| wc -l` ));echo;echo;done;echo "Total CNF POD: "$pod_count 
```

比如到每一个worker上ping一个走primary网络的远端IP，其中的ping可以换成人和想执行的命令比如`ip route get <IP>`等：
```bash
###左右滑动
for i in `kubectl get node --no-headers| awk '{print $1}'`;do echo $i; ssh -q `kubectl get node $i -o jsonpath='{.status.addresses[0].address}'` "ping <DST IP> -c 2";echo;echo;done 
```

对于云原生环境来说，将Linux的基础命令和云原生的命令结合起来会得到意想不到的效果，可以大大的提升效率，所以没事儿多攒命令吧！

Linux中的`grep`和`awk`尤其重要，过滤、统计、定向打印某些东西，这俩命令基本都可以覆盖到了！

#### 附录

label相关的命令：
```bash 
###左右滑动
#查看node的所有label
kubectl get nodes --show-labels  

例子：
ubuntu@VM-16-3-ubuntu:~$ kubectl get nodes --show-labels
NAME             STATUS   ROLES                  AGE   VERSION        LABELS
vm-16-3-ubuntu   Ready    control-plane,master   11d   v1.27.7+k3s2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=k3s,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=vm-16-3-ubuntu,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=true,node-role.kubernetes.io/master=true,node.kubernetes.io/instance-type=k3s
ubuntu@VM-16-3-ubuntu:~$ 

#再知道Label名字的前提下美观的打印
kubectl get nodes -L <label key> 

例子：
ubuntu@VM-16-3-ubuntu:~$ kubectl get nodes -L "node.kubernetes.io/instance-type"
NAME             STATUS   ROLES                  AGE   VERSION        INSTANCE-TYPE
vm-16-3-ubuntu   Ready    control-plane,master   11d   v1.27.7+k3s2   k3s
ubuntu@VM-16-3-ubuntu:~$ 

#给一个node打label
kubectl label node <node name> <label name>=<key value> 

例子：
ubuntu@VM-16-3-ubuntu:~$ kubectl label node vm-16-3-ubuntu test=123
node/vm-16-3-ubuntu labeled
ubuntu@VM-16-3-ubuntu:~$ kubectl get nodes -L test
NAME             STATUS   ROLES                  AGE   VERSION        TEST
vm-16-3-ubuntu   Ready    control-plane,master   11d   v1.27.7+k3s2   123
ubuntu@VM-16-3-ubuntu:~$ 

#删除一个node的某一个label，在label名字后使用减号“-”：
kubectl label node <node name> <label name>-

例子：
ubuntu@VM-16-3-ubuntu:~$ kubectl label node vm-16-3-ubuntu test-
node/vm-16-3-ubuntu unlabeled
```

给一个node打taint或移除taint：
```bash
###左右滑动

#查看是否有taint
kubectl describe node <node name> | grep -i taint
例子：
ubuntu@VM-16-3-ubuntu:~$ kubectl describe node vm-16-3-ubuntu |grep -i taint
Taints:             <none>
ubuntu@VM-16-3-ubuntu:~$ 

#给某一个node打taint
kubectl taint NODE NAME key=value:effect
其中：effect必须是NoSchedule, PreferNoSchedule或者NoExecute.
例子：
ubuntu@VM-16-3-ubuntu:~$ kubectl taint node  vm-16-3-ubuntu test=true:NoSchedule
node/vm-16-3-ubuntu tainted
ubuntu@VM-16-3-ubuntu:~$ kubectl describe node vm-16-3-ubuntu |grep -i taint
Taints:             test=true:NoSchedule
ubuntu@VM-16-3-ubuntu:~$ 

#移除node的taint
kubectl taint NODE NAME key=value:effect-
例子:
ubuntu@VM-16-3-ubuntu:~$ kubectl taint node  vm-16-3-ubuntu test=true:NoSchedule-
node/vm-16-3-ubuntu untainted
ubuntu@VM-16-3-ubuntu:~$ kubectl describe node vm-16-3-ubuntu |grep -i taint
Taints:             <none>
ubuntu@VM-16-3-ubuntu:~$ 
```

查看当前Kubernetes支持的api和对应的版本：
```bash
###左右滑动
kubectl api-resources

例子：
ubuntu@VM-16-3-ubuntu:~$ kubectl api-resources
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
```

进入到一个Pod的容器中：
```bash
###左右滑动
kubectl exec (POD | TYPE/NAME) [-c CONTAINER] [flags] -- COMMAND [args...] [options]

例子：
ubuntu@VM-16-3-ubuntu:~$ kubectl exec service/nginx -- date
Mon Dec  4 06:21:56 UTC 2023
ubuntu@VM-16-3-ubuntu:~$ kubectl exec -n default web-0 -c nginx -- hostname
web-0
ubuntu@VM-16-3-ubuntu:~$ kubectl exec statefulsets/web -- date
Mon Dec  4 06:22:15 UTC 2023
ubuntu@VM-16-3-ubuntu:~$ kubectl exec -it -n default web-0 -c nginx -- bash
root@web-0:/# hostname;date
web-0
Mon Dec  4 06:22:27 UTC 2023
root@web-0:/# exit
exit
ubuntu@VM-16-3-ubuntu:~$
```

以上都是我在`kubectl get/logs/describe`等命令的基础上常用的其他的命令。

当然命令很多，我们可以通过帮助来熟悉用法，比如`-h --help`等参数，然后灵活搭配运用，熟能生巧！

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eBUj77IN25PNvRCId3Z3NLiaJXibYp6ibInKr2iaK30NMrSx8guWhn0iapW2WuEpdfHmbIZ26swK2M9pLw/640?wx_fmt=jpeg&amp;from=appmsg


