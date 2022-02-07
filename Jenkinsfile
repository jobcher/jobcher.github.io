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
                sh 'docker pull hub.jobcher.com/blog/hugo:latest'
                sh 'docker run -d --name bloghugo -p 8020:80 hub.jobcher.com/blog/hugo:latest'
                echo "编译完成"
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
