# RocketMQ 安装和启动

# RocketMQ 安装和部署
部署RocketMQ

## 单机安装构建
1. 安装JDK 1.8.0
```sh
yum install java-1.8.0-openjdk*
```

2. 安装Maven
```sh
wget http://dlcdn.apache.org/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.tar.gz
tar -zxvf apache-maven-3.8.4-bin.tar.gz
mv -f apache-maven-3.8.4 /usr/local/
vim /etc/profile
# 末尾添加
export MAVEN_HOME=/usr/local/apache-maven-3.3.9
export PATH=${PATH}:${MAVEN_HOME}/bin
# 保存
source /etc/profile
# 查看maven是否正常
mvn -v
```

## 快速部署
```sh
#构建 DLedger
git clone https://github.com/openmessaging/openmessaging-storage-dledger.git
cd openmessaging-storage-dledger
mvn clean install -DskipTests
# 构建 RocketMQ
git clone https://github.com/apache/rocketmq.git
cd rocketmq
git checkout -b store_with_dledger origin/store_with_dledger
mvn -Prelease-all -DskipTests clean install -U
# 部署
cd rocketmq/distribution/target/apache-rocketmq
sh bin/dledger/fast-try.sh start
# 通过 mqadmin 运维命令查看集群状态
sh bin/mqadmin clusterList -n 127.0.0.1:9876
```
