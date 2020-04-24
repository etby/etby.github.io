---
title: 将 Kubernetes 从 1.17 升级到 1.18
date: 2020-04-24 21:58:56
tags:
---

## 经验总结

我的集群是自建的独立 ETCD 集群，然后其他组件全部用 kubeadm 来进行安装的，升级起来比较方便。

一般来说，只有第一台 Master 需要的操作比较多，后续的 Master 和 Node 操作基本相同。如果经常升级的话，倒是可以写一个脚本去做，我因为第一次做 K8S 的升级。不太熟，所以全程手动升级的。

过程中使用 Tmux 进行远程链接安全性的保证。如果因为某些操作中途中断导致升级失败，那麻烦就大了。

## 升级要求

- 必须在 1.17 以上版本
- 必须使用外部 ETCD
- 切记备份数据

## 确定目标版本

```
apt update
apt-cache madison kubeadm
```

得到最新版本 `1.18.2-00`

## 升级主节点集群


### 开始升级第一个主节点

升级 kubeadm 

```
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.18.2-00 && \
apt-mark hold kubeadm
```

确认版本

```
kubeadm version
```

清空节点上的容器

```
kubectl drain <cp-node-name> --ignore-daemonsets
```

在需要升级的节点上运行

```
kubeadm upgrade plan
```

选择要升级到的版本，然后运行相应的命令

```
kubeadm upgrade apply v1.18.2
```

升级完成之后需要升级 kubelet 和 kubectl ：

```
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.18.2-00 kubectl=1.18.2-00 && \
apt-mark hold kubelet kubectl
```

重启 kubelet

```
systemctl restart kubelet
```

取消节点限制策略

```
kubectl uncordon <cp-node-name>
```

### 升级其他主节点

先升级 kubeadm

```
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.18.2-00 && \
apt-mark hold kubeadm
```

然后清空节点：

```
kubectl drain <cp-node-name> --ignore-daemonsets
```

执行升级命令

```
kubeadm upgrade node
```

升级完成之后需要升级 kubelet 和 kubectl ：

```
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.18.2-00 kubectl=1.18.2-00 && \
apt-mark hold kubelet kubectl
```

重启 kubelet

```
systemctl restart kubelet
```

取消节点限制策略

```
kubectl uncordon <cp-node-name>
```

## 升级工作节点

升级工作节点的步骤与升级其他主节点步骤相同。

## 其他

网络插件按需升级，我这边暂时没有升级的需求，就没有升级了。

## 参考

- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

- https://mritd.me/2020/01/21/how-to-upgrade-kubeadm-cluster/