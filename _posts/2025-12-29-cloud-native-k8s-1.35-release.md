---
layout:     post
title:      云原生｜Kubernetes 1.35发布：Timbernetes——世界树，有多项重点确定性更新
date:       2025-12-29
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

![@七禾页话，8月份在呼伦贝尔休假，在自然中放纵，世界才是美好的][1]

> 学习永无止境，记录相伴相随！
> <p align="right">—— 琉璃康康</p>

上一篇都是三个月前了，作为一个不勤奋的自媒体博主更新的确实不是很勤奋，主打一个三天打鱼两天晒网。

为什么要写今天这一篇呢？是因为正好看到了Kubernetes 1.35于12月17号发布，而这个新版本中有一个更新正好跟我最近做的一个项目有关系，这就是稳定版中的“In-place update of Pod resources（Pod资源原地更新）”。

Kubernetes 1.35的主题叫做Timbernetes (The World Tree Release)——世界树，Logo如下图（三只松鼠大礼包）。

![@七禾页话][2]

在官网介绍中说2025年始于“Octarine: The Color of Magic（魔法之色）”（v1.33）的微光，又经历了“Wind&Will(风和意志)"(v1.34)的洗礼，我们得以双手触摸着世界树，灵感来自于链接众多领域的生命之树Yggdrasil。Kubernetes这个世界树也如同任何伟大的树一样，在全球摄影的关爱塑造下，伴随着一个又一个发行版本，一圈一圈的成长起来了。

```
Yggdrasil：北欧神话中构成整个世界的巨木，树种为白蜡树，树冠高达天际，枝干衍生九个世界，包括米德加尔特（人类世界）、阿斯加德（阿萨神族国度）、赫尔海姆（死之国）、尼福尔海姆（雾之国）、穆斯贝尔海姆（火之国）、约顿海姆（巨人国）、亚尔夫海姆（精灵国）、斯瓦塔尔夫海姆（矮人国）、华纳海姆（华纳神族居所）。
```

然后发布中又介绍了这个logo的含义，比如树中心是Kubernetes的标记齿轮，基石是那些坚实的拥护者、贡献者和用户。三只松鼠分别是持有LGTM卷轴的法师代表了审查者（reviewers），中间拿着战斧和盾牌的是发布团队，最右边提灯的调皮鬼（rogue）代表了为黑暗的问题队列带来光明的分类者（这个我是没明白到底是谁，是那些给Issue出谋划策解决bug的人？）。

反正这个世界树的logo原文看着很有西方修道士的感觉，原文粘贴如下，期待更好的翻译：

```
2025 began in the shimmer of Octarine: The Color of Magic (v1.33) and rode the gusts Of Wind & Will (v1.34). We close the year with our hands on the World Tree, inspired by Yggdrasil, the tree of life that binds many realms. Like any great tree, Kubernetes grows ring by ring and release by release, shaped by the care of a global community.

At its center sits the Kubernetes wheel wrapped around the Earth, grounded by the resilient maintainers, contributors and users who keep showing up. Between day jobs, life changes, and steady open-source stewardship, they prune old APIs, graft new features and keep one of the world’s largest open source projects healthy.

Three squirrels guard the tree: a wizard holding the LGTM scroll for reviewers, a warrior with an axe and Kubernetes shield for the release crews who cut new branches, and a rogue with a lantern for the triagers who bring light to dark issue queues.

Together, they stand in for a much larger adventuring party. Kubernetes v1.35 adds another growth ring to the World Tree, a fresh cut shaped by many hands, many paths and a community whose branches reach higher as its roots grow deeper.
```

Kubernetes这些年在云原生的发展中占据了绝对的统治地位，称之为当前ICT行业的世界树也不为过，所以这个logo的发布也是非常贴切的。

Kubernetes v1.35 充满了新功能和改进。其中有稳定功能（GA-General Availabiliyt，比如v1），有Beta功能(比如v1beta1)，也有Alpha功能（实验性阶段，比如v1alpha1），以下简单总结了稳定更新的十大功能(包括新功能，废弃的功能和对原有功能的改善)。

