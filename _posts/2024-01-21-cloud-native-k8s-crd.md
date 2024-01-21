---
layout:     post
title:      云原生｜什么是K8s里的CRD(Custom Resource Definitions)？
date:       2024-01-21
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

最近给同事排错的时候，又遇到了CRD丢失导致微服务无法创建的问题。

那么什么是CRD？CRD到底是做什么的呢？

Kubernetes 自定义资源定义（Custom Resource Definition，简称 CRD）是一种强大的 Kubernetes API 扩展机制。它允许用户创建和管理自定义资源，这些资源不是 Kubernetes 标准 API 的一部分。CRD 使得 Kubernetes 不仅限于内建资源（如 Pod、Service 等），还可以支持用户定义的资源类型。

CRD 的核心思想是通过声明式的 API 来扩展 Kubernetes 的功能。你可以定义自己的资源结构，然后像操作 Kubernetes 内建资源一样操作这些自定义资源。

#### 什么是Kubernetes内建资源呢？

一个Kubernetes集群建立后，为什么我们就可以创建POD、Config、Secret等资源呢？

这是因为集群建立起来后默认允许我们创建这些资源，这就是Kubernetes内建的可用API资源，可以通过命令`kubectl api-resources`查看，但是每个Kubernetes的版本由于默认安装的微服务不一样会导致其内建API资源会不一样，如下是一个例子，通过grep剔除掉crd创建的API资源，剩下的就是cluster内建的：
```bash
####左右滑动
crds=$(for i in `kubectl get crd --no-headers | awk -F"." '{print $1}'`;do echo -n "$i|";done | sed 's/|$//g')
kubectl api-resources | grep -vE ${crds}
```

![@七禾页话][2]

#### 为什么需要CRD呢？

Kubernetes项目组并不知道不同公司业务都需要哪些API资源，所以Kubernetes发行版只包含项目开发的API资源，比如POD、ConfigMap等，但是对于不同公司，这些基础的内置API资源并不能满足业务需求，而且有些资源是所有业务共用的，就可以把这个共用的资源在自己的集群中统一定义，从而成为此集群的一种API被调用，这就是CRD——Custom Resource Definition。

具体来说，Custom Resource是 Kubernetes 开放了API的定义规则，是对 Kubernetes 内置API的扩展。现在，很多 Kubernetes 核心功能也都用Custom Resource来实现了，这使得 Kubernetes 更加模块化。

Custom Resource可以通过动态注册的方式在运行中的集群内或出现(create)或消失(delete)，集群管理员可以独立于集群更新Custom Resource。 一旦某Custom Resource被安装，用户可以使用 kubectl 来创建和访问其中的对象。

CRD可以通过`kubectl get customresourcedefinitions.apiextensions.k8s.io`或者缩写的`kubectl get crd`来查看：
![@七禾页话][3]

#### 如何查看CRD定义的API资源？

CRD的资源虽然是各个公司、或者不同的项目自己定制的，但是创建后也是作为 Kubernetes 资源而存在，所以可以通过`kubectl api-resources`查看，如下是一个例子，通过grep过滤由CRD定制的API资源，这些资源都是通过后期创建的：
```bash
####左右滑动
crds=$(for i in `kubectl get crd --no-headers | awk -F"." '{print $1}'`;do echo -n "$i|";done | sed 's/|$//g')
kubectl api-resources | grep -E ${crds}
```

![@七禾页话][4]

#### 如何定义和使用CRD定义的API资源

Kubernetes里的所有东西都是通过创建实现的，CRD也不例外。我们也可以手动的apply一个CRD的yaml从而定义自己需要的API资源，编写 CRD YAML 的步骤包括定义 API 版本、Kind、以及资源的 schema。以下是一个简单的 CRD 示例，定义了一个名为 `CronTab` 的资源：
```yaml
####左右滑动
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
      - ct
```

![@七禾页话][5]

在这个例子中，我们定义了一个新的资源类型 `CronTab`，它有三个字段：`cronSpec`、`image` 和 `replicas`。

CRD创建后，就可以像操作 Kubernetes 内建资源一样操作自定义资源。比如，我们再创建一个 `CronTab` 实例，YAML如下所示：
```yaml
####左右滑动
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-crontab
spec:
  cronSpec: "*/5 * * * *"
  image: my-cron-image
  replicas: 1
```

![@七禾页话][6]

再举个例子，MetalLB作为一种负载均衡器，其也有很多自己的Customer Resource，从官网拿到的MetalLB的yaml里有很多的CRD定义，当我们创建了MetaLLB的微服务后，可以看到MetalLB的定义资源也同步创建好，并且可以在`api-resources`列表里看到：

```bash
####左右滑动
kubectl get crd | grep -i metallb
kubectl api-resources | grep -i metallb
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
kubectl get crd | grep -i metallb
kubectl api-resources | grep -i metallb
```
![@七禾页话][7]

当定义的资源出现在了`api-resources`里，表明此资源可以在当前集群中使用了。

我们一般定义MetalLB的IP资源池是直接写到`ConfigMap`里，但是我们通过CRD可以看到有一个独立的API资源叫做`ipaddresspools`，因此我们可以直接通过`ipaddresspools`的API资源创建MetalLB的IP Pool实例：
```bash
####左右滑动
cat <<EOF | kubectl create -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.10.0/24
  - 192.168.9.1-192.168.9.5
  - fc00:f853:0ccd:e799::/124
EOF
```

