# 优雅的使用Conda管理python环境

## 背景
{{< admonition type=tip title="摘要">}}
很多时候,避免不了同时使用python2和python3的环境,也避免不了不同的工作所需要不同版本的库文件,比如在想用TensorFlow较早版本的同时;还想运行Pytorch最新版；还想顺便学习Nao机器人编程,学习Django后台,这个时候,一款非常好用的包管理工具就显得十分重要了,这就是我写这篇博客的原因,这篇博客将会讲解：
- 如何安装conda; 
- 如何更换conda的下载源; 
- 如何使用canda;
{{< /admonition >}}
## 安装conda
在安装时这两个选项需要点上：  
![conda_install](/images/conda_install.png)  
  
### 更换conda的下载源
`Conda官方`的下载源太慢了,而且经常会出现HTTPERROR之类的错误,如果想要用Conda愉快的创建不同工作环境,愉快的下载安装各种库,那么换下载源是必不可少的  
```sh 
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/

conda config --set show_channel_urls yes
# 设置搜索时显示通道地址
```
具体操作同时按Win+R键打开运行窗口,输入cmd,回车：  
![cmd-conda](/images/cmd-conda.png)  
  
将上面的命令全部复制,到命令行里单击右键就会自动执行复制的命令,添加清华源  
  
## 使用conda
查看环境  
```sh
conda info -e
conda info --envs
```
  
创建环境
```sh
conda create -n name python=3.6
# name参数指定虚拟环境的名字,python参数指定要安装python的版本,但注意至少需要指定python版本或者要安装的包,在后一种情况下,自动安装最新python版本
# 例如
conda create -n jobcher pillow numpy python=2.7.14
# 创建名字为naoqi,Python版本为2.7.14的虚拟环境,同时还会安装上pillow numpy这两个库
```
环境切换
```sh
conda activate jobcher
# 切换到jobcher环境下,在切换环境后,所执行的Pip命令,Python命令,都是更改当前环境下的,不会影响到其他的环境
conda deactivate
# 退出当前环境
```
{{< admonition type=success title="关注">}}
欢迎关注我的博客[www.jobcher.com](https://www.jobcher.com/)
{{< /admonition >}}
