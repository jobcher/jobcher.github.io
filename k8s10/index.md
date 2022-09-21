# kubernetes 存储

# kubernetes 存储
k8s支持多种途径的多种类型的存储。例如iSCSI,SMB,NFS，以及对象存储。都是不同类型的部署在云上或者自建数据中心的外部存储系统。k8s上的所有存储都被称作`卷`  
## CSI 容器存储接口
CSI是k8s存储体系中一部分，是一个开源项目，定义了一套基于标准的接口，从而使容器能够以一种统一的方式被不同的容器编排的工具使用。可以将插件称为`provisioner`
## 持久化
- 持久化卷 （pv）
- 持久化卷申请 （pvc）
- 存储类 （sv）
  
PV 代表k8s的存储，pvc代表的是许可证，赋予pod访问pv的权限。cs使分配过程是动态的。

## 使用iSCSI操作存储
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
