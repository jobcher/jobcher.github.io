# 编写 kubernetes 资源描述文件


# 编写 kubernetes 资源描述文件

### 1. 部署一个应用

```yaml
apiVersion: apps/v1 #与k8s集群版本有关，使用 kubectl api-versions 即可查看当前集群支持的版本
kind: Deployment #该配置的类型，我们使用的是 Deployment
metadata: #译名为元数据，即 Deployment 的一些基本属性和信息
  name: nginx-deployment #Deployment 的名称
  labels: #标签，可以灵活定位一个或多个资源，其中key和value均可自定义，可以定义多组，目前不需要理解
    app: nginx #为该Deployment设置key为app，value为nginx的标签
spec: #这是关于该Deployment的描述，可以理解为你期待该Deployment在k8s中如何使用
  replicas: 1 #使用该Deployment创建一个应用程序实例
  selector: #标签选择器，与上面的标签共同作用，目前不需要理解
    matchLabels: #选择包含标签app:nginx的资源
      app: nginx
  template: #这是选择或创建的Pod的模板
    metadata: #Pod的元数据
      labels: #Pod的标签，上面的selector即选择包含标签app:nginx的Pod
        app: nginx
    spec: #期望Pod实现的功能（即在pod中部署）
      containers: #生成container，与docker中的container是同一种
        - name: nginx #container的名称
          image: nginx:1.7.9 #使用镜像nginx:1.7.9创建container，该container默认80端口可访问
```

> kubectl apply -f xxx.yaml

### 2、暴露应用

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service #Service 的名称
  labels: #Service 自己的标签
    app: nginx #为该 Service 设置 key 为 app，value 为 nginx 的标签
spec: #这是关于该 Service 的定义，描述了 Service 如何选择 Pod，如何被访问
  selector: #标签选择器
    app: nginx #选择包含标签 app:nginx 的 Pod
  ports:
    - name: nginx-port #端口的名字
      protocol: TCP #协议类型 TCP/UDP
      port: 80 #集群内的其他容器组可通过 80 端口访问 Service
      nodePort: 32600 #通过任意节点的 32600 端口访问 Service
      targetPort: 80 #将请求转发到匹配 Pod 的 80 端口
  type: NodePort #Serive的类型，ClusterIP/NodePort/LoaderBalancer
```

### 3、扩缩容

修改 deployment.yaml 中的 replicas 属性即可

完成后运行 `kubectl apply -f xxx.yaml`

### 4、滚动升级

修改 deployment.yaml 中的 imageName 属性等

完成后运行 `kubectl apply -f xxx.yaml`

