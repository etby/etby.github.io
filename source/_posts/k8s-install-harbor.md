---
title: 使用 Helm 在 Kubernetes 集群中安装 Harbor
date: 2020-04-28 16:44:46
tags:
- Helm
- Kubernetes
- Harbor
- Docker

---

## Harbor

Harbor 是一个企业级 Registry 服务器，可以用来存放 Docker 镜像，还可以存放 Helm Chats。

### Proxy

Nginx 代理，用来转发请求。用 Ingress 之后此模块不会开启。

### Registry

负责存储 Docker 镜像，内核是 Docker 官方的 Registry。

由于 Registry 没有权限认证，所有 Harbor 对每次请求都会进行鉴权，通过之后再由 Registry 处理。

### Core

Harbor的核心功能，主要包含以下功能：

- 认证与授权
- 配置管理
- 元数据管理
- 内容管理与分发

### 其他组件

**Chart Museum**：Helm Chart 仓库。

**Job Services**：处理异步计算请求。

**Log collector**：日志收集。

**Notary**：信任中心，详情：https://github.com/theupdateframework/notary 。

**Web Portal**：图形化的用户页面，用户通过这个管理 Registry。

## 安装

添加仓库

```
helm repo add harbor https://helm.goharbor.io
helm repo add bitnami https://charts.bitnami.com/bitnami
```

更新

```
helm repo update
```

## 配置

获取配置条目

```
helm inspect values harbor/harbor
```

可以保存为文件，后续参考方便

```
helm inspect values harbor/harbor > origin.yaml
```

### 配置文件

```
# 密码
harborAdminPassword: "Harbor12345"
# 用于加密的一个 secret key，必须是一个16位的字符串
secretKey: "not-a-secure-key"

# 对外暴露配置
expose:
  type: ingress
  tls:
    enabled: true
    secretName: harbor-tls
    notarySecretName: harbor-tls
  ingress:
    hosts:
      core: registry.harbor.com
      notary: notary.harbor.com
    annotations:
      kubernetes.io/ingress.class: "nginx"

      ingress.kubernetes.io/ssl-redirect: "true"
      nginx.ingress.kubernetes.io/ssl-redirect: "true"

      ingress.kubernetes.io/proxy-body-size: "0"
      nginx.ingress.kubernetes.io/proxy-body-size: "0"

# 外部地址
externalURL: https://registry.harbor.com

# 持久存储
persistence:
  enabled: true
  # Setting it to "keep" to avoid removing PVCs during a helm delete
  # operation. Leaving it empty will delete PVCs after the chart deleted
  resourcePolicy: "keep"
  persistentVolumeClaim:
    registry:
      existingClaim: "oss-pvc"
      subPath: "register"
      accessMode: ReadWriteMany
    chartmuseum:
      existingClaim: "oss-pvc"
      subPath: "chartmuseum"
      accessMode: ReadWriteMany
    jobservice:
      existingClaim: "host-pvc"
      subPath: "jobservice"
      accessMode: ReadWriteMany
    database:
      existingClaim: "host-pvc"
      subPath: "database"
      accessMode: ReadWriteMany
    redis:
      existingClaim: "host-pvc"
      subPath: "redis"
      accessMode: ReadWriteMany
    trivy:
      existingClaim: "host-pvc"
      subPath: "trivy"
      accessMode: ReadWriteMany
```

主要配置密码和密钥，以及域名与证书。

存储我这边分了两部分：

- `oss-pvc` 是阿里云的 OSS，主要保存 Docker 镜像和 Helm Chars。
- `host-pvc` 则是服务器本机的目录，需要高性能的数据库都用这个，后续会增加备份。没有必要区分的数据也放在本机。

### 安装

配置正确之后只需要使用 Helm 安装即可。

```
helm install harbor harbor/harbor -f values.yaml . --namespace NAMESPACE
```

安装成功之后打开配置的 `externalURL`，即可访问。用户名 `admin`，密码是配置的 `harborAdminPassword`。

## 测试

使用命令行登陆

```
docker login https://register.harbor.com
```

下载一个官方的镜像

```
docker pull alpine:3.9
```

新建标签

```
docker tag alpine:3.9 register.harbor.com/library/alpine:3.9
```

上传

```
docker push register.harbor.com/library/alpine:3.9
```

## 参考

- https://github.com/goharbor/harbor-helm
- https://www.qikqiak.com/post/harbor-quick-install/
- https://github.com/goharbor/harbor/wiki/Architecture-Overview-of-Harbor