# Kubernetes 实验手册（1）


# Kubernetes 实验手册（1）

通过在 pve 创建 5 台虚拟机：

| 节点  | IP             | 作用          |
| :---- | :------------- | :------------ |
| node0 | 192.168.99.69  | k8s-master01  |
| node1 | 192.168.99.9   | k8s-master02  |
| node2 | 192.168.99.53  | k8s-master03  |
| node3 | 192.168.99.41  | k8s-node01    |
| node4 | 192.168.99.219 | k8s-node02    |
| node5 | 192.168.99.42  | k8s-master-lb |

| 配置信息     | 备注           |
| :----------- | :------------- |
| 系统版本     | Ubuntu         |
| Docker       | 20.10.12       |
| pod 网段     | 172.168.0.0/12 |
| service 网段 | 10.96.0.0/12   |

VIP 不要和内网 IP 重复，VIP 需要和主机在同一个局域网内

1. 更新 ansible 连接

```sh
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.155
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.199
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.87
#ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.41
#ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.219
```

```sh
vim /etc/hosts
192.168.99.155  k8s-master01
192.168.99.199  k8s-master02
192.168.99.87  k8s-master03
#192.168.99.41   k8s-node01
#192.168.99.219  k8s-node02
```

## 基本配置

1. 安装基本软件包

```sh
apt install wget jq psmisc vim net-tools telnet lvm2 git -y
# 关闭swap分区
vim /etc/fstab
注释掉swap 内容 并重启
reboot
# 时间同步
apt install ntpdate -y
# 查看时区
timedatectl set-timezone 'Asia/Shanghai'
timedatectl
date
```

2. 安装 docker

```sh
curl -sSL https://get.daocloud.io/docker | sh
systemctl restart docker
```

3. 安装 k8s 组件

```sh
# 更新 apt 包索引并安装使用 Kubernetes
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
# 下载 Google Cloud 公开签名秘钥：
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
# 添加 Kubernetes apt 仓库
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
# 更新 apt 包索引，安装 kubelet、kubeadm 和 kubectl，并锁定其版本：
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

```

4. 安装 keepalived 和 haproxy
   所有 Master 节点安装 HAProxy 和 KeepAlived

```sh
apt install keepalived haproxy -y
cp -rf /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
rm -rf /etc/haproxy/haproxy.cfg
vim /etc/haproxy/haproxy.cfg
```

所有 Master 节点的 HAProxy 配置相同

```sh
global
  maxconn  2000
  ulimit-n  16384
  log  127.0.0.1 local0 err
  stats timeout 30s

defaults
  log global
  mode  http
  option  httplog
  timeout connect 5000
  timeout client  50000
  timeout server  50000
  timeout http-request 15s
  timeout http-keep-alive 15s

frontend monitor-in
  bind *:33305
  mode http
  option httplog
  monitor-uri /monitor

frontend k8s-master
  bind 0.0.0.0:16443
  bind 127.0.0.1:16443
  mode tcp
  option tcplog
  tcp-request inspect-delay 5s
  default_backend k8s-master

backend k8s-master
  mode tcp
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server k8s-master01	192.168.99.155:6443  check
  server k8s-master02	192.168.99.199:6443  check
  server k8s-master03	192.168.99.87:6443  check
```

所有 Master 节点配置 KeepAlived，配置不一样，注意区分  
注意每个节点的 IP 和网卡（interface 参数）
vim /etc/keepalived/keepalived.conf

```conf
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2
rise 1
}
vrrp_instance VI_1 {
    state MASTER
    interface ens18 #查看网关地址
    mcast_src_ip 192.168.99.155 #本机IP
    virtual_router_id 51
    priority 101
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.99.42 # vip地址
    }
#    track_script {
#       chk_apiserver
#    }
}
```

5. 配置 KeepAlived 健康检查文件
   vim /etc/keepalived/check_apiserver.sh

```sh
#!/bin/bash

err=0
for k in $(seq 1 3)
do
    check_code=$(pgrep haproxy)
    if [[ $check_code == "" ]]; then
        err=$(expr $err + 1)
        sleep 1
        continue
    else
        err=0
        break
    fi
done

if [[ $err != "0" ]]; then
    echo "systemctl stop keepalived"
    /usr/bin/systemctl stop keepalived
    exit 1
else
    exit 0
fi
```

chmod +x /etc/keepalived/check_apiserver.sh

```sh
systemctl restart haproxy.service
systemctl restart keepalived.service
apt install kubeadm -y
```

## 集群初始化

Master01 节点创建 new.yaml 配置文件如下：

```sh
mkdir -p k8s && cd k8s
vim new.yaml
```

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
  - groups:
      - system:bootstrappers:kubeadm:default-node-token
    token: 7t2weq.bjbawausm0jaxury
    ttl: 24h0m0s
    usages:
      - signing
      - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.99.155
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master01
  taints:
    - effect: NoSchedule
      key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
    - 192.168.99.42
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: 192.168.99.42:16443
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.23.1
networking:
  dnsDomain: cluster.local
  podSubnet: 172.168.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

```sh
kubeadm config images pull --config /root/k8s/new.yaml
```

master01 节点生成初始化,初始化以后会在/etc/kubernetes 目录下生成对应的证书和配置文件，之后其他 Master 节点加入 Master01 即可

```sh
systemctl enable --now kubelet
kubeadm init --config /root/k8s/new.yaml  --upload-certs
```

初始化成功以后，会产生 Token 值，用于其他节点加入时使用，因此要记录下初始化成功生成的 token 值（令牌值）：