**v1.35中毕业（GA）、废弃和删除的功能**

**Pod 资源的原地更新**

Kubernetes 已将 Pod 资源的原地更新升级为正式发布（GA）。此功能允许用户调整 CPU 和内存资源，而无需重启 Pod 或容器。以前，此类修改需要重新创建 Pod，这可能会中断工作负载，特别是对于有状态或批处理应用程序。早期的 Kubernetes 版本仅允许更改现有 Pod 的基础设施资源设置（请求和限制）。新的原地更新功能实现了更平滑、非中断的垂直扩缩容，提高了效率，同时也能简化开发。这项工作由 SIG Node 主导的 KEP #1287 完成。

**PreferSameNode 流量分发**

Service 的 `trafficDistribution` 字段已更新，以提供更明确的流量路由控制。引入了一个新选项 `PreferSameNode`，让服务在可用时严格优先选择本地节点上的端点，否则回退到远程端点。同时，现有的 `PreferClose` 选项已重命名为 `PreferSameZone`。这一变化通过明确指示流量优先在当前可用区内传输，使 API 不言自明。虽然 `PreferClose` 为向后兼容而保留，但 `PreferSameZone` 现在是区域路由的标准，确保节点级和区域级偏好被清晰区分。这项工作由 SIG Network 主导的 KEP #3015 完成。

**Job API 的 managed-by 机制**

Job API 现在包含一个 `managedBy` 字段，允许外部控制器处理 Job 状态同步。此功能在 Kubernetes v1.35 中升级为稳定版，主要由 MultiKueue（一个多集群分发系统）驱动，其中在管理集群中创建的 Job 被镜像并在工作集群中执行，状态更新被传播回来。为了实现此工作流，内置的 Job 控制器不得对特定的 Job 资源进行操作，以便 Kueue 控制器可以管理状态更新。目标是允许将 Job 同步干净地委托给另一个控制器。它并非旨在向该控制器传递自定义参数或修改 CronJob 的并发策略。这项工作由 SIG Apps 主导的 KEP #4368 完成。

**通过 .metadata.generation 实现可靠的 Pod 更新跟踪**

历史上，Pod API 缺少其他 Kubernetes 对象（如 Deployments）中存在的 `metadata.generation` 字段。由于此遗漏，控制器和用户没有可靠的方法来验证 kubelet 是否实际处理了 Pod 规范的最新更改。这种模糊性对于像 Pod 原地垂直扩缩容这样的功能尤其成问题，因为很难知道资源调整请求何时生效。Kubernetes v1.33 作为 Alpha 功能为 Pod 添加了 `.metadata.generation` 字段。该字段现在在 v1.35 的 Pod API 中稳定，这意味着每次 Pod 的 spec 更新时，`.metadata.generation` 的值都会递增。作为此改进的一部分，Pod API 还获得了一个 `.status.observedGeneration` 字段，用于报告 kubelet 已成功看到并处理的 generation。Pod 条件也各自包含其独立的 `observedGeneration` 字段，客户端可以报告和/或观察。由于此功能在 v1.35 中已稳定，因此可用于所有工作负载。这项工作由 SIG Node 主导的 KEP #5067 完成。

**拓扑管理器可配置的 NUMA 节点限制**

拓扑管理器历史上使用硬编码的限制，即最多支持 8 个 NUMA 节点，以防止在亲和性计算期间出现状态爆炸。（这里有一个重要的细节；NUMA 节点与 Kubernetes API 中的 Node 不同。）对 NUMA 节点数量的限制阻止了 Kubernetes 充分利用现代高端服务器，这些服务器越来越多地采用具有超过 8 个 NUMA 节点的 CPU 架构。Kubernetes v1.31 在拓扑管理器策略配置中引入了一个新的测试版选项 `max-allowable-numa-nodes`。在 Kubernetes v1.35 中，该选项已稳定。启用它的集群管理员可以使用具有超过 8 个 NUMA 节点的服务器。尽管配置选项是稳定的，但 Kubernetes 社区意识到大型 NUMA 主机的性能较差，并且有一个提议的增强（KEP-5726）旨在改进它。您可以通过阅读[控制节点上的拓扑管理策略]来了解更多信息。这项工作由 SIG Node 主导的 KEP #4622 完成。

