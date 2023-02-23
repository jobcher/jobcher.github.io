# Kubernetes — Rook云存储介绍和部署


# Rook 云存储介绍和部署

`Rook `将`分布式存储软件`转变为`自我管理`，`自我缩放`和`自我修复`的`存储服务`。它通过自动化部署，引导、配置、供应、扩展、升级、迁移、灾难恢复、监控和资源管理来实现。 `Rook` 使用基础的云原生容器管理、调度和编排平台提供的功能来履行其职责。

`Rook` 利用扩展点深入融入`云原生环境`，为调度、生命周期管理、资源管理、安全性、监控和用户体验提供无缝体验。

## 部署

### 使用 helm 部署

```sh
helm init -i jimmysong/kubernetes-helm-tiller:v2.8.1
helm repo add rook-alpha https://charts.rook.io/alpha
helm install rook-alpha/rook --name rook --namespace rook-system
```

### 直接使用 yaml 文件部署

```sh
kubectl apply -f rook-operator.yaml
```

不论使用那种方式部署的 rook operator，都会在 rook-agent 中看到 rook-agent 用户无法列出集群中某些资源的错误，可以通过为 rook-agent 的分配 cluster-admin 权限临时解决，详见 [Issue 1472](https://github.com/rook/rook/issues/1472)。

使用如下 yaml 文件创建一个 `ClusterRoleBinding` 并应用到集群中。

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: rookagent-clusterrolebinding
subjects:
  - kind: ServiceAccount
    name: rook-agent
    namespace: rook-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: ""
```

### 部署 rook cluster

创建完 `rook operator` 后，我们再部署 `rook cluster`。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: rook
---
apiVersion: rook.io/v1alpha1
kind: Cluster
metadata:
  name: rook
  namespace: rook
spec:
  versionTag: v0.6.2
  dataDirHostPath: /var/lib/rook
  storage:
    useAllNodes: true
    useAllDevices: false
    storeConfig:
      storeType: bluestore
      databaseSizeMB: 1024
      journalSizeMB: 1024
```

> `注意`：需要手动指定 `versionTag`，因为该镜像 `repo` 中没有 `latest` 标签，如不指定的话 Pod 将出现镜像拉取错误。

```sh
kubectl apply -f rook-cluster.yaml
```

rook 集群运行在`rook` namespace 下，查看 rook 集群中的 pod：

```sh
$ kubectl -n rook get pod
NAME                             READY     STATUS    RESTARTS   AGE
rook-api-848df956bf-q6zf2        1/1       Running   0          4m
rook-ceph-mgr0-cfccfd6b8-cpk5p   1/1       Running   0          4m
rook-ceph-mon0-t496l             1/1       Running   0          6m
rook-ceph-mon1-zcn7v             1/1       Running   0          5m
rook-ceph-mon3-h97qx             1/1       Running   0          3m
rook-ceph-osd-557tn              1/1       Running   0          4m
rook-ceph-osd-74frb              1/1       Running   0          4m
rook-ceph-osd-zf7rg              1/1       Running   1          4m
rook-tools                       1/1       Running   0          2m
```

### 部署 StorageClass

StorageClass `rook-block` 的 yaml 文件（rook-storage.yaml）如下：

```yaml
apiVersion: rook.io/v1alpha1
kind: Pool
metadata:
  name: replicapool
  namespace: rook
spec:
  replicated:
    size: 1
  # For an erasure-coded pool, comment out the replication size above and uncomment the following settings.
  # Make sure you have enough OSDs to support the replica size or erasure code chunks.
  #erasureCoded:
  #  dataChunks: 2
  #  codingChunks: 1
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-block
provisioner: rook.io/block
parameters:
  pool: replicapool
  # Specify the Rook cluster from which to create volumes.
  # If not specified, it will use `rook` as the name of the cluster.
  # This is also the namespace where the cluster will be
  clusterName: rook
  # Specify the filesystem type of the volume. If not specified, it will use `ext4`.
  # fstype: ext4
```

## 工具

部署 `Rook` 操作工具 pod，该工具 pod 的 yaml 文件（`rook-tools.yaml`）如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rook-tools
  namespace: rook-system
spec:
  dnsPolicy: ClusterFirstWithHostNet
  serviceAccountName: rook-operator
  containers:
    - name: rook-tools
      image: rook/toolbox:master
      imagePullPolicy: IfNotPresent
      env:
        - name: ROOK_ADMIN_SECRET
          valueFrom:
            secretKeyRef:
              name: rook-ceph-mon
              key: admin-secret
      securityContext:
        privileged: true
      volumeMounts:
        - mountPath: /dev
          name: dev
        - mountPath: /sys/bus
          name: sysbus
        - mountPath: /lib/modules
          name: libmodules
        - name: mon-endpoint-volume
          mountPath: /etc/rook
  hostNetwork: false
  volumes:
    - name: dev
      hostPath:
        path: /dev
    - name: sysbus
      hostPath:
        path: /sys/bus
    - name: libmodules
      hostPath:
        path: /lib/modules
    - name: mon-endpoint-volume
      configMap:
        name: rook-ceph-mon-endpoints
        items:
          - key: endpoint
            path: mon-endpoints
```

> `ConfigMap` 和 `Secret` 中的配置项内容是自定义的。
> 使用下面的命令部署工具 `pod`：

```sh
kubectl apply -f rook-tools.yaml
```

这是一个独立的 pod，没有使用其他高级的 `controller` 来管理，我们将它部署在 `rook-system` 的 `namespace` 下。

```sh
kubectl -n rook exec -it rook-tools bash
```

使用下面的命令查看 `rook` 集群状态。

```sh
$ rookctl status
OVERALL STATUS: OK

USAGE:
TOTAL       USED       DATA      AVAILABLE
37.95 GiB   1.50 GiB   0 B       36.45 GiB

MONITORS:
NAME             ADDRESS                IN QUORUM   STATUS
rook-ceph-mon0   10.254.162.99:6790/0   true        UNKNOWN

MGRs:
NAME             STATUS
rook-ceph-mgr0   Active

OSDs:
TOTAL     UP        IN        FULL      NEAR FULL
1         1         1         false     false

PLACEMENT GROUPS (0 total):
STATE     COUNT
none

$ ceph df
GLOBAL:
    SIZE       AVAIL      RAW USED     %RAW USED
    38861M     37323M        1537M          3.96
POOLS:
    NAME     ID     USED     %USED     MAX AVAIL     OBJECTS
```

## 示例

官方提供了使用 Rook 作为典型的 LAMP（`Linux + Apache + MySQL + PHP`）应用 Wordpress 的存储后端的示例的 yaml 文件 `mysql.yaml` 和 `wordpress.yaml`，使用下面的命令创建。

```sh
kubectl apply -f mysql.yaml
kubectl apply -f wordpress.yaml
```

## 清理

如果使用 `helm` 部署，则执行下面的命令：

```sh
helm delete --purge rook
helm delete daemonset rook-agent
```

如果使用 `yaml` 文件直接部署，则使用 `kubectl delete -f` 加当初使用的 `yaml` 文件即可删除集群。

