# kubernetes 脚本快速安装


# kubernetes 脚本快速安装

- 1、三台机器设置自己的 hostname（不能是 localhost）

```sh
# 修改 hostname;  k8s-01要变为自己的hostname
hostnamectl set-hostname k8s-01
# 设置 hostname 解析
echo "127.0.0.1   $(hostname)" >> /etc/hosts
```

- 2、所有机器批量执行如下脚本

- ```sh
  #先在所有机器执行 vi k8s.sh
  # 进入编辑模式（输入i），把如下脚本复制
  # 所有机器给脚本权限  chmod +x k8s.sh
  #执行脚本 ./k8s.sh
  ```

```sh
#/bin/sh

#######################开始设置环境##################################### \n


printf "##################正在配置所有基础环境信息################## \n"


printf "##################关闭selinux################## \n"
sed -i 's/enforcing/disabled/' /etc/selinux/config
setenforce 0
printf "##################关闭swap################## \n"
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab

printf "##################配置路由转发################## \n"
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.d/k8s.conf

## 必须 ipv6流量桥接
echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.d/k8s.conf
## 必须 ipv4流量桥接
echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.d/k8s.conf
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.d/k8s.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.d/k8s.conf
echo "net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.d/k8s.conf
echo "net.ipv6.conf.all.forwarding = 1"  >> /etc/sysctl.d/k8s.conf
modprobe br_netfilter
sudo sysctl --system


printf "##################配置ipvs################## \n"
cat <<EOF | sudo tee /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules
sh /etc/sysconfig/modules/ipvs.modules


printf "##################安装ipvsadm相关软件################## \n"
yum install -y ipset ipvsadm




printf "##################安装docker容器环境################## \n"
sudo yum remove docker*
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce-19.03.9  docker-ce-cli-19.03.9 containerd.io
systemctl enable docker
systemctl start docker

sudo systemctl daemon-reload
sudo systemctl restart docker


printf "##################安装k8s核心包 kubeadm kubelet kubectl################## \n"
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

###指定k8s安装版本
yum install -y kubelet-1.21.0 kubeadm-1.21.0 kubectl-1.21.0

###要把kubelet立即启动。
systemctl enable kubelet
systemctl start kubelet

printf "##################下载api-server等核心镜像################## \n"
sudo tee ./images.sh <<-'EOF'
#!/bin/bash
docker pull k8s.gcr.io/kube-apiserver:v1.21.9
docker pull k8s.gcr.io/kube-controller-manager:v1.21.9
docker pull k8s.gcr.io/kube-scheduler:v1.21.9
docker pull k8s.gcr.io/kube-proxy:v1.21.9
docker pull k8s.gcr.io/pause:3.4.1
docker pull k8s.gcr.io/etcd:3.4.13-0
docker pull k8s.gcr.io/coredns/coredns:v1.8.0
EOF

chmod +x ./images.sh && ./images.sh

### k8s的所有基本环境全部完成
```

- 3、使用 kubeadm 引导集群（参照初始化 master 继续做）

```sh

#### --apiserver-advertise-address 的地址一定写成自己master机器的ip地址
#### 虚拟机或者其他云厂商给你的机器ip  10.96  192.168
#### 以下的只在master节点执行
kubeadm init \
--apiserver-advertise-address=10.12.12.24 \
--kubernetes-version v1.21.0 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=10.124.0.0/16

```

- 4、master 结束以后，按照控制台引导继续往下

```sh
## 第一步
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

##第二步
export KUBECONFIG=/etc/kubernetes/admin.conf

##第三步 部署网络插件
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml



##第四步，用控制台打印的kubeadm join 去其他node节点执行
kubeadm join 10.170.11.8:6443 --token cnb7x2.lzgz7mfzcjutn0nk \
	--discovery-token-ca-cert-hash sha256:00c9e977ee52632098aadb515c90076603daee94a167728110ef8086d0d5b37d
```

## 初始化 worker 节点（worker 执行）

```sh
##过期怎么办
kubeadm token create --print-join-command
kubeadm join --token y1eyw5.ylg568kvohfdsfco --discovery-token-ca-cert-hash sha256: 6c35e4f73f72afd89bf1c8c303ee55677d2cdb1342d67bb23c852aba2efc7c73
```

- 5、验证集群

```sh
#等一会，在master节点执行
kubectl get nodes
```

- 6、设置 kube-proxy 的 ipvs 模式

```sh
##修改kube-proxy默认的配置
kubectl edit cm kube-proxy -n kube-system
## 修改mode: "ipvs"

##改完以后重启kube-proxy
### 查到所有的kube-proxy
kubectl get pod -n kube-system |grep kube-proxy
### 删除之前的即可
kubectl delete pod 【用自己查出来的kube-proxy-dw5sf kube-proxy-hsrwp kube-proxy-vqv7n】  -n kube-system

###

```

