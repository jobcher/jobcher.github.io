# kubernetes 存储


# kubernetes 存储

k8s 支持多种途径的多种类型的存储。例如 iSCSI,SMB,NFS，以及对象存储。都是不同类型的部署在云上或者自建数据中心的外部存储系统。k8s 上的所有存储都被称作`卷`

## CSI 容器存储接口

CSI 是 k8s 存储体系中一部分，是一个开源项目，定义了一套基于标准的接口，从而使容器能够以一种统一的方式被不同的容器编排的工具使用。可以将插件称为`provisioner`

## 持久化

- 持久化卷 （pv）
- 持久化卷申请 （pvc）
- 存储类 （sv）

PV 代表 k8s 的存储，pvc 代表的是许可证，赋予 pod 访问 pv 的权限。cs 使分配过程是动态的。

## 使用 iSCSI 操作存储

`iscsi` 卷能将 iSCSI (基于 IP 的 SCSI) 卷挂载到你的 Pod 中。 不像 emptyDir 那样会在删除 Pod 的同时也会被删除，iscsi 卷的内容在删除 Pod 时会被保留，卷只是被卸载。 这意味着 iscsi 卷可以被预先填充数据，并且这些数据可以在 Pod 之间共享。  
`iSCSI` 的一个特点是它可以同时被多个用户以只读方式挂载。 这意味着你可以用数据集预先填充卷，然后根据需要在尽可能多的 Pod 上使用它。 不幸的是，iSCSI 卷只能由单个使用者以读写模式挂载。不允许同时写入。

### 创建 iscsi-pv.yaml iscsi-pvc.yaml

iscsi-pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: iscsi-pv
spec:
  capacity:
    storage: 500Gi
  accessModes:
    - ReadWriteOnce
  iscsi:
    targetPortal: 10.12.12.xxx:3260 # 修改
    iqn: iqn.2000-01.com.synology:xxx.Target-1.21xxxxx344 # 修改
    lun: 1
```

iscsi-pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: iscsi-pv
spec:
  capacity:
    storage: 500Gi
  accessModes:
    - ReadWriteOnce
  iscsi:
    targetPortal: 10.12.12.xxx:3260 # 修改
    iqn: iqn.2000-01.com.synology:xxx.Target-1.21xxxxx344 # 修改
    lun: 1
```

## hostPath

`hostPath` 卷能将主机节点文件系统上的文件或目录挂载到你的 `Pod` 中。 虽然这不是大多数 `Pod` 需要的，但是它为一些应用程序提供了强大的逃生舱。

- 运行一个需要访问 Docker 内部机制的容器；可使用 `hostPath` 挂载 `/var/lib/docker` 路径。
- 在容器中运行 `cAdvisor` 时，以 `hostPath` 方式挂载 `/sys`。
- 允许 `Pod` 指定给定的 `hostPath` 在运行 `Pod` 之前是否应该存在，是否应该创建以及应该以什么方式存在。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
    - image: registry.k8s.io/test-webserver
      name: test-container
      volumeMounts:
        - mountPath: /test-pd
          name: test-volume
  volumes:
    - name: test-volume
      hostPath:
        # 宿主上目录位置
        path: /data
        # 此字段为可选
        type: Directory
```

> 注意： `FileOrCreate` 模式不会负责创建文件的父目录。 如果欲挂载的文件的父目录不存在，Pod 启动会失败。 为了确保这种模式能够工作，可以尝试把文件和它对应的目录分开挂载，如 `FileOrCreate` 配置 所示。

### hostPath FileOrCreate 配置示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-webserver
spec:
  containers:
    - name: test-webserver
      image: registry.k8s.io/test-webserver:latest
      volumeMounts:
        - mountPath: /var/local/aaa
          name: mydir
        - mountPath: /var/local/aaa/1.txt
          name: myfile
  volumes:
    - name: mydir
      hostPath:
        # 确保文件所在目录成功创建。
        path: /var/local/aaa
        type: DirectoryOrCreate
    - name: myfile
      hostPath:
        path: /var/local/aaa/1.txt
        type: FileOrCreate
```