**Ingress NGINX 退役**

多年来，Ingress NGINX 控制器一直是将流量路由到 Kubernetes 集群的流行选择。它灵活、被广泛采用，并作为无数应用程序的标准入口点。然而，维护该项目已变得不可持续。由于维护人员严重短缺和技术债务堆积，社区最近做出了退役它的艰难决定。

这并非严格属于 v1.35 发布的一部分，但这是一个如此重要的变化，我们想在此强调一下。因此，Kubernetes 项目宣布，Ingress NGINX 在 2026 年 3 月之前将仅接受尽力而为的维护。在此日期之后，它将被归档，不再提供更新。推荐的迁移路径是转向 Gateway API，它为流量管理提供了一个更现代、更安全、更可扩展的标准。您可以在官方博客文章中了解更多信息。

**移除 cgroup v1 支持**

在管理 Linux 节点上的资源时，Kubernetes 历来依赖于 cgroups（控制组）。虽然最初的 cgroup v1 功能可用，但它通常不一致且有限制。这就是为什么 Kubernetes 在 v1.25 中引入了对 cgroup v2 的支持，提供了更清晰、统一的层次结构和更好的资源隔离。

由于 cgroup v2 现在是现代标准，Kubernetes 准备在 v1.35 中废弃旧版的 cgroup v1 支持。这对集群管理员来说是一个重要通知：如果您仍在运行不支持 cgroup v2 的旧版 Linux 发行版上运行节点，您的 kubelet 将无法启动。为避免停机，您需要将这些节点迁移到已启用 cgroup v2 的系统上。要了解更多信息，请阅读关于 cgroup v2 的内容；您也可以通过 KEP-5573：移除 cgroup v1 支持来跟踪切换工作。

**废弃 kube-proxy 中的 ipvs 模式**

多年前，Kubernetes 采用了 kube-proxy 中的 ipvs 模式，以提供比标准 iptables 更快的负载均衡。虽然它提供了性能提升，但使其与不断发展的网络需求保持同步造成了太多的技术债务和复杂性。

由于这种维护负担，Kubernetes v1.35 废弃了 ipvs 模式。尽管该模式在此版本中仍然可用，但当配置为使用它时，kube-proxy 现在将在启动时发出警告。目标是简化代码库并专注于现代标准。对于 Linux 节点，您应开始过渡到 nftables，这是现在推荐的替代方案。您可以在 KEP-5495：废弃 kube-proxy 中的 ipvs 模式中找到更多信息。

**对 containerd v1.X 的最后支持**

虽然 Kubernetes v1.35 仍然支持 containerd 1.7 和其他 LTS 版本，但这是最后一个提供此类支持的版本。SIG Node 社区已指定 v1.35 为支持 containerd v1.X 系列的最后一个版本。这是一个重要的提醒：在升级到下一个 Kubernetes 版本之前，您必须切换到 containerd 2.0 或更高版本。为了帮助识别哪些节点需要注意，您可以在集群内监控 `kubelet_cri_losing_support` 指标。您可以在官方博客文章或 KEP-4033：从 CRI 发现 cgroup 驱动程序中找到更多信息。

**kubelet 重启期间改进的 Pod 稳定性**

以前，重启 kubelet 服务通常会导致 Pod 状态出现暂时性中断。在重启期间，kubelet 会重置容器状态，导致健康的 Pod 被标记为 `NotReady` 并从负载均衡器中移除，即使应用程序本身仍在正确运行。

为了解决这个可靠性问题，此行为已得到纠正，以确保无缝的节点维护。kubelet 现在在启动时能正确从运行时恢复现有容器的状态。这确保您的工作负载在 kubelet 重启或升级期间保持 `Ready` 状态，流量继续不间断流动。您可以在 KEP-4781：修复 kubelet 重启后容器就绪状态不一致中找到更多信息。


**测试版中的新功能**

另外 v1.35 Beta版本中也有一些有用的功能，AI翻译如下。

**通过 Downward API 暴露节点拓扑标签**

