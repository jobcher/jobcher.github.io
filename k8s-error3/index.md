# k8s.gcr.io国内无法连接解决方法


# k8s.gcr.io国内无法连接解决方法
Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)  
这个一看知道什么原因了，应该GFW！那好吧，只能给docker加个代理了。  

## 解决问题
添加mirror站点
```sh
registry.cn-hangzhou.aliyuncs.com/google_containers
```
