# Kubernetes 实验手册（1）


# Kubernetes 实验手册（1）
通过在pve创建5台虚拟机：  

|节点|IP|作用|
|:----|:----|:----|
|node0|192.168.99.155|k8s-master01|
|node1|192.168.99.199|k8s-master02|
|node2|192.168.99.87|k8s-master03|
|node3|192.168.99.41|k8s-node01|
|node4|192.168.99.219|k8s-node02|
|node5|192.168.99.42|k8s-master-lb|

|配置信息|备注|
|:----|:----|
|系统版本|Ubuntu|
|Docker|20.10.12|
|pod网段|172.168.0.0/12|
|service网段|10.96.0.0/12|

VIP 不要和内网IP重复，VIP需要和主机在同一个局域网内  


1. 更新ansible连接  
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
set-timezone 'Asia/Shanghai'
timedatectl
date
```
2. 安装docker
```sh
curl -sSL https://get.daocloud.io/docker | sh
systemctl restart docker
```
3. 安装k8s组件
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

4. 安装keepalived和haproxy
所有Master节点安装HAProxy和KeepAlived
```sh
apt install keepalived haproxy -y
vim /etc/haproxy/haproxy.cfg 
```
所有Master节点的HAProxy配置相同

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
所有Master节点配置KeepAlived，配置不一样，注意区分  
注意每个节点的IP和网卡（interface参数）
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
```sh
systemctl restart haproxy.service
systemctl restart keepalived.service
```


## 集群初始化
Master01节点创建new.yaml配置文件如下：  
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
kubernetesVersion: v1.23.0
networking:
  dnsDomain: cluster.local
  podSubnet: 172.168.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}

```

```sh
kubeadm config images pull --config /root/k8s/new.yaml 
```
master01 节点生成初始化,初始化以后会在/etc/kubernetes目录下生成对应的证书和配置文件，之后其他Master节点加入Master01即可
```sh
kubeadm init --config /root/k8s/new.yaml  --upload-certs
```
初始化成功以后，会产生Token值，用于其他节点加入时使用，因此要记录下初始化成功生成的token值（令牌值）：  


