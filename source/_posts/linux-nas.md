---
title: 在 Linux 服务器上搭建 NAS 服务
date: 2019-08-10 14:25:42
tags:
- linux
- nas
- nfs
---

## NFS

### 服务端

ubuntu 直接使用包管理器安装，其他平台类似。

```
apt install nfs-kernel-server
```

创建专用目录。

```
mkdir -p /data/nas-root
chown nobody:nogroup /data/nas-root
chmod 777 /data/nas-root
```

导出文件配置

```
vim /etc/exports

/data/nas-root 10.10.10.0/24(rw,sync,no_subtree_check)
```

使配置生效

```
exportfs -a
systemctl restart nfs-kernel-server
```

### 客户端

查看服务器共享文件状态

```
showmount -e 10.10.10.100

Exports list on 10.10.10.100:
/data/nas-root                      10.10.10.0/24
```

#### 挂载

我这边是在 Mac 下挂载，Linux 平台只需要改变路径就好。

```
sudo mount -o resvport -t nfs 10.10.10.100:/data/nas-root /Volumes/NAS
```

## SAMBA

### 服务端

Ubuntu 使用包管理器安装

```
apt install samba
```

增加 SAMBA 账户

```
smbpasswd -a etby
```

编辑配置文件

```
vi /etc/samba/smb.conf

[NAS]
comment = NAS Directory
browseable = yes
path = /data/nas-root
create mask = 0777
directory mask = 0777
valid users = etby
force user = nobody
force group = nogroup
public = yes 
available = yes 
writable = yes
```

