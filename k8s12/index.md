# Kubernetes — 开放标准（OCI、CRI、CNI、CSI、SMI、CPI）概述


# Kubernetes — 开放标准（OCI、CRI、CNI、CSI、SMI、CPI）概述

什么是 Kubernetes 开放标准？— K8s 开放标准简介  
`开放标准`有助于和补充像 `Kubernetes` 这样的系统，Kubernetes 是用于`编排容器`的事实上的标准平台。开放标准定义了实施 `Kubernetes` 的`最佳实践`，并在支持此实施方面发挥着至关重要的作用。开放标准由开源 `Kubernetes 社区`而非某个特定供应商制定，以确保更高的效率、避免供应商锁定以及更轻松地将其他软件集成到技术堆栈中。  
![k8s](https://miro.medium.com/max/720/1*YUMhWTm6QGKVAygl3UiSYw.png)

## OCI

容器开放接口规范，由多家公司共同组成于 2015 年 6 月成立的项目（Docker, Google, CoreOS 等公司），并由 Linux 基金会运行管理，旨在围绕容器格式和运行时制定一个开放的工业化标准，目前主要有两个标准文档：容器运行时标准 （runtime spec）和 容器镜像标准（image spec）

- `OCI` 是一个开放的治理结构，其明确目的是围绕容器格式和运行时创建开放的行业标准。
- 它提供了必须由容器运行时引擎实现的规范。两个重要的规格是：
  - `runC`：种子容器运行时引擎。大多数现代容器运行时环境都使用 `runC` 并围绕这个种子引擎开发附加功能。
  - 这种低级运行时用于启动容器的各种工具，包括 `Docker` 本身。
- `OCI 规范`：关于如何运行、构建和分发容器的映像、运行时和分发规范。
- 虽然 `Docker` 经常与容器技术同步使用，但社区一直致力于 OCI 的开放行业标准。

### Image-Spec

- image-spec 定义了如何构建和打包容器镜像。
- 本规范的目标是创建可互操作的工具，用于构建、传输和准备要运行的容器映像。

### Runtime-Spec

- `runtime-spec` 指定容器的配置、执行环境和生命周期。
- 这概述了如何运行在磁盘上解压的“文件系统包(filesystem bundle)”。概括地说，OCI 实现会下载一个 OCI 映像，然后将该映像解压缩到一个 OCI 运行时文件系统包中。

### Distribution-Spec

`Distribution-Spec` 提供了一个标准，用于一般内容的分发，特别是容器图像的分发。它是 OCI 项目的`最新补充`。
实现分发规范的容器注册表为容器映像提供可靠、高度可扩展、安全的存储服务。
客户要么使用云提供商实施、供应商实施，要么使用分发的开源实施。

## CRI

`CRI`（Container Runtime Interface）：容器运行时接口，提供计算资源。​ ​kubernetes1.5​​ 版本之后，kubernetes 项目推出了自己的运行时接口 api–CRI(container runtime interface)。

- `CRI` 是关于如何在容器编排系统中实现容器运行时的规范。它在容器运行时的集成之上提供了一个抽象层。
- 它是一个插件接口，使 `kubelet` 能够使用各种容器运行时。 CRI 的核心机制是每个容器项目都可以自己实现一个 `CRI shim` 并自己处理 CRI 请求。
  > 为了允许使用 Docker 以外的其他容器运行时，Kubernetes 在 2016 年引入了 CRI。

### Docker

- `Docker` 长期以来一直是标准，但从未真正为容器编排而生。
- 它是数百万正在构建容器化应用程序的开发人员的首选。
  > 使用 Docker 作为 Kubernetes 的运行时已被弃用，并将在 Kubernetes 1.23 中删除。

### containerd

- `containerd` 是运行容器的轻量级和高性能实现。
- 它被所有主要的云提供商用于 `Kubernetes` 即服务产品。
- 使用 `containerd` 创建容器比使用 Docker 简单得多。它最近越来越受欢迎。

### CRI-O

- `CRI-O` 是 `Kubernetes CRI` 的一种实现，可以使用 `OCI` 兼容的运行时。它是使用 `Docker` 作为 `Kubernetes` 运行时的轻量级替代方案。
- 它由 `Red Hat` 创建，并具有与 `podman` 和 `buildah` 密切相关的类似代码库。
  > `containerd` 和 `CRI-O` 的想法非常简单：提供一个只包含运行容器的绝对必需品的运行时。

## CNI

`CNI`（Container Network Interface）：容器网络接口，提供网络资源。是和 CoreOS 主导制定的容器网络标准，它本身并不是实现或者代码，可以理解成一个协议。CNI 旨在为容器平台提供网络的标准化。容器平台可以从 CNI 获取到满足网络互通条件的网络参数(如 IP 地址、网关、路由、DNS 等)。

- `CNI` 是关于如何`为容器实现网络的标准`，可用于编写或配置网络插件，并且可以很容易地在各种容器编排平台中交换不同的插件。
- 每个 `CNI 插件`都必须实现为由`容器管理系统`调用的可执行文件。 CNI 插件负责将网络接口插入容器网络命名空间并在主机上进行任何必要的更改。然后它应该将 IP 分配给接口，并通过调用适当的 IPAM 插件来设置与 IP 地址管理部分一致的路由。
- `Kubernetes` 支持各种不同的网络解决方案和插件，可以在各种不同的环境中实现 Kubernetes 网络。请参阅下面的一些 CNI 实现：

### Calico

- `Calico` 是另一个可用于 Kubernetes 生态系统的流行开源 CNI 插件。 `Calico` 由 Tigera 维护，定位于网络性能、灵活性和功率等因素至关重要的环境。
- 与 `Flannel` 不同，`Calico` 提供高级网络管理安全功能，同时提供主机和 Pod 之间连接的整体概览。
- 在标准的 Kubernetes 集群上，`Calico` 可以很容易地部署为每个节点上的 `DaemonSet`。集群中的每个节点都将安装三个 `Calico` 组件：`Felix`、`BIRD` 和 `confd`，用于管理多个网络任务。

### Flannel

- `Flannel` 是一种为容器配置第 3 层网络结构的简单方法，专为 Kubernetes 设计。
- `Flannel` 由 CoreOS 开发，是可用于 Kubernetes 的最成熟的开源 CNI 项目之一。
- 它在每台主机上运行一个名为 `flanneld` 的小型单一二进制`代理`，并负责从更大的预配置地址空间中为每台主机分配子网租约。

### Cilium

- `Cilium` 是由 Linux 内核开发人员开发的开源、高度可扩展的 `Kubernetes CNI` 解决方案。
- 它作为守护进程 `cilium-agent` 部署在 `Kubernetes` 集群的每个节点上，以管理操作并将网络定义转换为 `eBPF 程序`。
- `Pod` 之间的通信通过覆盖网络或使用路由协议进行。案例支持 IPv4 和 IPv6 地址。它还通过 HTTP 请求过滤器支持 Kubernetes 网络策略。

### WeaveNet

- `Weavescope` 开发的 `Weave Net` 是一个支持 CNI 的网络解决方案，允许在 Kubernetes 集群中进行灵活的网络连接。
- 它可以在 `Kubernetes` 集群上轻松安装和配置为守护程序集，在每个节点上安装必要的网络组件。
- 它利用内核系统在节点之间传输数据包。内核利用的协议被称为`快速数据路径`，它将数据包直接传输到目标 pod，而无需多次进出用户空间。

## CSI

`CSI`（Container Storage Interface）：容器存储接口，提供存储资源。由 kubernetes、Mesos、Docker 等社区成员联合制定的一个行业标准接口规范，旨在将任意存储系统暴露给容器化应用程序。

- `CSI` 是关于如何在容器编排系统中实现存储的规范。
- 它是一种标准，用于将任意块和文件存储系统暴露给 `Kubernetes` 等容器编排系统上的容器化工作负载。
- 第三方存储提供商使用 `CSI` 公开他们的新存储系统变得非常可扩展，而无需实际接触 `Kubernetes` 代码。

### 请参阅下面的一些 CSI 实现：

- Rook
- Ceph
- OpenEBS

## SMI

- `SMI`(Service Mesh Interface) : 是关于如何在容器编排系统中实现 Service Mesh 的应用规范，重点关注 Kubernetes 和最常见的服务网格用例的基本功能集，而无需担心锁定。它涵盖了最常见的服务网格功能：
- 流量策略——跨服务应用身份和传输加密等策略
- 流量遥测——捕获关键指标，如错误率和服务之间的延迟
- 流量管理——在不同服务之间转移流量

### Istio

- `Istio` 是一个开源服务网格，它透明地分层到现有的分布式应用程序上。它提供了一种统一且更有效的方式来保护、连接和监控服务。
- 它是负载平衡、服务到服务身份验证和监控的途径——几乎不需要更改服务代码。
- `Istio` 通过`分布式`或`微服务架构`解决了开发人员和运营商面临的挑战。

### Linkerd

- `Linkerd` 是 `Kubernetes`的服务网格，它通过为您提供运行时调试、可观察性、可靠性和安全性，使运行服务更容易、更安全——所有这些都不需要对代码进行任何更改。
- 它通过在每个服务实例旁边安装一组超轻、透明的`代理`来工作。这些代理会自动处理进出服务的所有流量。

## CPI

- `CPI` (Cloud Provider Interface) :是关于如何实现 Kubernetes 集群的规范。它将底层云基础设施功能的智能与核心 `Kubernetes` 分离。

### 请参阅下面的一些 CPI 实现：

- AWS
- Azure
- GCP

