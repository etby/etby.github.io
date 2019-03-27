---
title: Linux 初始化配置
date: 2019-03-27 23:13:12
categories: DevOps
tags: 
  - linux
---

## 前言

每次`Linux`初始化配置都比较麻烦，所以在此记录一下，以后方便配置。我主要是用 `Ubuntu` ，所以以下是基于它的，其他系统类似。

## 配置

### SUDO 免密码

```
sudo vi /etc/sudoers

etby	ALL=(ALL) NOPASSWD:ALL
```

### 卸载 SNAP

针对主要用来做母鸡的服务器

```
sudo apt autoremove --purge snapd
```



## 安装软件

### OH MY ZSH

`OH MY ZSH` 以及它的工具链用的很熟练了，感觉非常舒服。

#### 安装

```
# 安装本体
sudo apt-get install zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# 安装插件
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

#### 配置

```
plugins=(
# Common
  git
  zsh-syntax-highlighting
  zsh-autosuggestions
  
# Develop
  git-flow-avh # 需要安装 git-flow-avh # sudo apt-get install git-flow
  
# Server
  docker
  docker-compose
  kubectl
)
```

### 显卡驱动

```
# 添加驱动仓库
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update

# 安装 Ubuntu 驱动工具
sudo apt-get install ubuntu-drivers-common
# 查看设备以及推荐
sudo ubuntu-drivers devices
# 按推荐的显卡驱动安装
sudo apt-get install nvidia-driver-418

# 重启
sudo reboot
# 查看安装情况
nvidia-smi
```

