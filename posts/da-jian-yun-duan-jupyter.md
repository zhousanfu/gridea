---
title: '搭建云端 Jupyter'
date: 2020-02-09 12:28:42
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
Jupyter Notebook 是数据科学 / 机器学习社区内一款非常流行的工具

最近入手了一台新的服务器，于是我想到将 Jupyter 搭建到服务器上，在任何只要有浏览器的地方都能进行 Python 编程

我的服务器是 ubuntu 的，不同系统根据自己系统的命令进行操作

### **创建用户、切换用户展开目录**

首先在 root 用户下打开防火墙 8888 端口，这是提供 Jupyter 服务的端口

```
sudo ufw allow 8888

```

然后创建一个用户名为 `mathor` 的用户

```
sudo adduser mathor

```

输入密码并确认密码

然后一路 Enter，默认就行，最后输入 `y` 确认一下

然后切换到新用户，并进入当前用户的主目录

```
su mathor

cd ~

```

### **下载并安装 Anaconda展开目录**

Anaconda 的 Linux 下载网址是 **[https://www.anaconda.com/download/#linux](https://www.anaconda.com/download/#linux)**

截止到今天的最新版本是 2018.12，所以通过命令下载

```
wget https://repo.continuum.io/archive/Anaconda3-2018.12-Linux-x86_64.sh

```

下载完成后运行

```
bash Anaconda3-2018.12-Linux-x86_64.sh

```

之后会有一个协议，输入 `yes`，然后会有安装路径选择，按下 Enter 就是默认路径，之后会问是否要加入到环境变量，输入 `yes`，之后问要不要安装 vs code，输入 `no`。最后安装完成，输入

```
jupyter

```

按两下 tab 键提示很多东西，就证明通过 Anaconda 安装 Jupyter 成功了，如果没有反应，同时发现输入 `conda` 没有命令，那么执行下面两步就可以了

```
echo 'export PATH="~/anaconda3/bin:$PATH"'>>~/.bashrc

source ~/.bashrc

```

### **配置 Jupyter展开目录**

运行命令

```
jupyter-notebook --generate-config

```

这时看到一个反馈

```
Writingdefault config to:/home/mathor/.jupyter/jupyter_notebook_config.py

```

这就是配置的目录。然后运行命令

```
jupyter-notebook password

```

输入密码并确认，这就是以后登陆的密码

输入命令

```
vi .jupyter/jupyter_notebook_config.json

```

可以看到有一个字符串 `sha1:xxxxxxx`，复制下来，一会要用到。然后运行命令

```
mkdir jupyterdata

```

创造一个文件夹存放 jupyter 的代码。最后配置端口与代码存放路径

```
vi .jupyter/jupyter_notebook_config.py

```

随便在空白处写上下面的关键配置即可：

```
# 设置默认目录

c.NotebookApp.notebook_dir = u'/home/mathor/jupyterdata'

# 允许通过任意绑定服务器的ip访问

c.NotebookApp.ip = '*'

# 用于访问的端口

c.NotebookApp.port = 8888

# 不自动打开浏览器

c.NotebookApp.open_browser = False

# 设置登录密码

c.NotebookApp.password = u'sha1:xxxxxxxxxxxxxxxx'

```

保存并退出（按下 ESC，输入`:wq` 回车）

然后运行

```
jupyter-notebook

```

如果出现下面的报错

```
PermissionError: [Errno 13] Permission denied: '/run/user/xxxx/jupyter'

```

则输入

```
export XDG_RUNTIME_DIR="/home/mathor/anaconda3"

source .bashrc

jupyter-notebook

```

然后就 OK 了

### **本地测试展开目录**

随便在一个客户机浏览器里输入 `http://ip:8888` 就可以进入 Jupyter 的登陆界面了