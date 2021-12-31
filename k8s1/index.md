# Kubernetes 实验手册（1）


# Kubernetes 实验手册（1）
通过在pve创建5台虚拟机：  

|节点|IP|
|:----|:----|
|node1|192.168.99.217|
|node2|192.168.99.224|
|node3|192.168.99.41|
|node4|192.168.99.219|

更新ansible连接
```sh
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.217
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.224
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.41
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.219
```

