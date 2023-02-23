# Kubernetes — 探针和生命周期


# Kubernetes — 探针和生命周期

用于判断容器内应用程序是否已经启动。

- 存活（Liveness）探针
  - 用于探测容器是否运行，如果探测失败，kubelet 会根据配置的重启策略进行相应的处理，若没有配置探针该返回值默认为 success
- 就绪（Readiness）探针
  - 用于探测容器内的程序是否健康，如果返回值为 success，那么代表这个容器已经完全启动，并且程序已经是可以接受流量的状态
- 启动（Startup）探针
  - 用于探测容器是否启动，如果配置了 startup 就会先禁止其他探测，直到它成功，成功后将不在运行探测

## Pod 检测方式

- ExecAction：在容器执行一个命令，返回值为 0，则认为容器健康
- TCPSocketAction：通过 TCP 连接检查容器是否联通，通的话，则认为容器正常
- HTTPGetAction：通过应用程序暴露的 API 地址来检查程序是否正常的，如果状态码为 200-400 之间，则认为容器健康
- gRPCAction：通过 gRPC 的检查机制，判断容器是不是正常

## StartupProbe 启动探针

有时候，会有一些现有的应用在启动时需要`较长的初始化时间`。 要这种情况下，若要不影响对`死锁`作出快速响应的探测，设置存活探测参数是要技巧的。 技巧就是使用相同的命令来设置启动探测，针对 HTTP 或 TCP 检测，可以通过将 `failureThreshold * periodSeconds` 参数设置为足够长的时间来应对糟糕情况下的启动时间。

```yaml
ports:
  - name: liveness-port
    containerPort: 8080
    hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```

幸亏有启动探测，应用程序将会有最多 `5 分钟（30 * 10 = 300s）`的时间来完成其启动过程。 一旦启动探测成功一次，存活探测任务就会接管对容器的探测，对容器死锁作出快速响应。 如果启动探测一直没有成功，容器会在 300 秒后被杀死，并且根据`restartPolicy`来执行进一步处置。

## LivenessProbe 存活探针

### ExecAction

许多长时间运行的应用最终会进入损坏状态，除非重新启动，否则无法被恢复。 `Kubernetes` 提供了存活探针来发现并处理这种情况。  
在本练习中，你会创建一个 `Pod`，其中运行一个基于 `registry.k8s.io/busybox` 镜像的容器。 下面是这个 `Pod` 的配置文件。

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
    - name: liveness
      image: registry.k8s.io/busybox
      args:
        - /bin/sh
        - -c
        - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
      livenessProbe:
        exec:
          command:
            - cat
            - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 5
```

在这个配置文件中，可以看到 `Pod` 中只有一个 `Container`。 `periodSeconds` 字段指定了 `kubelet` 应该每 `5` 秒执行一次存活探测。 `initialDelaySeconds` 字段告诉 `kubelet` 在执行第一次探测前应该等待 `5` 秒。 kubelet 在容器内执行命令 `cat /tmp/healthy` 来进行探测。 如果命令执行成功并且返回值为 `0`，`kubelet` 就会认为这个容器是健康存活的。 如果这个命令返回`非 0` 值，`kubelet` 会杀死这个容器并重新启动它。

当容器启动时，执行如下的命令：

> /bin/sh -c "touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600"

这个容器生命的前 30 秒，`/tmp/healthy` 文件是存在的。 所以在这最开始的 30 秒内，执行命令 `cat /tmp/healthy` 会返回成功代码。 30 秒之后，执行命令 `cat /tmp/healthy` 就会返回失败代码。  
在 30 秒内，查看 Pod 的事件：

> kubectl describe pod liveness-exec

### HTTPGetAction

另外一种类型的存活探测方式是使用 `HTTP GET` 请求。 下面是一个 `Pod` 的配置文件，其中运行一个基于 `registry.k8s.io/liveness` 镜像的容器。

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
    - name: liveness
      image: registry.k8s.io/liveness
      args:
        - /server
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
          httpHeaders:
            - name: Custom-Header
              value: Awesome
        initialDelaySeconds: 3
        periodSeconds: 3
```