从 Pod 内部访问节点拓扑信息（例如区域和可用区）通常需要查询 Kubernetes API 服务器。虽然功能上可行，但这种方法需要广泛的 RBAC 权限或边车容器来获取基础设施元数据，从而增加了复杂性和安全风险。Kubernetes v1.35 将通过 Downward API 直接暴露节点拓扑标签的能力提升至测试版。kubelet 现在可以将标准拓扑标签（如 `topology.kubernetes.io/zone` 和 `topology.kubernetes.io/region`）作为环境变量或投射卷文件注入到 Pod 中。主要好处是为工作负载提供了一种更安全、更高效的方式来感知拓扑。这允许应用程序原生适应其可用区或区域，而无需依赖 API 服务器，通过遵循最小权限原则来加强安全性并简化集群配置。
注意：Kubernetes 现在将可用的拓扑标签注入到每个 Pod 中，以便它们可以用作 Downward API 的输入。随着 v1.35 的升级，大多数集群管理员会看到每个 Pod 上添加了几个新标签；这是设计预期的一部分。这项工作由 SIG Node 主导的 KEP #4742 完成。

**存储版本迁移的原生支持**

在 Kubernetes v1.35 中，存储版本迁移的原生支持升级为测试版并默认启用。此举将迁移逻辑直接集成到核心 Kubernetes 控制平面（"树内"），消除了对外部工具的依赖。历史上，管理员依靠手动"读/写循环"——通常是将 `kubectl get` 的输出通过管道传递给 `kubectl replace`——来更新模式或重新加密静态数据。这种方法效率低下且容易发生冲突，特别是对于像 Secrets 这样的大型资源。此版本中，内置控制器自动处理更新冲突和一致性令牌，提供了一种安全、精简且可靠的方法，以最小的操作开销确保存储的数据保持最新。这项工作由 SIG API Machinery 主导的 KEP #4192 完成。

**可变的卷附加限制**

CSI（容器存储接口）驱动是 Kubernetes 的一个插件，为存储系统暴露给容器化工作负载提供了一致的方式。CSINode 对象记录了节点上安装的所有 CSI 驱动的详细信息。然而，节点上报告的附加容量和实际附加容量之间可能出现不匹配。当 CSI 驱动启动后卷槽位被占用时，kube-scheduler 可能会将需要状态存储的 Pod 分配到容量不足的节点上，最终导致 Pod 卡在 ContainerCreating 状态。Kubernetes v1.35 使 `CSINode.spec.drivers[*].allocatable.count` 变为可变的，从而可以动态更新节点的可用卷附加容量。它还允许 CSI 驱动通过引入可配置的刷新间隔（通过 CSIDriver 对象定义）来控制所有节点上 `allocatable.count` 值的更新频率。此外，它在检测到由于容量不足导致卷附加失败时，会自动更新 `CSINode.spec.drivers[*].allocatable.count`。尽管此功能在 v1.34 中作为测试版毕业，但默认禁用功能标志 `MutableCSINodeAllocatableCount`，在 v1.35 中它仍处于测试阶段以收集反馈，但功能标志已默认启用。这项工作由 SIG Storage 主导的 KEP #4876 完成。

**机会主义批处理**

历史上，Kubernetes 调度器按顺序处理 Pod，时间复杂度为 O(Pod 数量 × 节点数量)，这可能导致对兼容 Pod 的冗余计算。此 KEP 引入了一种机会主义批处理机制，旨在通过 Pod 调度签名识别此类兼容 Pod 并将它们批处理在一起，从而在它们之间共享过滤和评分结果，以提高性能。Pod 调度签名确保具有相同签名的两个 Pod 从调度角度来看是"相同的"。它不仅考虑了 Pod 和节点属性，还考虑了系统中的其他 Pod 以及关于 Pod 放置的全局数据。这意味着任何具有给定签名的 Pod 将从任意节点集获得相同的分数/可行性结果。批处理机制包括两个可以在需要时调用的操作 - `create` 和 `nominate`。`create` 根据具有有效签名的 Pod 的调度结果创建新的批处理信息集。`nominate` 使用来自 `create` 的批处理信息，为签名与规范 Pod 签名匹配的新 Pod 设置提名节点名称。这项工作由 SIG Scheduling 主导的 KEP #5598 完成。

