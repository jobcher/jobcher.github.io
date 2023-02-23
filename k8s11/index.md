# kubernetes 部署插件 (Flannel、Web UI、CoreDNS、Ingress Controller)


# k8s 部署插件

`Kubernetes` 是高度可配置且可扩展的。因此，大多数情况下， 你不需要派生自己的 Kubernetes 副本或者向项目代码提交补丁，本文会介绍几种常用的 k8s 插件，如果大家喜欢的话，希望大家点赞支持。

## 1. Flannel 网络插件

`Flannel`是由 go 语言开发，是一种基于 Overlay 网络的`跨主机容器网络解决方案`，也就是将`TCP数据包`封装在另一种网络包里面进行`路由转发和通信`，Flannel 是 CoreOS 开发，专门用于 docker 多主机互联的一个工具，简单来说，它的功能是`让集群中的不同节点主机创建的容器都具有全局唯一的虚拟IP地址`  
主要功能：

- 为每个 node 分配 subnet，容器将自动从该子网中获取 IP 地址
- 当有 node 加入到网络中时，为每个 node 增加路由配置

![flannel](/images/flannel-networking.png)

### 下载并安装

```sh
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

如果 yml 中的"Network": `10.244.0.0/16`和`kubeadm init xxx --pod-network-cidr`不一样，就需要修改成一样的。不然可能会使得`Node`间`Cluster IP`不通。

## 2. Ingress Controller

`Ingress` 是对集群中服务的外部访问进行管理的 API 对象，典型的访问方式是 HTTP。  
`Ingress` 可以提供负载均衡、SSL 终结和基于名称的虚拟托管  
下面是一个将所有流量都发送到同一 Service 的简单 Ingress 示例：  
![image](/images/ingress.svg)

`Ingress` 可为 `Service` 提供外部可访问的 URL、负载均衡流量、终止 SSL/TLS，以及基于名称的虚拟托管。 `Ingress 控制器` 通常负责通过负载均衡器来实现 Ingress，尽管它也可以配置边缘路由器或其他前端来帮助处理流量。

`Ingress 不会公开`任意端口或协议。 将 `HTTP` 和 `HTTPS` 以外的服务公开到 `Internet` 时，通常使用 `Service`.`Type=NodePort` 或 Service.`Type=LoadBalancer` 类型的 Service

### 其他

为了让 `Ingress` 资源工作，集群必须有一个正在运行的 `Ingress 控制器`。

与作为 `kube-controller-manager` 可执行文件的一部分运行的其他类型的控制器不同， Ingress 控制器不是随集群自动启动的。 基于此页面，你可选择最适合你的集群的 ingress 控制器实现。

## 3. CoreDNS

`CoreDNS` 是一个灵活可扩展的 `DNS 服务器`，可以作为 `Kubernetes 集群 DNS`。 与 Kubernetes 一样，CoreDNS 项目由 CNCF 托管。  
在 `Kubernetes 1.21` 版本中，`kubeadm` 移除了对将 `kube-dns` 作为 DNS 应用的支持。 对于 `kubeadm v1.25`，所支持的唯一的集群 DNS 应用是 `CoreDNS`。

当你使用 `kubeadm` 升级使用 `kube-dns` 的集群时，你还可以执行到 `CoreDNS` 的迁移。 在这种场景中，`kubeadm` 将基于 `kube-dns ConfigMap` 生成 `CoreDNS` 配置`（"Corefile"）`， 保存存根域和上游名称服务器的配置。

通过替换现有集群部署中的 `kube-dns`，或者使用 kubeadm 等工具来为你部署和升级集群， 可以在你的集群中使用 CoreDNS 而非 kube-dns
你必须拥有一个 `Kubernetes 的集群`，同时你的 Kubernetes 集群必须带有 `kubectl 命令行工具`。 建议在至少有两个节点的集群上运行本教程，且这些节点不作为控制平面主机。

## 4. Web UI

`Dashboard` 是基于网页的 `Kubernetes` 用户界面。 你可以使用 `Dashboard` 将容器应用部署到 `Kubernetes` 集群中，也可以对容器应用排错，还能管理集群资源。 你可以使用 `Dashboard` 获取运行在集群中的应用的概览信息，也可以创建或者修改 `Kubernetes` 资源 （如 `Deployment，Job，DaemonSet` 等等）。 例如，你可以对 `Deployment` 实现弹性伸缩、发起滚动升级、重启 `Pod` 或者使用向导创建新的应用  
![image](/images/ui-dashboard.png)

### 安装

默认情况下不会部署 `Dashboard`

```sh
#https 方式
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml

# 或者 http方式
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/alternative.yaml
```

### 暴露外部连接

创建 `webUIservice.yaml`

```sh
vim webUIservice.yaml
```

使用 loadBalancer

```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
  annotations:
    lb.kubesphere.io/v1alpha1: openelb
    protocol.openelb.kubesphere.io/v1alpha1: layer2
    eip.openelb.kubesphere.io/v1alpha2: layer2-eip
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9090
  selector:
    k8s-app: kubernetes-dashboard
  type: LoadBalancer
```

> kubectl apply -f webUIservice.yaml

### 创建并获取 token

创建管理员

```sh
kubectl create serviceaccount dashboard-admin -n kube-system
```

```sh
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
```

获取 token

```sh
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | grep dashboard-admin | awk '{print $1}')
```

