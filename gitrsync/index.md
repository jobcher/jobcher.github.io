# gitlab与github同步项目

# gitlab与github同步项目
1. 本地同步项目
```sh
git clone
```
2. 创建一个同名的项目,命令行终端中添加remote地址
```sh
git remote add githubOrigin git@github.com:sjtfreaks/blog.git
```

3. 项目同步到Github上
```sh
git push -u githubOrigin main
```

4. 分别同步github与gitlab即可
```sh
git push -u githubOrigin main
git push -u origin main
```
