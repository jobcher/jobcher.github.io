# Kubernetes 安装


# Kubernetes 安装

## 环境配置

### 关闭防火墙： 如果是云服务器，需要设置安全组策略放行端口

```sh
systemctl stop firewalld
systemctl disable firewalld
```

### 修改 hostname

```sh
hostnamectl set-hostname k8s-01
echo "127.0.0.1   $(hostname)" >> /etc/hosts
reboot
```

### 关闭 selinux：

```sh
sed -i 's/enforcing/disabled/' /etc/selinux/config
setenforce 0
```

### 关闭 swap：

```sh
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

### 修改 /etc/sysctl.conf

```sh
# 如果有配置，则修改
sed -i "s#^net.ipv4.ip_forward.*#net.ipv4.ip_forward=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-ip6tables.*#net.bridge.bridge-nf-call-ip6tables=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-iptables.*#net.bridge.bridge-nf-call-iptables=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.all.disable_ipv6.*#net.ipv6.conf.all.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.default.disable_ipv6.*#net.ipv6.conf.default.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.lo.disable_ipv6.*#net.ipv6.conf.lo.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.all.forwarding.*#net.ipv6.conf.all.forwarding=1#g"  /etc/sysctl.conf
# 可能没有，追加
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding = 1"  >> /etc/sysctl.conf
# 执行命令以应用
sysctl -p
```

## 安装 docker

```sh
sudo yum remove docker*
sudo yum install -y yum-utils
#配置docker yum 源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#安装docker 19.03.9
yum install -y docker-ce-3:19.03.9-3.el7.x86_64  docker-ce-cli-3:19.03.9-3.el7.x86_64 containerd.io

#安装docker 19.03.9   docker-ce  19.03.9
yum install -y docker-ce-19.03.9-3  docker-ce-cli-19.03.9 containerd.io

#启动服务
systemctl start docker
systemctl enable docker

sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 安装 k8s 核心（都执行）

### 配置 K8S 的 yum 源

```sh
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]ß
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 卸载旧版本,安装新版本

```sh
yum remove -y kubelet kubeadm kubectl

# 查看可以安装的版本
yum list kubelet --showduplicates | sort -r

# 安装kubelet、kubeadm、kubectl 指定版本
yum install -y kubelet-1.21.0 kubeadm-1.21.0 kubectl-1.21.0

# 开机启动kubelet
systemctl enable kubelet && systemctl start kubelet
```

## 初始化 master 节点

### 创建 images.sh,`vim images.sh`粘贴以下命令

```sh
docker pull k8s.gcr.io/kube-apiserver:v1.21.9
docker pull k8s.gcr.io/kube-controller-manager:v1.21.9
docker pull k8s.gcr.io/kube-scheduler:v1.21.9
docker pull k8s.gcr.io/kube-proxy:v1.21.9
docker pull k8s.gcr.io/pause:3.4.1
docker pull k8s.gcr.io/etcd:3.4.13-0
docker pull k8s.gcr.io/coredns/coredns:v1.8.0
```

```sh
chmod +x images.shß
sh images.sh
```

### kubeadm init master 节点

```sh
kubeadm init \
--apiserver-advertise-address=192.168.99.19 \
--kubernetes-version v1.23.3 \
--service-cidr=10.99.0.0/16 \
--pod-network-cidr=10.124.0.0/16
```

### 复制相关文件夹

```sh
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 导出环境变量

```sh
export KUBECONFIG=/etc/kubernetes/admin.conf

```

### 部署一个 pod 网络

```sh
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### 命令检查

```sh
kubectl get pod -A  ##获取集群中所有部署好的应用Pod
kubectl get nodes  ##查看集群所有机器的状态
```

## 初始化 worker 节点（worker 执行）

```sh
##过期怎么办
kubeadm token create --print-join-command
kubeadm join --token y1eyw5.ylg568kvohfdsfco --discovery-token-ca-cert-hash sha256: 6c35e4f73f72afd89bf1c8c303ee55677d2cdb1342d67bb23c852aba2efc7c73
```

### 验证集群

```sh
#获取所有节点
kubectl get nodes
```

