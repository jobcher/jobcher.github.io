# Kubernetes — 探针和生命周期

# Kubernetes — 探针和生命周期
用于判断容器内应用程序是否已经启动。
- 存活（Liveness）探针
    - 用于探测容器是否运行，如果探测失败，kubelet会根据配置的重启策略进行相应的处理，若没有配置探针该返回值默认为success
- 就绪（Readiness）探针
    - 用于探测容器内的程序是否健康，如果返回值为success，那么代表这个容器已经完全启动，并且程序已经是可以接受流量的状态
- 启动（Startup）探针
    - 用于探测容器是否启动，如果配置了startup 就会先禁止其他探测，直到它成功，成功后将不在运行探测

## Pod检测方式
- ExecAction：在容器执行一个命令，返回值为0，则认为容器健康
- TCPSocketAction：通过TCP连接检查容器是否联通，通的话，则认为容器正常
- HTTPGetAction：通过应用程序暴露的API地址来检查程序是否正常的，如果状态码为200-400之间，则认为容器健康

## StartupProbe 启动探针

--collector.scheduled_task.whitelist
