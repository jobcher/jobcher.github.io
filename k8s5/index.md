# kubernetes ansible自动化部署


# kubernetes ansible 自动化部署

## 服务器规划

| 角色                  | IP                           | 组件                                                       |
| :-------------------- | :--------------------------- | :--------------------------------------------------------- |
| k8s-master1           | 10.12.12.15                  | kube-apiserver kube-controller-manager kube-scheduler etcd |
| k8s-master2           | 10.12.12.17                  | kube-apiserver kube-controller-manager kube-scheduler etcd |
| k8s-02                | 10.12.12.22                  | kubelet kube-proxy docker etcd                             |
| k8s-03                | 10.12.12.21                  | kubelet kube-proxy docker etcd                             |
| load Balancer(master) | 10.12.12.15 10.12.12.23(VIP) | nginx keepalived                                           |
| load Balancer(backup) | 10.12.12.17                  | nginx keepalived                                           |

## 系统初始化

1. 关闭 selinux，firewalld
2. 关闭 swap
3. 时间同步
4. 写 hosts
5. ssh 免密（可选）

## etcd 集群部署

1. 生成 etcd 证书
2. 部署三个 ETC 集群
3. 查看集群状态

## 部署 Masterß

1. 生成 apiserver 证书
2. 部署 apiserver、controller-manager 和 scheduler 组件
3. 启动 TLS Bootstrapping

## 部署 Node

1. 安装 Docker
2. 部署 Kubelet 和 kube-proxy
3. 在 Master 上运行为新 Node 颁发证书
4. 授权 apiserver 访问 kubelet

## 部署插件（准备好镜像）

1. Flannel
2. Web UI
3. CoreDNS
4. Ingress Controller

## Master 高可用

1. 增加 Master 节点（与 Master1 一致）
2. 部署 nginx 负载均衡器
3. Nginx+Keepalived 高可用
4. 修改 Node 连接 VIP