![@七禾页话][8]

#### CRD创建的API资源会消失吗？

CRD创建的API资源会消失吗？答案是肯定会的。

可以看到我们查看CRD的命令是`get`，创建可以是`create`，那么对应的`delete`就可以删除CRD了，CRD删除后，对应的`api-resources`和相关的实例也就消失了，如果CRD通过helm等方式安装的，一旦helm里的instance被uninstall，其对应的CRD资源也会消失：

![@七禾页话][9]

这便是同事遇到的问题，其实提示很明显，一旦get或者创建中提示什么什么version没有，基本就是CRD定义的API资源没有了，比如下图：

![@七禾页话][10]

如何解决呢？必须要通过重新创建CRD将对应的API资源复原，CRD是通过yaml创建的，所以我们只要能写出来CRD的yaml，就可以复原其API资源，这个时候可以在其他集群中查看是否有我们需要的CRD，然后通过`kubectl get crd <crd name> -o yaml`打印后，删除会导致冲突的信息，比如创建时间、uuid、status等，剩下的就是CRD的yaml信息，另外把握住CRD的一些主要内容，如API 版本、Kind、以及资源的 schema 即可：
![@七禾页话][11]

有了CRD的yaml后通过`kubectl apply -f <crd yaml file>`或者`kubectl create -f <crd yaml file>`使其生效即可。

#### 后记

值得注意的是，CRD是集群级别的，跟内置API资源一样，不会被namespace隔离，一种CRD只需要在一个集群中创建一次，同名的CRD会被新版本的创建所覆盖，从而集群中保留的永远都是最新版的CRD，也就是最新版本的API资源类型。

因此在使用的时候需要特别注意CRD的问题，当多个微服务共用一个集群的时候，CRD是只需要安装最新版本的即可，因此每一个运维人员在安装CRD前都应该要先查看自己需要的CRD是否已经被安装，版本是否低于自己的需求：如果版本低，可以安装新的的CRD；如果集群里的CRD已经是自己需要的版本或者更高，那么不要再进行CRD的安装，否则会降低CRD的版本导致其他调用此CRD资源的微服务出问题。

CRD 作为 Kubernetes API的扩展机制，已经是 Kubernetes 集群中不可或缺的组成部分，为 Kubernetes 的扩展性和灵活性提供了基础。通过定义自己的资源类型，开发者可以创建完全定制的、高度适应特定需求的 Kubernetes 应用。不管是简单的配置管理，还是复杂的运维自动化，CRD 都为 Kubernetes 用户打开了一扇新的大门。

以上，有想法欢迎留言来聊！

------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCURq9Libic7AZouYPpXd1JuuTxPYD4r3WFBIEt428zSffnqwibVTNjPoGCoUUItPJ8pSvPvtcXsElrg/640?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDIribribiaicFibfKibadkFGxPBg6bUF1en9GLQXu5iczOKx76E59sfVHibR7tDEC89LGvLIddcGk8j7IXbw/640?wx_fmt=png&amp;from=appmsg


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDIribribiaicFibfKibadkFGxPBgJHRl6JWyjuSWoic1VWCgC9ibasMuE7xIU4QCrhhzsWZC2OV4UcsPxrBg/640?wx_fmt=png&amp;from=appmsg


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDIribribiaicFibfKibadkFGxPBgFOehiaMLnmdS2dW13ZbSOn85Y5RuribQLz0nrD5RKXkgic9IibwW5UMAtQ/640?wx_fmt=png&amp;from=appmsg


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDIribribiaicFibfKibadkFGxPBglic2T4L48EZCibn6ibrrRvwRcpszjKJ2f0iaIEWnrB1HTG3kXymvViaD6BQ/640?wx_fmt=png&amp;from=appmsg


[6]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDIribribiaicFibfKibadkFGxPBgOPvJkJXyZHKVM33O4B8JLpEUAiacefGKbwDtE0STxW7KWb20GuybmoA/640?wx_fmt=png&amp;from=appmsg


[7]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDIribribiaicFibfKibadkFGxPBgYTKjoVQKC8F319vW0A0hMoV2qpYRYehviacy0XYgE8enyPjwgUrfvxw/640?wx_fmt=png&amp;from=appmsg


[8]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDIribribiaicFibfKibadkFGxPBggicvK2mUWzSTS9DiaNdcMlFVTe6X0NRicx5xqvEp2VDiaQFT6IQfjostWw/640?wx_fmt=png&amp;from=appmsg


[9]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDIribribiaicFibfKibadkFGxPBgIvu5y5PR6m5pCyFHM6ZgUmZ6qQSgvjG6neFkzVZRmoL1KiaZhkRMIicg/640?wx_fmt=png&amp;from=appmsg


[10]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDIribribiaicFibfKibadkFGxPBgur3kDbPyTWMd0XeibUed0YtvF9llAyKFgFrfyVU0Rb8rXRH3xDjl1VQ/640?wx_fmt=png&amp;from=appmsg


[11]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eDIribribiaicFibfKibadkFGxPBg9OH32lU4OaBzEBI5pgaGYNFYc1tlOasmCzLFCXBMIV1fpaetbMExibA/640?wx_fmt=png&amp;from=appmsg