**StatefulSets 的 maxUnavailable**

StatefulSet 运行一组 Pod 并维护每个 Pod 的粘性身份。这对于需要稳定网络标识符或持久存储的有状态工作负载至关重要。当 StatefulSet 的 `.spec.updateStrategy.<type>` 设置为 `RollingUpdate` 时，StatefulSet 控制器将删除并重新创建 StatefulSet 中的每个 Pod。它将按照 Pod 终止的顺序（从最大序号到最小序号）进行，每次更新一个 Pod。Kubernetes v1.24 向 StatefulSet 的滚动更新配置设置添加了一个新的 Alpha 字段，称为 `maxUnavailable`。除非集群管理员明确选择加入，否则该字段不是 Kubernetes API 的一部分。在 Kubernetes v1.35 中，该字段已进入测试版并默认可用。您可以使用它来定义更新期间可以不可用的最大 Pod 数量。此设置与将 `.spec.podManagementPolicy` 设置为 `Parallel` 结合使用时最为有效。您可以将 `maxUnavailable` 设置为正数（例如：2）或所需 Pod 数量的百分比（例如：10%）。如果未指定此字段，则默认为 1，以保持之前一次只更新一个 Pod 的行为。此改进允许有状态应用程序（可以容忍多个 Pod 关闭）更快地完成更新。这项工作由 SIG Apps 主导的 KEP #961 完成。

**kubeconfig 中可配置的凭据插件策略**

可选的 kubeconfig 文件是一种将服务器配置和集群凭据与用户偏好分开的方式，而不会因意外输出干扰正在运行的 CI 流水线。作为 v1.35 发布的一部分，kubeconfig 获得了额外功能，允许用户配置凭据插件策略。此更改引入了两个字段：`credentialPluginPolicy`（允许或拒绝所有插件）和 `credentialPluginAllowlist`（允许指定允许的插件列表）。这项工作由 SIG Auth 和 SIG CLI 合作的 KEP #3104 完成。

**KYAML**

YAML 是一种人类可读的数据序列化格式。在 Kubernetes 中，YAML 文件用于定义和配置资源，例如 Pods、Services 和 Deployments。然而，复杂的 YAML 难以阅读。YAML 重要的空格需要仔细注意缩进和嵌套，而其可选的字符串引号可能导致意外的类型强制转换（参见：挪威错误）。虽然 JSON 是一种替代方案，但它不支持注释，并且对尾随逗号和带引号的键有严格的要求。KYAML 是专门为 Kubernetes 设计的更安全、歧义更少的 YAML 子集。此功能在 v1.34 中作为可选的 Alpha 功能引入，在 Kubernetes v1.35 中已升级为测试版并默认启用。可以通过设置环境变量 `KUBECTL_KYAML=false` 来禁用它。KYAML 解决了 YAML 和 JSON 的挑战。所有 KYAML 文件也是有效的 YAML 文件。这意味着您可以编写 KYAML 并将其作为输入传递给任何版本的 kubectl。这也意味着您不需要编写严格的 KYAML 来解析输入。这项工作由 SIG CLI 主导的 KEP #5295 完成。

**HorizontalPodAutoscalers 的可配置容忍度**

Horizontal Pod Autoscaler (HPA) 历史上依赖于固定的全局 10% 容忍度来进行扩缩容操作。这种硬编码值的一个缺点是，需要高敏感性的工作负载（例如需要在 5% 负载增加时扩容的工作负载）经常被阻止扩容，而其他工作负载可能会不必要地振荡。在 Kubernetes v1.35 中，可配置容忍度功能升级为测试版并默认启用。此增强功能允许用户在 HPA 行为字段中为每个资源定义自定义的容忍度窗口。通过设置特定的容忍度（例如，将其降低到 0.05 表示 5%），操作员可以精确控制自动扩缩容的敏感性，确保关键工作负载对小的指标变化做出快速反应，而无需进行集群范围的配置调整。这项工作由 SIG Autoscaling 主导的 KEP #4951 完成。

