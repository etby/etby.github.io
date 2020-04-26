---
title: 在 K8S 集群上发布应用
date: 2020-04-26 22:23:42
tags:
- Kubernetes
- Helm
---

## 应用介绍

目标是将公司网站发布到集群中。前期准备：域名备案，编写代码与打包镜像。由于目前还是简易的纯前端展示页面，以及基础设施还不太完善，我就直接将镜像放到 DockerHub 上了。

### 源码

https://github.com/Retirwer/www.retirwer.com

### 镜像

https://hub.docker.com/repository/docker/retirwer/www.retirwer.com

## 集群环境

### CertManager

用来自动申请 HTTPS 证书。

### Helm

用来进行 Kubernetes 应用的管理。

## 应用配置

### 创建 Helm Chars

```
helm create www.retirwer.com
```

由于基本模板对于简单的展示站够用，目前只需要修改 `Chart.yaml` 和 `values.yaml` 文件即可。

### Chart.yaml

```
apiVersion: v2
name: www.retirwer.com
description: Retirwer Technology Co., Ltd Official Web Site.
type: application
version: 0.1.0
appVersion: 0.0.1
```

### values.yaml

修改镜像，版本则和 `appVersion` 相同，后续更改版本只需要修改 `Chart.yaml` 中的 `appVersion`。

```
image:
  repository: retirwer/www.retirwer.com
  pullPolicy: IfNotPresen
```

配置 `Ingress`

```
ingress:
  enabled: true
  annotations:
    # 使用 Nginx-Ingress 的配置
    kubernetes.io/ingress.class: nginx
    # 根据自己的 cert-manager 的配置修改
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: www.retirwer.com
    	# 只配置根目录
      paths: [/]
  tls:
   - secretName: www.retirwer.com-tls
     hosts:
       - www.retirwer.com
```

### 其他

- 对于其他用不到的配置模版可以删掉，也可以不启用。

## 发布

一般必须指定命名空间，如果不指定则使用当前命名空间。

### 确认发布配置

```
helm install <release-name> . --debug --dry-run
```

### 首次发布

```
helm install <release-name> . --namespace <namespace>
```

### 更新发布

```
helm upgrade <release-name> . --namespace <namespace>
```

### 卸载发布

```
helm uninstall <release-name> --namespace <namespace>
```





























