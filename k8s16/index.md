# Kubernetes — 更新证书


## 背景

使用 `kubeadm` 安装 `kubernetes` 集群非常方便，但是也有一个比较烦人的问题就是默认的证书有效期只有`一年`时间，所以需要考虑`证书升级`的问题

## 检查证书

由 kubeadm 生成的客户端证书默认只有一年有效期，我们可以通过 check-expiration 命令来检查证书是否过期：

```sh
kubeadm alpha certs check-expiration
```

该命令显示 `/etc/kubernetes/pki` 文件夹中的客户端证书以及 `kubeadm` 使用的 `KUBECONFIG` 文件中嵌入的客户端证书的`到期时间`/`剩余时间`。

## 手动更新

> kubeadm alpha certs renew  
> 这个命令用 CA（或者 front-proxy-CA ）证书和存储在 `/etc/kubernetes/pki` 中的密钥执行更新。  
> 高可用的集群，这个命令需要在所有控制面板节点上执行

### 具体执行

接下来我们来更新我们的集群证书，下面的操作都是在 master 节点上进行

1. 备份节点

```sh
$ mkdir /etc/kubernetes.bak
$ cp -r /etc/kubernetes/pki/ /etc/kubernetes.bak
$ cp /etc/kubernetes/*.conf /etc/kubernetes.bak
```

2. 备份 etcd 数据目录

```sh
$ cp -r /var/lib/etcd /var/lib/etcd.bak
```

3. 执行更新证书的命令

```sh
kubeadm alpha certs renew all --config=kubeadm.yaml
```

4. 检查更新

```sh
kubeadm alpha certs check-expiration
```

5. 更新下 kubeconfig 文件

```sh
kubeadm init phase kubeconfig all --config kubeadm.yaml
```

6. 覆盖掉原本的 admin 文件

```sh
$ mv $HOME/.kube/config $HOME/.kube/config.old
$ cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ chown $(id -u):$(id -g) $HOME/.kube/config
```

7. 重启 kube-apiserver、kube-controller、kube-scheduler、etcd ,查看 apiserver 的证书的有效期

```sh
echo | openssl s_client -showcerts -connect 127.0.0.1:6443 -servername api 2>/dev/null | openssl x509 -noout -enddate
```

## 总结

可以看到现在的有效期是一年过后的，证明已经更新成功了。