**Pod 中用户命名空间的支持**

Kubernetes 正在增加对用户命名空间的支持，允许 Pod 使用隔离的用户和组 ID 映射运行，而不是共享主机 ID。这意味着容器可以在内部以 root 身份运行，但实际上在主机上映射到一个非特权用户，从而在发生安全漏洞时降低权限提升的风险。该功能提高了 Pod 级别的安全性，并使需要容器内 root 权限的工作负载运行更安全。随着时间的推移，通过 id 映射挂载，支持已扩展到无状态和有状态 Pod。这项工作由 SIG Node 主导的 KEP #127 完成。

**VolumeSource：OCI 制品和/或镜像**

创建 Pod 时，您通常需要为容器提供数据、二进制文件或配置文件。这意味着需要将内容包含到主容器镜像中，或使用自定义的初始化容器将文件下载并解压到 `emptyDir` 中。这两种方法仍然有效。Kubernetes v1.31 增加了对 `image` 卷类型的支持，允许 Pod 声明式地拉取 OCI 容器镜像制品并解压到卷中。这使您可以使用标准的 OCI 注册表工具来打包和交付仅包含数据的制品，例如配置、二进制文件或机器学习模型。使用此功能，您可以完全将数据与容器镜像分离，并消除对额外初始化容器或启动脚本的需求。`image` 卷类型自 v1.33 起处于测试版，并在 v1.35 中默认启用。请注意，使用此功能需要兼容的容器运行时，例如 containerd v2.1 或更高版本。这项工作由 SIG Node 主导的 KEP #4639 完成。

**对缓存镜像强制执行 kubelet 凭据验证**

当前的 `imagePullPolicy: IfNotPresent` 设置允许 Pod 使用节点上已缓存的容器镜像，即使 Pod 本身没有拉取该镜像的凭据。这种行为的一个缺点是，它在多租户集群中造成了安全漏洞：如果具有有效凭据的 Pod 将敏感的私有镜像拉取到节点上，同一节点上后续未经授权的 Pod 只需依赖本地缓存即可访问该镜像。此 KEP 引入了一种机制，kubelet 对缓存的镜像强制执行凭据验证。在允许 Pod 使用本地缓存的镜像之前，kubelet 会检查 Pod 是否具有拉取它的有效凭据。这确保了只有授权的工作负载才能使用私有镜像，无论它们是否已存在于节点上，显著加强了共享集群的安全态势。在 Kubernetes v1.35 中，此功能已升级为测试版并默认启用。用户仍可通过将 `KubeletEnsureSecretPulledImages` 特性门禁设置为 `false` 来禁用它。此外，`imagePullCredentialsVerificationPolicy` 标志允许操作员配置所需的安全级别，范围从优先考虑向后兼容性的模式到提供最大安全性的严格强制执行模式。这项工作由 SIG Node 主导的 KEP #2535 完成。

**细粒度的容器重启规则**

历史上，`restartPolicy` 字段严格在 Pod 级别定义，强制 Pod 内的所有容器具有相同的行为。这种全局设置的一个缺点是缺乏对复杂工作负载（如 AI/ML 训练作业）的细粒度控制。这些工作负载通常需要将 Pod 的 `restartPolicy` 设置为 `Never` 来管理工作完成，但个别容器会受益于针对特定可重试错误（如网络故障或 GPU 初始化失败）的原位重启。Kubernetes v1.35 通过在容器 API 本身启用 `restartPolicy` 和 `restartPolicyRules` 来解决这个问题。这允许用户为单个常规容器和初始化容器定义重启策略，这些策略独立于 Pod 的整体策略运行。例如，现在可以将容器配置为仅在以特定错误代码退出时自动重启，从而避免因瞬时故障而重新调度整个 Pod 的昂贵开销。在此版本中，该功能已升级为测试版并默认启用。用户可以立即在其容器规范中利用 `restartPolicyRules` 来优化长时间运行工作负载的恢复时间和资源利用率，而无需更改其 Pod 的更广泛生命周期逻辑。这项工作由 SIG Node 主导的 KEP #5307 完成。

