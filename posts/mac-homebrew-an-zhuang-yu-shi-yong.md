---
title: 'Mac Homebrew 安装与使用'
date: 2021-01-15 12:09:09
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
# **简单使用**

1. 安装软件：brew install 软件名，例：brew install wget
2. 搜索软件：brew search 软件名，例：brew search wget
3. 卸载软件：brew uninstall 软件名，例：brew uninstall wget
4. 更新所有软件：brew update
5. 更新具体软件：brew upgrade 软件名 ，例：brew upgrade git
6. 显示已安装软件：brew list
7. 查看软件信息：brew info／home 软件名 ，例：brew info git ／ brew home git
8. PS：brew home指令是用浏览器打开官方网页查看软件信息
9. 查看哪些已安装的程序需要更新： brew outdated
10. 显示包依赖：brew reps
11. 显示帮助：brew help

# 二、ARM X86安装与配置

### Arm版安装：

```bash
cd /opt # 切换到 /opt 目录
mkdir homebrew # 创建 homebrew 目录
sudo chown -R $(whoami) /opt/homebrew # 修改目录所属用户
curl -L [https://github.com/Homebrew/brew/tarball/master](https://github.com/Homebrew/brew/tarball/master) | tar xz --strip 1 -C homebrew
```

### 卸载命令

```bash
/usr/bin/ruby -e "(surl -fsSL https://...)"
```

### intel版的安装：

```bash
```shell

arch -x86_64 /bin/bash -c "$(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/master/install.sh](https://raw.githubusercontent.com/Homebrew/install/master/install.sh))"

```
```

### 环境变量设置

```bash
#vim ~/.zshrc 或者 vim ~/.bash_profile

# x86
$ export PATH="/usr/local/bin:$PATH"
$ alias abrew='arch -x86_64 /usr/local/bin/brew'

# arm
$ export PATH="/opt/homebrew/bin:$PATH"
$ alias brew='/opt/homebrew/bin/brew'
```

### 换源下载源

```bash
# Arm 版

cd "$(brew --repo)"
git remote set-url origin [git://mirrors](git://mirrors).[ustc.edu.cn/brew.git](http://ustc.edu.cn/brew.git)
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin [git://mirrors](git://mirrors).[ustc.edu.cn/homebrew-core.git](http://ustc.edu.cn/homebrew-core.git)

# intel版
cd "$(abrew --repo)"
git remote set-url origin [git://mirrors](git://mirrors).[ustc.edu.cn/brew.git](http://ustc.edu.cn/brew.git)
cd "$(abrew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin [git://mirrors](git://mirrors).[ustc.edu.cn/homebrew-core.git](http://ustc.edu.cn/homebrew-core.git)

bash用户：
echo 'export HOMEBREW_BOTTLE_DOMAIN=[https://mirrors](https://mirrors/).[ustc.edu.cn/homebrew-bottles](http://ustc.edu.cn/homebrew-bottles)' >> ~/.bash_profile
source ~/.bash_profile
zsh用户：
echo 'export HOMEBREW_BOTTLE_DOMAIN=[https://mirrors](https://mirrors/).[ustc.edu.cn/homebrew-bottles](http://ustc.edu.cn/homebrew-bottles)' >> ~/.zshrc
source ~/.zshrc
```