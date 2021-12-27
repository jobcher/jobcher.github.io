# k8s 部署loki日志

# k8s 部署loki日志

## helm 拉取loki
```sh
#加源
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
#拉取
helm fetch grafana/loki-stack --untar --untardir .
cd loki-stack
# 生成 k8s 配置
helm template loki . > loki.yaml
# 部署（如果要修改默认配置必须要修改一下yaml）
k3s kubectl apply -f loki.yaml
```
