# kubernetes 从1.23.x 升级到 1.24.x


# kubernetes 从`1.23.x` 升级到 `1.24.x`

k8s 在`1.24.x`之后的版本放弃了和 docker 的兼容，使用 containerd 作为底层的容器，直接参照官方文档的资料进行更新就会报错。因为你没有安装 containerd，所以要安装 containerd 并配置才能正确的升级 k8s  
我用的是`CentOS7.9`的版本，因此以下操作都是在`CentOS`下操作。

## Master 节点操作

### 1.升级 kubeadm

```sh
yum install -y kubeadm-1.24.2-0 --disableexcludes=kubernetes
kubeadm version
kubeadm upgrade plan
sudo kubeadm upgrade apply v1.24.2
```

### 2.安装 containerd

```sh
yum install containerd.io -y
containerd config default > /etc/containerd/config.toml
vim /var/lib/kubelet/kubeadm-flags.env
```

修改 kubeadm-flags.env 变量：

```sh
KUBELET_KUBEADM_ARGS="--pod-infra-container-image=k8s.gcr.io/pause:3.6 --container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock"
```

### 3.升级 kubelet

```sh
yum install -y kubelet-1.24.2-0 kubectl-1.24.2-0 --disableexcludes=kubernetes
systemctl daemon-reload && systemctl restart containerd  && systemctl restart kubelet
```

查看状态：

> kubectl get nodes  
> systemctl status kubelet

## Worker 节点操作

### 1.升级 kubeadm

```sh
yum install -y kubeadm-1.24.2-0 --disableexcludes=kubernetes
kubeadm version
kubeadm upgrade plan
sudo kubeadm upgrade node
```

### 2.安装 containerd

```sh
yum install containerd.io -y
containerd config default > /etc/containerd/config.toml
vim /var/lib/kubelet/kubeadm-flags.env
```

修改 kubeadm-flags.env 变量：

```sh
KUBELET_KUBEADM_ARGS="--pod-infra-container-image=k8s.gcr.io/pause:3.6 --container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock"
```

### 3.升级 kubelet

```sh
yum install -y kubelet-1.24.2-0 kubectl-1.24.2-0 --disableexcludes=kubernetes
systemctl daemon-reload && systemctl restart containerd  && systemctl restart kubelet
```

查看状态：

> systemctl status kubelet

### 4.优化的维护节点

```sh
# 设置为不可调度
kubectl cordon <nodename>
# 优雅排出容器
kubectl drain <nodename> --ignore-daemonsets --delete-emptydir-data
# 确认维护完成之后，恢复正常
kubectl uncordon <nodename>
```

