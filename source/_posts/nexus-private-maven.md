---
title: 使用Nexus搭建私有Maven仓库
date: 2017-10-22 21:39:31
categories: DevOps
tags: 
- Maven
- Linux
- Docker
---

## 环境

服务器在阿里云香港, 避免不必要的网络问题. 使用Ubuntu系统, 因为包管理比较方便. 系统版本是`Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-49-generic x86_64)`

## 安装

### 安装Docker

下面是一段在Ubuntu中安装Docker的脚步, 虽然我只在`17.04`版本上测试过, 但是其他版本应该也可以用. 

```shell
#!/bin/bash

sudo apt-get update

sudo apt-get install \
         apt-transport-https \
         ca-certificates \
         curl \
         software-properties-common -y

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
         "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
         $(lsb_release -cs) \
         stable"

sudo apt-get update

sudo apt-get install docker-ce -y
```

如果不能使用, 请按照官网步骤进行. 如果是其他系统, 请参考官网其他系统的文档
https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/

### 安装&启动 Nexus3

如果Docker已经安装成功, 则只需要一行命令即可

`docker run -d -p 127.0.0.1:8081:8081 --name nexus sonatype/nexus3`

>  我这边需要使用Nginx反向代理, 所以直接本地访问, 如果想通过IP访问, 去掉IP配置即可

安装完成之后`docker ps`查看一下, 应该会是下面这样

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
64c907fbac26        sonatype/nexus3     "bin/nexus run"          25 hours ago        Up 25 hours         127.0.0.1:8081->8081/tcp   nexus
```

 这样就已经启动了, 之后再用Nginx配置域名之后反向代理就可以通过域名访问了.

## 代理JCenter等仓库

![](http://oy6e75a1i.bkt.clouddn.com/nexus-jcenter.png)

以上这个是我的jcenter的配置, 其他仓库配置都是一样, 只需要替换名称和远程连接就行

![](http://oy6e75a1i.bkt.clouddn.com/neuxs-repositorys)

这是目前的配置, 代理了jcenter和mavenCentral, 还有google和jitpack. 
然后创建了一个组, `etby_works`. 这样我在studio客户端只需要配置一个就好, 在需要的时候可以直接在服务器上新增远程仓库. 新项目可以继续创建新的组, 也可以使用一个大组来负责所有的远程仓库.

## 权限配置

### 关闭匿名访问

Security -> Anonymous -> 取消勾选 Allow anonymous users to access the server 之后确认

### 添加只读权限用户

创建用户时, 配置`Save` 按钮上方的 `Granted`为`nx-anonymous`, 此权限默认只读.

### 客户端配置

```groovy
allprojects {
    repositories {
        maven {
            url 'http://maven.etby.studio/repository/etby_works/'
            credentials {
                username 'username'
                password 'password'
            }
        }

    }
}
```

在Android Studio中配置成这样基本就没问题了, `url`和账号密码都改成自己的, 上面的也不是我的账号密码 =-=

## 其他

目前暂时没有仓库上传需求, 所以没有写上传的一些东西, 如果以后有机会再补全吧~

如果有不详细的地方, 可以直接Google, 也可以联系我.

> 纪念第一篇原创文章