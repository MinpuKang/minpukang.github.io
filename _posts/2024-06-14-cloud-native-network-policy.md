---
layout:     post
title:      云原生｜以为理解了External-Traffic-Policy，结果又被NetworkPolicy坑了
date:       2024-06-14
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

![@七禾页话，6月13日夜的雷暴闪电@大连][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

前几天聊了[external-traffic-policy](https://mp.weixin.qq.com/s/3zJqkQj59LuYyIKbJc1AwA)的问题，结果遇到了一个值是Cluster，但是只能在POD所在的Node使用SVC IP和业务通信，让我怀疑了这个Cluster的值到底是不是真的可以转给集群中任何Node上的业务POD，结果最后发现是学艺不精导致。

那么是什么问题呢？又是什么原因呢？在此之前先来看看[external-traffic-policy](https://mp.weixin.qq.com/s/3zJqkQj59LuYyIKbJc1AwA)两个值Cluster和Local的特性：

- Local：流量只转给收到此流量的Node上的业务POD。
- Cluster：流量可以转给集群中任何Node的业务POD（可以是本Node上的，也可以是其他Node上的）。

![@七禾页话][2]

对于Cluser或者Local的使用也做了简单的说明：

- 如果想要性能好、时延小，选择Local，少一次SNAT，也少了跨Node的转发，性能就会好很多，但是需要注意负载和路由的问题。

- 如果在没有很好的LoadBalancer的情况下，Cluster可以保证业务的均衡以及在逻辑上绝对不丢包的保证。

另外有一种情况是必须要用Cluster的，那就是多service共享同一个LoadBalancer IP，这个在K8s集群中通过插件就可以支持，比如使用metallb分配ip的时候给每个service定义相同的allow-shared-ip值就可以让多个service共用同一个IP，但是不同service关联的业务POD是不一样的，那么不同的POD肯定是可以被schedule到不同的Node上，比如下图，不同业务的POD落在不同的Node上，所以需要DCGW路由器上针对这个共享的SVC VIP定义的路由是要到集群中所有的Node：

![@七禾页话][3]

假设如果某个service的external-traffic-policy使用的是local，那么对于DCGW这台路由器来说是无法感知不同service对应的POD在哪个Node上的，当需要转外部流量给集群时，DCGW只按照自己的路由表通过ECMP规则转发，这样就可能将流量转给没有业务POD的其他Node，导致业务不通：

![@七禾页话][5]

因此针对多service共享IP的情况，所有service的external-traffic-policy必须要使用Cluster。

到这里理论和实践都没有问题，结果这次在一个产品升级后，也是多service共用同一个IP，这些service的external-traffic-policy都是使用Cluster，但是其中的一个service只能在其业务POD所在的Node上业务才可以通，在其他的Node上或者在集群外部想要访问此service(IP+Port)的时候都是不通的，即下图中的1和2都不通，只有3在POD所在的node上访问业务才会成功，但是service2没有这个问题，不管是从外部还是其他Node上访问Service2（IP+Port）都是成功的：

![@七禾页话][4]

这完全不是Cluster想要的效果，但是问题出在哪里了呢？

经过检查产品软件新版本，发现了一个叫做networkPolicy的参数，然后就去集群里发现有两个networpolicy，一个是允许http的流量，一个是deny其他所有的流量，而ssh业务匹配了deny的networkPolicy，因此不通：
```bash
### 左右滑动
username@k8s-controller:~> kubectl get networkpolicies.networking.k8s.io -n test
NAME                                                  POD-SELECTOR                                               AGE
allow-http                                            app.kubernetes.io/name=http                                15h
deny-from-other-namespaces                            <none>                                                     15h
username@k8s-controller:~> kubectl get networkpolicies.networking.k8s.io -n test -o yaml
apiVersion: v1
items:
- apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    annotations:
      meta.helm.sh/release-namespace: test
    creationTimestamp: "2024-06-10T11:17:20Z"
    generation: 1
    labels:
      app: test
      heritage: Helm
      name: test
    name: allow-http
    namespace: test
    resourceVersion: "2134612259"
    uid: abc6bb3d-217f-46dc-8dea-ba6c6411abca
  spec:
    ingress:
    - {}
    podSelector:
      matchLabels:
        app.kubernetes.io/name: http
    policyTypes:
    - Ingress
  status: {}
- apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    annotations:
      meta.helm.sh/release-namespace: test
    creationTimestamp: "2024-06-10T11:17:20Z"
    generation: 1
    labels:
      app: test
      heritage: Helm
      name: test
    name: deny-from-other-namespaces
    namespace: test
    resourceVersion: "2134612251"
    uid: ac480203-6e32-49f1-8f79-381bd4cdabcb
  spec:
    ingress:
    - from:
      - podSelector: {}
    podSelector: {}
    policyTypes:
    - Ingress
  status: {}
kind: List
metadata:
  resourceVersion: ""
```

然后，直接创建一个针对ssh的networkPolicy后，问题解决，需要注意matchLabels里app.kubernetes.io/name的`ssh`需要跟POD打的label名字一样：
```bash
### 左右滑动
cat <<EOF | kubectl create -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ssh
  namespace: test
spec:
  ingress:
  - {}
  podSelector:
    matchLabels:
      app.kubernetes.io/name: ssh
  policyTypes:
  - Ingress
EOF
```

创建之后的NetworkPolicy如下：
```bash
username@k8s-controller:~> kubectl get networkpolicies.networking.k8s.io -n test
NAME                                                  POD-SELECTOR                                               AGE
allow-http                                            app.kubernetes.io/name=http                                15h
allow-ssh                                             app.kubernetes.io/name=ssh                                 1m28s
deny-from-other-namespaces                            <none>                                                     15h
```

然后external-traffic-policy的Cluster就继续按照它最开始的意义工作，即流量可以转给集群中任何Node的业务POD（可以是本Node上的，也可以是其他Node上的）。

所以如果在K8s中遇到网络业务不通的问题，NetworkPolicy也是一个需要检查的点，那么NetworkPolicy到底是什么呢？

简单来说NetworkPolicy就是K8s里的ACL，通过NetworkPolicy的定义来允许某些流量的进出，进来的就是ingress+from，出去的就egress+to，下边是一个相对细致的NetworkPolicy例子：
```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

上边名为test-network-policy的网络策略起到了如下的作用：

1、 隔离 default 名字空间下 role=db 的 Pod 。

2、（Ingress 规则）允许以下 Pod 连接到 default 名字空间下的带有 role=db 标签的所有 Pod 的 6379 TCP 端口：

  - default 名字空间下带有 role=frontend 标签的所有 Pod

  - 带有 project=myproject 标签的所有名字空间中的 Pod
  
  - IP 地址范围为 172.17.0.0–172.17.0.255 和 172.17.2.0–172.17.255.255 （即，除了 172.17.1.0/24 之外的所有 172.17.0.0/16）

3、（Egress 规则）允许 default 名字空间中任何带有标签 role=db 的 Pod 到 CIDR 10.0.0.0/24 下 5978 TCP 端口的连接。

更多NetworkPolicy的解释和如何使用可以参考k8s的官网介绍：[https://kubernetes.io/zh-cn/docs/concepts/services-networking/network-policies/](https://kubernetes.io/zh-cn/docs/concepts/services-networking/network-policies/)

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCVTK33mzF6maEBzVTwpUapicGnppjQicasl0UD7zepENaBPTZQa0I9wra2QAFFhiaHNjdDcvjSXxglg/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eBGYDy2fxoHGcuvLEMRByib2gejW6mMzNibjCibjOqejaohPMm3HXH1Ep6ybpb2PsCbfgicskiccIiczH5A/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCVTK33mzF6maEBzVTwpUapibkC5I2N49iadsiceZwooHfbA7tZdS7QhII2W5Vt6lUDnaIvwteaeTOzQ/640?wx_fmt=png&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCVTK33mzF6maEBzVTwpUapGhibPNIe1JiaRlkWSHO65hG8p38cOYc0gibug3uSrQXUbtIK9057UaT7w/640?wx_fmt=png&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCVTK33mzF6maEBzVTwpUapp04lTKe6DgibyLGx30VXvg1q6VLnXEMJvibdHTkicib49EAvZ8Yx1jLddg/640?wx_fmt=png&amp;from=appmsg