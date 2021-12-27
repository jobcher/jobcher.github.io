# rsync 文件同步


# rsync 文件同步
rsync 是一个常用的Linux应用程序，用于文件同步
## 安装
```sh
# Debian or Ubuntu
$ sudo apt-get install rsync

# Red Hat
$ sudo yum install rsync

# Arch Linux
$ sudo pacman -S rsync
```

## 基本用法
使用 rsync 命令时，可以作为cp和mv命令的替代方法，将源目录同步到目标目录。  
-r 表示递归，即包含子目录。注意，-r是必须的，否则 rsync 运行不会成功。source目录表示源目录，destination表示目标目录。  
-a 参数可以替代-r，除了可以递归同步以外，还可以同步元信息（比如修改时间、权限等）。由于 rsync 默认使用文件大小和修改时间决定文件是否需要更新  
```sh
rsync -r source destination
```
远程同步
```sh
rsync -av -e 'ssh -p 2234' source/ user@remote_host:/destination
```