在这个配置文件中，你可以看到 `Pod` 也只有一个容器。 `periodSeconds` 字段指定了 `kubelet` 每隔 3 秒执行一次存活探测。 `initialDelaySeconds` 字段告诉 `kubelet` 在执行第一次探测前应该等待 3 秒。 `kubelet` 会向容器内运行的服务（服务在监听 `8080 端口`）发送一个 `HTTP GET` 请求来执行探测。 如果服务器上 `/healthz` 路径下的处理程序返回成功代码，则 `kubelet` 认为容器是健康存活的。 如果处理程序返回失败代码，则 `kubelet` 会杀死这个容器并将其重启。

返回大于或等于 `200` 并且小于 `400` 的任何代码都标示成功，其它返回代码都标示失败。

你可以访问 `server.go`。 阅读服务的源码。 容器存活期间的最开始 `10 秒中`，`/healthz` 处理程序返回 `200` 的状态码。 之后处理程序返回 `500` 的状态码。

### TCPSocketAction

第三种类型的存活探测是使用 `TCP 套接字`。 使用这种配置时，`kubelet` 会尝试在指定端口和容器建立套接字链接。 如果`能建立连接`，这个容器就被看作是`健康`的，如果`不能`则这个容器就被看作是`有问题`的。

```yml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
    - name: goproxy
      image: registry.k8s.io/goproxy:0.1
      ports:
        - containerPort: 8080
      readinessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
      livenessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
```

如是汝闻，`TCP 检测`的配置和 `HTTP 检测`非常相似。 下面这个例子同时使用就绪和存活探针。`kubelet` 会在容器启动 `5 秒`后发送第一个就绪探针。 探针会尝试连接 `goproxy` 容器的 8080 端口。 如果探测成功，这个 `Pod` 会被标记为就绪状态，kubelet 将继续每隔 `10 秒`运行一次探测。

除了就绪探针，这个配置包括了一个存活探针。 `kubelet` 会在容器启动 `15 秒`后进行第一次存活探测。 与就绪探针类似，存活探针会尝试连接 `goproxy 容器`的 `8080 端口`。 如果存活探测失败，容器会被重新启动。

### gRPCAction

如果你的应用实现了 gRPC 健康检查协议， `kubelet` 可以配置为使用该协议来执行应用存活性检查。 你必须启用 `GRPCContainerProbe` 特性门控 才能配置依赖于 `gRPC 的检查机制`。

```yml
apiVersion: v1
kind: Pod
metadata:
  name: etcd-with-grpc
spec:
  containers:
    - name: etcd
      image: registry.k8s.io/etcd:3.5.1-0
      command:
        [
          "/usr/local/bin/etcd",
          "--data-dir",
          "/var/lib/etcd",
          "--listen-client-urls",
          "http://0.0.0.0:2379",
          "--advertise-client-urls",
          "http://127.0.0.1:2379",
          "--log-level",
          "debug",
        ]
      ports:
        - containerPort: 2379
      livenessProbe:
        grpc:
          port: 2379
        initialDelaySeconds: 10
```

要使用 `gRPC 探针`，必须`配置 port 属性`。如果健康状态端点配置在非默认服务之上， 你还必须设置 `service 属性`。

在 `Kubernetes 1.23` 之前，`gRPC 健康探测`通常使用 `grpc-health-probe` 来实现，如博客 `Health checking gRPC servers on Kubernetes`（对 `Kubernetes` 上的 `gRPC 服务器`执行健康检查）所描述。 内置的 `gRPC 探针`行为与 `grpc-health-probe` 所实现的行为类似。 从 `grpc-health-probe` 迁移到内置探针时，请注意以下差异：

内置探针运行时针对的是 Pod 的 IP 地址，不像 `grpc-health-probe` 那样通常针对 127.0.0.1 执行探测； 请一定配置你的 gRPC 端点使之监听于 Pod 的 IP 地址之上。
内置探针不支持任何身份认证参数（例如 -tls）。
对于内置的探针而言，不存在错误代码。所有错误都被视作探测失败。
如果 `ExecProbeTimeout` 特性门控被设置为 `false`，则 `grpc-health-probe` 不会考虑 `timeoutSeconds` 设置状态（默认值为 1s）， 而内置探针则会在超时时返回失败。

## ReadinessProbe 就绪探针

有时候，应用会暂时性地无法为请求提供服务。 例如，应用在启动时可能需要加载大量的数据或配置文件，或是启动后要依赖等待外部服务。 在这种情况下，`既不想杀死应用`，也不想给它发送请求。 `Kubernetes` 提供了就绪探针来发现并缓解这些情况。 容器所在 Pod 上报还未就绪的信息，并且不接受通过 `Kubernetes Service` 的流量。

```yml
readinessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

