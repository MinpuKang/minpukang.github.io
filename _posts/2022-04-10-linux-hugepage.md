---
layout:     post
title:      Linux｜大内存页(HugePage)的三三两两
date:       2022-04-10
categories: blog
author:     "琉璃康康"
header-img: "img/post.jpg"
tags:
    - Linux
    - 大内存页
    - HugePage
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

![1][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

最近有同事问了几个关于大内存页(HugePage)的问题，就顺便复习并拓展的看了下相关的内容，根据自己的理解做个简单总结，如有纰漏欢迎指正。

大内存页，简称大页，但是要想聊这个，首先得简单聊聊内存。

#### 物理内存和虚拟内存
内存分为物理内存，也就是服务器电脑上直接插的内存条；CPU是直接操作物理内存的，但是当多程序运行的时候，就可能出现内存使用重叠覆盖的情况，导致程序无法同时运行，因此就有了虚拟内存。
![2][2]

虚拟内存，是给每一个进程都独立分配了一套虚拟内存，虚拟内存间独立互不干涉，然后操作系统再将虚拟内存映射到物理内存。

虚拟内存和物理内存的映射就需要有一定的机制，来保证进程运行访问虚拟内存的时候，可以写入不同的物理内存，避免冲突的发生。

虚拟内存和物理内存的映射机制有分页和分段两大类，我们的大页就跟内存分页机制有关系。

#### 内存分页
内存分页是将虚拟内存和物理内存按照固定大小切割成一段一段，这样一段尺寸固定的内存空间，就称为一个页(Page)。

虚拟内存和物理内存之间通过页表来存储其映射关系，然后CPU在使用内存的时候，内存管理单元(MMU)在页表中找到虚拟内存所以映射的物理内存，然后将物理内存告诉CPU来完成内存调度。
![3][3]

在MMU查找虚拟和物理内存映射关系，为了减少直接访问页表，就有了TLB(Translation Lookaside Buffer)的缓存机制，TLB是一块儿高速缓存，它缓存了使用过的虚拟和物理内存映射关系，当MMU想要查找内存映射关系的时候首先从TLB里查找，如果找不到再去访问页表。
![4][4]

那么我可以看到页的大小，直接影响了页表的大小，这也间接地造成了内存的消耗(页表也要占内存的)。

因此页越大，映射关系越少，页表也就越小，页表也小，TLB的失效情况也就越小，那么在MMU查找关系的时候直接访问TLB查到的几率也就越大，速度也就更快了。

大部分处理器默认的页大小是4KB，也有8KB、16KB或者64KB，显而易见这样的页太小了，尤其是在云和虚拟化中，这样的页大小将大大降低相应速度，因此就引入了HugePage的概念，将页扩大到2M甚至1G，目前Linux常用的HugePage大小为2M和1GB。

#### Linux的HugePage
Linux是如何查看大页的配置？可以直接查看/proc/meminfo中的Mem和HugePage相关内容，如下的结果中一共有2G的内存，大页是2M的页，但是没有任何可以使用的大页(HugePages_Total=0)：
```
$ grep -iE "mem|huge" /proc/meminfo 
MemTotal:        2035212 kB
MemFree:         1165688 kB
MemAvailable:    1612916 kB
Shmem:              2600 kB
AnonHugePages:         0 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
FileHugePages:         0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
```

这个时候想要使用大页，就需要在内核中声明可以使用的大页数量，文件/proc/sys/vm/nr_hugepages中保存了可以使用的大页数量，比如允许10个大页数量的使用：
```
$ echo 10 > /proc/sys/vm/nr_hugepages
```

再次打印Mem和HugePage的结果：
```
$ grep -iE "mem|huge" /proc/meminfo 
MemTotal:        2035212 kB
MemFree:         1135996 kB
MemAvailable:    1584272 kB
Shmem:              2608 kB
AnonHugePages:         0 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
FileHugePages:         0 kB
HugePages_Total:      10
HugePages_Free:       10
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:           20480 kB
```

对比设置10个大页数量之前的结果可以看到可以使用内存减少了1165688-1135996=29692KB，其中大页占据了20480KB，还有一部分的内存被页表使用。

这样配置之后，进程就可以使用2M的大页了，但是数量最多是10个。

#### Openstack中的大页使用
我们做云和虚拟化的时候经常会涉及到大页的使用，比如openstack中的flavor定义里必须要开启大页的使用：
```
# 可以左右滑动
$ openstack flavor set FLAVOR-NAME --ram RAM_IN_MB --disk ROOT_DISK_IN_GB --vcpus NUMBER_OF_VCPUS --property hw:mem_page_size=PAGE_SIZE
```
PAGE_SIZE值如下：
- **small**: (default) The smallest page size is used. Example: 4 KB on x86.
- **large**: Only use larger page sizes for guest RAM. Example: either 2 MB or 1 GB on x86.
- **any**: It is left up to the compute driver to decide. In this case, the libvirt driver might try to find large pages, but fall back to small pages. Other drivers may choose alternate policies for any.
- **pagesize**: (string) An explicit page size can be set if the workload has specific requirements. This value can be an integer value for the page size in KB like 1048576(1GB), or can use any standard suffix. Example: 4KB, 2MB, 2048, 1GB. This is also called customized page size.

那么openstack中创建某一个VM后，它所使用的大页数量就是定义的Mem/PageSize，比如下边的一个flavor：
```
# 可以左右滑动
$ openstack flavor set test --ram 10240 --disk 40 --vcpus 10 10240 --property hw:mem_page_size=1048576
内存将要使用10240MB=10GB，使用大页的页大小是1048576Kb=1G，所以最终占用的大页数量就是10Gb/1Gb=10个
```

这里多说一点儿，现在的server都有numa架构的，也就是多个物理cpu共存， 每一个物理cpu称为一个node，一个node不仅仅包括了cpu，也包括对应的内存；cpu调度的内存的时候，可以访问自己node下的内存，也可以访问其他的node下的内存，但是显而易见的是访问自己的node速度更快。
![5][5]

可以使用numactl查看各个numa的cpu、mem，需要注意的是这个命令不是系统默认安装的：
```
# 可以左右滑动
numactl --hardware    #共有2个node，各领取16个CPU和128G内存
available: 2 nodes (0-1)
node 0 cpus: 0 1 2 3 4 5 6 7 16 17 18 19 20 21 22 23 
node 0 size: 131037 MB
node 0 free: 3019 MB
node 1 cpus: 8 9 10 11 12 13 14 15 24 25 26 27 28 29 30 31
node 1 size: 131071 MB
node 1 free: 9799 MB
node distances:
node 0 1
 0: 10 20
 1: 20 10
```

默认情况下在openstack中创建一个vm分配资源的时候要尽量不跨numa。

但是受限于一个node下的cpu和内存数量的影响，对于需要很大资源的vm或者单一node已经不足以满足一个vm的情况下，跨numa的分配就是必须要的了。

在openstack是通过flavor中设定numa的property来允许是否跨numa并且对各个numa的是如何调度的，参数如下：
```
$ openstack flavor set FLAVOR-NAME \
    --property hw:numa_nodes=FLAVOR-NODES \
    --property hw:numa_cpus.N=FLAVOR-CORES \
    --property hw:numa_mem.N=FLAVOR-MEMORY
```
参数解释：
- **FLAVOR-NODES**: (integer) The number of host NUMA nodes to restrict execution of instance vCPU threads to. If not specified, the vCPU threads can run on any number of the host NUMA nodes available.
- **N**: (integer) The instance NUMA node to apply a given CPU or memory configuration to, where N is in the range 0 to FLAVOR-NODES - 1.
- **FLAVOR-CORES**: (comma-separated list of integers) A list of instance vCPUs to map to instance NUMA node N. If not specified, vCPUs are evenly divided among available NUMA nodes.
- **FLAVOR-MEMORY**: (integer) The number of MB of instance memory to map to instance NUMA node N. If not specified, memory is evenly divided among available NUMA nodes.

举个例子，允许将来的vm跨numa使用，并且使用numa0的vcpu是0和1，内存1G；同时使用numa1的vcpu1、3、5、7，内存2G：
```
$ openstack flavor set FLAVOR-NAME \
    --property hw:numa_nodes=2 \
    --property hw:numa_cpus.0=0,2 \
    --property hw:numa_mem.0=1024 \
    --property hw:numa_cpus.1=1,3,5,7 \
    --property hw:numa_mem.1=2048
```

当然也可以只允许跨numa调度资源，但是至于每个numa是用多少自行决定，比如：
```
$ openstack flavor set FLAVOR-NAME \
    --property hw:numa_nodes=2
```

回到大页的问题，每个server上都可以查看各个numa上可以使用的大页数量：
```
# 可以左右滑动
cat /sys/devices/system/node/node*/meminfo | grep -i huge
Node 0 AnonHugePages:         0 kB
Node 0 ShmemHugePages:        0 kB
Node 0 HugePages_Total:   174
Node 0 HugePages_Free:     14
Node 0 HugePages_Surp:      0
Node 1 AnonHugePages:         0 kB
Node 1 ShmemHugePages:        0 kB
Node 1 HugePages_Total:   174
Node 1 HugePages_Free:     14
Node 1 HugePages_Surp:      0
```

### K8s中的大页使用
那么k8s里的大页是如何计算使用的呢？首先是在部署k8s的时候各个node可以使用的大页数量已经定义好，可以使用kubectl describe nodes查看各个worker是否有可以使用的大页：
```
# 可以左右滑动
apiVersion: v1
kind: Node
metadata:
  name: node1
# ignore...
status:
  capacity:
    memory: 10Gi
    hugepages-2Mi: 1Gi
  allocatable:
    memory: 9Gi
    hugepages-2Mi: 1Gi
# ignore...

Note: 上述结果说明这个node允许使用大页的页大小是2M，然后一共有1G可以使用，那么大页的数量就是1G/2M=512个。
```

Pod使用大页，就在pod的配置中直接写入使用的大页页大小和总量即可：
```
# 可以左右滑动
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
    # use the image you like
    volumeMounts:
      - mountPath: /hugepages
        name: hugepage
    resources:
      requests:
        hugepages-2Mi: 100Mi
      limits:
        hugepages-2Mi: 400Mi
  volumes:
    - name: hugepage
      emptyDir:
        medium: HugePages

Note：上述结果就是当前pod使用的是2M的大页，使用总量是100M，也就是会占用100/2=50个页；最大可是使用的大页是400M，也就是最多可以占用400/2=200个页。
```

以上就是我自己关于大页的来源和在openstack以及k8s中的使用的小总结，有些地方可能理解的比较片面，对于运维来说应该是够用了，但是对于开发来说可能需要更深入的挖掘。

以下是参考的一些博客和官网链接，有更详细的针对内存和大页的细节和原理的介绍，深挖请继续：
- https://zhuanlan.zhihu.com/p/491457649
- https://zhuanlan.zhihu.com/p/366702339
- https://docs.openstack.org/nova/pike/admin/huge-pages.html
- https://docs.openstack.org/nova/pike/admin/flavors.html
- https://access.redhat.com/documentation/en-us/openshift_container_platform/4.5/html/scalability_and_performance/what-huge-pages-do-and-how-they-are-consumed
- https://dannypsnl.github.io/blog/2019/05/04/cs/hugepages-on-kubernetes/


------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eCXG000nAxX49ib6bibTgWmA1ibzRQGP8jichcXjW3QFy7X77Y53XnDEhUiboBArIlYeIksNzRZS9HSTmA/0?wx_fmt=jpeg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXG000nAxX49ib6bibTgWmA1s33fVmMDtYiclaWOtD4L6N7qADkZfHlWJmia2ia8PnpxyJ7p78ZXERGtw/0?wx_fmt=png


[3]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXG000nAxX49ib6bibTgWmA1x8v5PdnJibhUOia4PILCEjcjRjf0SxYBopv3uKmFOzMnUnjY9vkqbOag/0?wx_fmt=png


[4]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eCXG000nAxX49ib6bibTgWmA10P4p8Lt46vZdMOPnRAibxnc7iaomTDg0HD0JfW7kKGIoficVJYUTF8iaLA/0?wx_fmt=png


[5]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eB7FCCRye6AzPH4E053nbs6czQE6af15uVD8B5wWBBouI8ywK4ibyT6HoibvOzMQXxv2iaqib6OQibibFJA/0?wx_fmt=png
