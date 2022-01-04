# Kubernetes 实验手册（1）


# Kubernetes 实验手册（1）
通过在pve创建5台虚拟机：  

|节点|IP|作用|
|:----|:----|:----|
|node1|192.168.99.217|k8s-master01|
|node2|192.168.99.224|k8s-master02|
|node3|192.168.99.41|k8s-node01|
|node4|192.168.99.219|k8s-node02|
|node5|192.168.99.155|k8s-lb01|

1. 更新ansible连接  
```sh
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.217
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.224
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.41
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.219
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.155
```

2. Keepalived 安装
Keepalived高可用  
```sh

```


