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
