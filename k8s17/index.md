# Kubernetes — kubecost 分析 Kubernetes 成本

## 简介
企业在上云之后，云计算基础设施支出不断创造新高，但 IT 团队却难以找到成本失控的源头，跟每一个业务沟通，所需要的资源都是必须的，降本增效无从谈起。  
引入`FinOps` 的目标是在云上创造一种财务问责制度，每个业务团队需要根据 FinOps 团队的数据做出更加合理的配置、规划，从而在财务成本、业务稳定之间找到一种平衡。FinOps 并不是一次性、短暂的任务，而是在规划实施之后依旧需要进行持续管理，这要求企业必须设定明确的、持续的角色和责任，以保持对成本长期控制。  
### 概念
![概念](/images/finops-1a566da4.jpg)  
- `建立对云成本的共识`：企业中各个相关角色应该意识到云成本的重要性，并将成本管理纳入到决策过程中。通过提高成本意识，可以更好地控制和优化云资源的使用。
- `明确云成本管理的责任和角色`：确定负责 FinOps 团队成员，建立相应责任制度。这样确保有专门人员负责云成本的监控、分析和优化，从而提高整体的财务管理效果。
- `提供培训和教育资源`：培训企业成员了解成本管理的基本概念、工具和技术。这有助于增强团队的能力，使他们能够更好地理解和应对云成本挑战。
- `促进不同团队之间的合作`：财务团队、开发团队和运维团队应该紧密合作，共同制定和实施成本管理策略。通过协作，可以更好地理解业务需求、优化资源配置，并确保成本管理策略与业务目标相一致。
- `利用自动化技术提高效率和准确性`：通过采用自动化工具收集、分析和报告云成本数据。自动化还可以帮助实现实时监控和警报，以及自动化资源管理，从而提高成本管理的效率和准确性。
## 使用 kubecost 分析 Kubernetes 成本
接下来我们展开今天的具体内容，如何使用 kubecost 分析 Kubernetes 成本。  
kubecost 是目前较优秀的开源 Kubernetes 成本分析工具，它提供了丰富的功能和仪表板，帮助用户更好地理解和控制其容器化工作负载的成本。  
kubecost 目前支持 `阿里云`、`AWS` 等云厂商对接，它能够提供集群中命名空间、应用等各类资源成本分配，用户还可以基于这些信息在 Kubecost 中设置`预算`和`警报`，帮助运维和财务管理人员进一步实现`成本管理`。  
### 安装 Kubecost
安装 Kubecost 建议使用 Helm 进行安装，使用以下命令：
```sh
helm repo add kubecost https://kubecost.github.io/cost-analyzer/
helm repo update
helm upgrade --install kubecost kubecost/cost-analyzer --namespace kubecost --create-namespace
```
几分钟后，检查以确保 Kubecost 已启动并运行：
```sh
kubectl get pods -n kubecost
# Connect to the Kubecost dashboard UI
kubectl port-forward -n kubecost svc/kubecost-cost-analyzer 9090:9090
```
现在可以打开浏览器并指向 `http://127.0.0.1:9090`以打开 `Kubecost UI`。 在 Kubecost UI 中，选择群集以查看`成本分配信息`。  
![kubecost](/images/kubecost-215b96ad.png)  
### kubecost 成本统计原理
#### CPU/内存/GPU/存储分析
Kubecost 通过 AWS/GCP 等云服务商 API 动态获取各 region/zone 的上述四项资源的每小时成本，或者通过 json 文件静态配置这几项资源的成本。 kubecost 的成本统计粒度为 container，kubecost 根据每个容器的资源请求 requests 以及资源用量监控进行成本分配，对于未配置 requests 的资源将仅按实际用量监控进行成本分配。  
#### 网络成本分析
对于提供线上服务的 Kubernetes 集群，网络成本（跨区/跨域传输的流量成本，以及 NAT 网关成本）很可能等于甚至超过计算成本。 kubecost 支持使用 Pod network 监控指标对整个集群的流量成本进行拆分，kubecost 会部署一个绑定 hostNetwork 的 daemonset 来采集需要的网络指标，提供给 prometheus 拉取，再进行进一步的分析。  


