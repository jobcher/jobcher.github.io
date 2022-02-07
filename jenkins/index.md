# Jenkins 安装与使用


# Jenkins 安装与使用

## 安装docker
[安装docker](https://www.jobcher.com/docker/)

## docker安装jenkins

```sh
docker run \
  -u root \
  -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins-data:/var/jenkins_home \
  -v /etc/localtime:/etc/localtime:ro \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --restart=always \
  jenkinsci/blueocean
```

## 访问
http://localhost:8080  
  
显示初始密码
```sh
docker exec -ti <容器名称> sh
cat /var/jenkins_home/secrets/initialAdminPassword
```

## 工作流程
1. 先定义一个流水线项目，指定项目的git位置
2. git位置自动拉取代码
3. 解析拉取代码里面的Jenkinsfile文件
4. 按照Jenkinsfile指定的流水线开始加工项目

## Jenkins语法
基础语法
```json
pipeline {
    /* 全部的CICD流程都在这里定义 */

    //任意代理可用就可以执行
    agent any
    //定义流水线的加工流程
    stages {
        /* 流水线的所有阶段
            1.编译 "常量"'变量'
            2.测试
            3.打包
            4.部署
        */

        stage('代码编译'){
            steps {
                //要做的所有事情
                echo "编译……"
            }
        }

        stage('代码测试'){
            steps {
                //要做的所有事情
                echo "测试……"
            }
        }

        stage('打包'){
            steps {
                //要做的所有事情
                echo "打包……"
            }
        }

        stage('部署'){
            steps {
                //要做的所有事情
                echo "部署……"
            }
        }
    }
}
```