**通过 secrets 字段为 CSI 驱动选择加入服务账户令牌**

向容器存储接口（CSI）驱动提供 ServiceAccount 令牌历来依赖于将它们注入到 `volume_context` 字段中。这种方法存在显著的安全风险，因为 `volume_context` 旨在用于非敏感配置数据，并且经常被驱动和调试工具以明文记录，可能泄露凭据。Kubernetes v1.35 引入了一种选择加入机制，允许 CSI 驱动通过 NodePublishVolume 请求中的专用 `secrets` 字段接收 ServiceAccount 令牌。驱动现在可以通过在其 CSIDriver 对象中将 `serviceAccountTokenInSecrets` 字段设置为 `true` 来启用此行为，指示 kubelet 安全地填充令牌。主要好处是防止凭据意外暴露在日志和错误消息中。此更改确保敏感的工作负载身份通过适当的安全通道处理，符合密钥管理的最佳实践，同时保持对现有驱动的向后兼容性。这项工作由 SIG Auth 与 SIG Storage 合作的 KEP #5538 完成。

**Deployment 状态：终止中的副本计数**

历史上，Deployment 状态提供了有关可用副本和已更新副本的详细信息，但缺乏对正在关闭过程中的 Pod 的明确可见性。这种遗漏的一个缺点是，用户和控制器无法轻松区分稳定的 Deployment 和仍有 Pod 正在执行清理任务或遵守较长宽限期的 Deployment。Kubernetes v1.35 将 Deployment 状态中的 `terminatingReplicas` 字段提升至测试版。此字段提供已设置删除时间戳但尚未从系统中移除的 Pod 的计数。此功能是改进 Deployment 处理 Pod 替换的更广泛计划中的基础步骤，为未来关于在滚动更新期间何时创建新 Pod 的策略奠定了基础。主要好处是改善了生命周期管理工具和操作员的可观察性。通过暴露正在终止的 Pod 数量，外部系统现在可以做出更明智的决策，例如在继续后续任务之前等待完全关闭，而无需手动查询和过滤单个 Pod 列表。这项工作由 SIG Apps 主导的 KEP #3973 完成。


以上，有想法欢迎留言来聊！

引用链接:

[1]Kubernetes v1.35: Timbernetes: https://kubernetes.io/blog/2025/12/17/kubernetes-v1-35-release/ (可点击**阅读原文**直接跳转)

[2]Kubernetes CHANGELOG-1.35.md: https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.35.md

[3]Kubernetes v1.35 抢先一览: https://kubernetes.io/zh-cn/blog/2025/11/26/kubernetes-v1-35-sneak-peek/


------------
<p align="center">网络和应用</p>
<p align="center">摄影和旅行</p>
<p align="center">工作和生活</p>
<p align="center">欢迎关注公众号：七禾页话(qiheyehk)</p>
<img src="https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAaCjLpPgUZricqK7lIOO3hYEYIbjibRlYaiaTsib0reaQfQTmaibVw2QqZLibBWpCHJdg0v3V7yX8sQgWw/0?wx_fmt=jpeg" width="50%"/>


[0]: http://mmbiz.qpic.cn/mmbiz_gif/QqiaFS6NT0eCHicr2j8v4oD4rClUscedr9r55alibqTP1e9kss3HO7voULLsEv4yicuFFy0IJJeLAzX88yzyU9VTgA/640?wx_fmt=gif


[1]: https://mmbiz.qpic.cn/mmbiz_jpg/QqiaFS6NT0eAFmmGTicGCr8zrgBx8qy73vDsfILP82qKW8Ykr5hNxfC26fFnFRFD9VjHqCfIvdlsl2WmaYdQXQyg/640?wx_fmt=jpeg&amp;from=appmsg


[2]: https://mmbiz.qpic.cn/mmbiz_png/QqiaFS6NT0eAFmmGTicGCr8zrgBx8qy73vD4LVjxicYib1jd54qwXicm6k9unzfPfGRwNlFlYsNpEQ0GMa3CfkBMvzQ/640?wx_fmt=png&amp;from=appmsg

