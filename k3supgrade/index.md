# k3s 升级版本


# k3s 升级版本

## 停止所有的 K3s 容器（慎用）

从 server 节点运行 killall 脚本

```sh
/usr/local/bin/k3s-killall.sh
```

## 开始升级

1. 使用安装脚本升级 K3s

```sh
curl -sfL https://get.k3s.io | sh -
#国内可用
curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -
```

2. 重启 k3s

```sh
sudo systemctl restart k3s
```

