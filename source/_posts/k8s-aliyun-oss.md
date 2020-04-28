---
title: Kubernetes 使用 Aliyun OSS 作为持久化存储
date: 2020-04-28 08:26:52
tags:
- Kubernetes
- Aliyun OSS
- CSI
---

## 插件选择

`Kubernetes` 与外部存储对接时要使用插件，自定义插件有很多协议。阿里云支持的插件为 `Flexvolume` 和 `CSI`，需要选择用哪一个插件搭建系统。由于 CSI 比较新，而且社区推荐使用，我这边就选择了 CSI。

### Flexvolume

Flexvolume插件是Kubernetes社区较早实现的存储卷扩展机制。ACK从上线起，即支持Flexvolume类型数据卷服务。Flexvolume插件包括以下三部分。

- Flexvolume：负责数据卷的挂载、卸载功能。ACK默认提供云盘、NAS、OSS三种存储卷的挂载能力。
- Disk-Controller：负责云盘卷的自动创建能力。
- Nas-Controller：负责NAS卷的自动创建能力。

### CSI

CSI插件是当前Kubernetes社区推荐的插件实现方案。ACK集群提供的CSI存储插件兼容社区的CSI特性。CSI插件包括以下两部分：

- CSI-Plugin：实现数据卷的挂载、卸载功能。ACK默认提供云盘、NAS、OSS三种存储卷的挂载能力。
- CSI-Provisioner：实现数据卷的自动创建能力，目前支持云盘、NAS两种存储卷创建能力。

## 安装插件

### 命名空间

```
kubectl create namespace cloud-retirwer-k8s-plugin
```

### RBAC

下载 RBAC 配置

```
wget https://raw.githubusercontent.com/kubernetes-sigs/alibaba-cloud-csi-driver/master/deploy/rbac.yaml
```

修改 `namespace` ，后续的所有配置也都要修改命名空间。

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  # replace with the same namespace name with plugin
  namespace: cloud-retirwer-k8s-plugin

---
# 省略权限配置
---

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: alicloud-csi-plugin
subjects:
  - kind: ServiceAccount
    name: admin
    namespace: cloud-retirwer-k8s-plugin
roleRef:
  kind: ClusterRole
  name: alicloud-csi-plugin
  apiGroup: rbac.authorization.k8s.io
```

加载配置

```
kubectl apply -f rbac.yaml
```

### OSS-Plugin

下载配置

```
wget https://raw.githubusercontent.com/kubernetes-sigs/alibaba-cloud-csi-driver/master/deploy/oss/oss-plugin.yaml
```

修改 `namespace`

```
kind: DaemonSet
apiVersion: apps/v1beta2
metadata:
  name: csi-ossplugin
  namespace: cloud-retirwer-k8s-plugin
```

加载配置

```
kubectl apply -f oss-plugin.yaml
```

常见问题如下，k8s 高版本对于 beta 版 API 有所更改。

```
error: unable to recognize "oss-plugin.yaml": no matches for kind "DaemonSet" in version "apps/v1beta2"
```

如果碰见以上问题 ，则将版本修改为 V1

```
kind: DaemonSet
apiVersion: apps/v1
```

### 测试

>  需要提前创建好 OSS Bucket，配置好权限，获得访问密钥。

下载测试配置

```
wget https://raw.githubusercontent.com/kubernetes-sigs/alibaba-cloud-csi-driver/master/examples/oss/deploy.yaml
wget https://raw.githubusercontent.com/kubernetes-sigs/alibaba-cloud-csi-driver/master/examples/oss/pv.yaml
wget https://raw.githubusercontent.com/kubernetes-sigs/alibaba-cloud-csi-driver/master/examples/oss/pvc.yaml
```

修改配置

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: oss-csi-pv
  labels:
    alicloud-pvname: oss-csi-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: ossplugin.csi.alibabacloud.com
    # set volumeHandle same value pv name
    volumeHandle: oss-csi-pv
    volumeAttributes:
      bucket: "<*>"
      url: "oss-cn-hangzhou.aliyuncs.com"
      otherOpts: "-o max_stat_cache_size=0 -o allow_other"
      akId: "<*>"
      akSecret: "<*>"
      path: "/test"
```

加载配置

```
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f deploy.yaml
```

检查运行状态，如果三个都正常就没问题了

```
kubectl get pv | grep oss
kubectl get pvc | grep oss
kubectl get pod | grep oss
```

## 使用

### PV

使用此 CSI 主要是需要在定义 PV 的配置中做特殊配置：

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-name
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  # 回收模式 一定要选保留
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: ossplugin.csi.alibabacloud.com
    # set volumeHandle same value pv name
    volumeHandle: oss-csi-pv
    volumeAttributes:
      bucket: "<*>"
      url: "oss-cn-hangzhou.aliyuncs.com"
      otherOpts: "-o max_stat_cache_size=0 -o allow_other"
      akId: "<*>"
      akSecret: "<*>"
      path: "/test"
```

#### 存储大小

```
# 定义存储大小，Gi 代表以 1024 的算法进行计算大小
capacity:
	storage: 5Gi
```

#### 访问模式

```
# 访问模式 OSS 支持 ReadWriteMany
accessModes:
	- ReadWriteOnce
```

#### 回收模式

```
persistentVolumeReclaimPolicy: Retain
```

回收模式一般要选保留，不排除特殊情况

#### CSI

```
  csi:
    driver: ossplugin.csi.alibabacloud.com
    volumeHandle: oss-csi-pv
    volumeAttributes:
      bucket: "<*>"
      url: "oss-cn-hangzhou.aliyuncs.com"
      otherOpts: "-o max_stat_cache_size=0 -o allow_other"
      akId: "<*>"
      akSecret: "<*>"
      path: "/test"
```

`driver` 是所要挂载的驱动，需要和之前安装的相同，如果你用其他厂家或者是不同的产品，根据情况调整。`volumeHandle` 唯一标识从 CSI 卷插件的 `CreateVolume` 调用返回的卷名。随后在卷驱动程序的所有后续调用中使用卷句柄来引用该卷，一般和 PV 名称相同。`volumeAttributes` 则是外部服务提供商自己定义的值，OSS 目前是这些配置，其他厂家有不同的配置。

### PVC

不需要做特殊配置。选择器需要匹配自己想要使用的 PV。

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: oss-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      alicloud-pvname: oss-csi-pv
```

## 参考

- https://github.com/kubernetes-sigs/alibaba-cloud-csi-driver/blob/master/README-zh_CN.md

- https://help.aliyun.com/document_detail/157026.html