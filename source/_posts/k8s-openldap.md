---
title: 在 Kubernetes 上部署 OpenLDAP 服务
date: 2020-05-05 09:05:28
tags:
- Kubernetes
- OpenLDAP
---

## 安装

安装这块我就不细讲了，不论是用 `Docker` 还是 `Helm` ，官方的文档永远是最好的参考。我这边是用 `Helm` 在 `Kubernetes`  上安装的。安装其实很简单，

### 环境变量

官方的 `Docker` 镜像中，环境变量有一些是只在初始化过程中有用的。比如：`LDAP_DOMAIN` ， `LDAP_ADMIN_PASSWORD` 后期一般在 `OpenLDAP` 内部自己改即可。

### Helm 配置

创建自定义的配置文件

```
vi values.yaml
```

添加配置，注意修改域名和密码为自己的。我这边由于是在集群中使用，因此没有开 `TLS`。开启只读用户并且配置，用来让其他平台的服务读取 `LDAP` 配置，使用其登陆。

```
nameOverride: ldap
fullnameOverride: ldap

# Use the env variables from https://github.com/osixia/docker-openldap#beginner-guide
env:
  LDAP_ORGANISATION: "Retirwer Technology Co., Ltd"
  LDAP_DOMAIN: "retirwer.com"
  LDAP_BACKEND: "hdb"
  LDAP_TLS: "false"
  LDAP_TLS_ENFORCE: "false"
  LDAP_REMOVE_CONFIG_AFTER_SETUP: "true"
  LDAP_READONLY_USER: "true"
  LDAP_READONLY_USER_USERNAME: "readonly"
  LDAP_READONLY_USER_PASSWORD: "<password>"

# Default Passwords to use, stored as a secret. If unset, passwords are auto-generated.
# You can override these at install time with
# helm install openldap --set openldap.adminPassword=<passwd>,openldap.configPassword=<passwd>
adminPassword: <passwd>
configPassword: <passwd>
```

### 部署

```
helm install ldap stable/openldap -f values.yaml
```

一般情况下，访问地址是

```
ldap.<namespace>.svc.cluster.local
```

## 配置

OpenLDAP 虽然应用非常广泛，但网上资料却少的可怜。如果你有 **Kindle Unlimited**，上面有一本**郭大勇**写的**《Linux/UNIX OpenLDAP实战指南》**可以免费做参考。如果没有则不建议去买，普通用户只能用到一点点里面的知识，如果你是真正的运维可以考虑买来看看。

### 客户端

客户端我推荐 **Apache Directory Studio**，毕竟 **LDAP** 的配置是在是太难了！普通用户没必要去学习配置脚本甚至命令，开源跨平台图形化界面的配置工具足够好用。

**安装 Apache Directory Studio**

```
brew cask install apache-directory-studio
```

### 管理内容

管理用户账户、群组和设备，用此方式登陆。

使用以下 `DN` 和 `adminPassword` 登陆，域名需要配置成自己的域名，如果有三层，就配置三层 `dc`。

```
cn=admin,dc=retirwer,dc=com
```

### 管理配置

管理基础配置、权限控制等，用此方式登陆。

使用以下 `DN` 和 `configPassword` 登陆

```
cn=admin,dc=config
```

### 接入配置

第三方平台接入登陆，比如 `Gitlab`、`Harbor`、`Jenkins `等等。需要读取用户和群组信息，使用只读账户。

使用以下 `DN` 和 `LDAP_READONLY_USER_PASSWORD` 登陆

```
cn=<LDAP_READONLY_USER_USERNAME>,dc=retirwer,dc=com
```

### 权限配置

参考官方文档： https://www.openldap.org/doc/admin24/access-control.html 

## 参考

- https://github.com/osixia/docker-openldap

- https://github.com/helm/charts/tree/master/stable/openldap

- 《Linux/UNIX OpenLDAP实战指南》郭大勇
- https://directory.apache.org/studio/
- https://www.openldap.org/doc/admin24/access-control.html