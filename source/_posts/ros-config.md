---
title: RouterOS 配置
date: 2020-01-01 11:12:14
tags:
- RouterOS
- Router
---

## 基本操作

ROS 命令行一个命令其实逻辑上会进入到对应的目录，只有最后一个执行文件才会真正执行，所以可以用这种操作简化命令输入。

```shell
# 输入 system 其实进入了 system 目录， 之后进入 routerboard 目录，
[admin@MikroTik] > system
[admin@MikroTik] /system> routerboard
[admin@MikroTik] /system routerboard>

# 再输入最后的可执行命令，就执行了 print 命令
[admin@MikroTik] /system routerboard> print
       routerboard: yes
             model: RB4011iGS+
     serial-number: B8FF0B97D030
     firmware-type: al2
  factory-firmware: 6.44.5
  current-firmware: 6.44.6
  upgrade-firmware: 6.44.6
  
# 如果想回到根目录，使用 /
[admin@MikroTik] /system routerboard> /
[admin@MikroTik] >

# 在命令行中随时输入 ？ 会弹出命令和帮助
[admin@MikroTik] /ip address>
IP addresses are given to router to access it remotely and to specify it as a gateway for other hosts/routers.

.. -- go up to ip
add -- Create a new item
comment -- Set comment for items
disable -- Disable items
edit --
enable -- Enable items
export -- Print or save an export script that can be used to restore configuration
find -- Find items by value
get -- Gets value of item's property
print -- Print values of item properties
remove -- Remove item
set -- Change item properties
```

## 新机联网

### 修改默认配置

```
# 修改默认的空密码为自定义的密码
/password

# 关闭一些不用的服务 （ 我这边只保留了 ssh 和 winbox ）
/ip service disable telnet,ftp,www,api,api-ssl
```

### 配置公网

由于目前都是光猫直接拨号，一般情况只需要配置IP即可。

```
# 修改与光猫连接端口名称 
/interface ethernet set ether1 name=WAN-CT-HOME

# 配置光猫内网IP 针对与光猫链接的端口
/ip address add address=192.168.1.2/24 interface=WAN-CT-HOME

# 可以尝试 ping 光猫测试是否成功
/ping 192.168.1.1
```

### 内网配置

**网络配置**

```
# 新建网桥
/interface bridge add name=bridge

# 将接口加入网桥
/interface bridge port add bridge=bridge interface=ether5 hw=yes
/interface bridge prot add bridge=bridge interface=ether6 hw=yes
...

# 在网桥上绑定 IP
/ip address add address=10.10.10.1/24 interface=bridge

# 添加默认网关
/ip route add gateway=192.168.1.1

# 出口 SNAT 转换
/ip firewall nat add chain=srcnat action=masquerade
```

**DNS 和 DHCP**

```
# 配置 DNS 并且允许其他设备访问
/ip dns set servers=1.1.1.1,114.114.114.114 allow-remote-requests=yes

# 新建 IP 池
/ip pool add ranges=10.10.10.100-10.10.10.200 name=NET-POOL

# 建立 DHCP 服务器
/ip dhcp-server add name=NET-DHCP interface=bridge address-pool=NET-POOL

# 配置 DHCP 网络
/ip dhcp-server network add address=10.10.10.0/24 dns-server=10.10.10.1 gateway=10.10.10.1
```

## VLAN

### 内网 VLAN 配置

**网桥**

```
# 新建支持 VLAN 的网桥
/interface bridge add name=bridge vlan-filtering=no

# 将端口加入网桥 并配置 VLAN
/interface bridge port add bridge=bridge interface=ether5 pvid=10 hw=yes
/interface bridge prot add bridge=bridge interface=ether6 pvid=10 hw=yes
...

# 在网桥上新建路由器使用的 VLAN
/interface vlan add interface=bridge name=VLAN-NET vlan-id=10

# 在 VLAN 上绑定 IP
/ip address add address=10.10.10.1/24 interface=VLAN-NET
```

**VLAN**

```
# 开启入口流量过滤 提高安全性
/interface bridge port 
set [f] ingress-filtering=yes

# 设置网桥端口过滤 VLAN 标签模式
/interface bridge port
# 允许所有类型帧通过
set [find where interface=<接口>] frame-types=admit-all
# 只允许打了合适 VLAN 标签的帧通过
set [find where interface=<接口>] frame-types=admit-only-vlan-tagged
# 允许未打标签和优先标签通过
set [find where interface=<接口>] frame-types=admit-only-untagged-and-priority-tagged

# 配置网桥 VLAN
# 针对 VLAN 10，sfp1 打标签，ether1,ether2 去掉标签
add bridge=bridge vlan-ids=10 tagged=sfp1 untagged=ether1,ether2 
# 如果网桥自身也要转发，需要配置网桥自身
add bridge=bridge tagged=bridge,ether1 vlan-ids=1

```

## Firewall

### NAT

**查看现有的NAT**

```
[admin@MikroTik] /ip firewall nat> print
Flags: X - disabled, I - invalid, D - dynamic
 0    ;;; defconf: masquerade
      chain=srcnat action=masquerade out-interface-list=WAN ipsec-policy=out,none
```

目前是默认配置，对于所有 WAN 接口列表中的接口，都使用改变源IP的方式做NAT。

#### **配置端口转发**

```
# 将外网8000端口映射到内网服务器上的80端口
/ip firewall nat add chain=dstnat protocol=tcp dst-port=8000 in-interface=ether5 action=dst-nat to-addresses=10.10.10.80 to-ports=80
```

### Filter

```
# 允许已经建立的链接通过
/ip firewall filter add chain=input connection-state=established,related in-interface=WAN-CT-DIRECT action=drop
# 过滤掉所有的外部链接进入
/ip firewall filter add chain=input in-interface=WAN-CT-DIRECT action=drop
```

### WAN 负载均衡

**标记公网网卡进入的链接**

```
/ip firewall mangle
add action=mark-connection chain=prerouting in-interface=WAN-CT-DIRECT connection-mark=no-mark new-connection-mark=WAN_DIRECT 
add action=mark-connection chain=prerouting in-interface=WAN-CT-HOME connection-mark=no-mark new-connection-mark=WAN_HOME
```

**创建 PCC 规则**

https://wiki.mikrotik.com/wiki/Manual:PCC

```
# 未标记的内网流量根据算法选择路由出去
/ip firewall mangle 
add chain=prerouting in-interface=LAN connection-mark=no-mark dst-address-type=!local action=mark-connection new-connection-mark=WAN_DIRECT per-connection-classifier=both-address:2/0 
add chain=prerouting in-interface=LAN connection-mark=no-mark dst-address-type=!local action=mark-connection new-connection-mark=WAN_HOME per-connection-classifier=both-address:2/1
```

**建立路由标记**

```
/ip firewall mangle 
add chain=prerouting connection-mark=WAN_DIRECT in-interface=LAN action=mark-routing \ 
    new-routing-mark=TO_WAN_DIRECT
add chain=prerouting connection-mark=WAN_HOME in-interface=LAN action=mark-routing \ 
    new-routing-mark=TO_WAN_HOME
add chain=output connection-mark=WAN_DIRECT action=mark-routing new-routing-mark=TO_WAN_DIRECT     
add chain=output connection-mark=WAN_HOME action=mark-routing new-routing-mark=TO_WAN_HOME
```

**创建路由**

策略路由 distance 默认为 0，之后创建备用的 distance 为 1 的路由备用

```
/ip route
add dst-address=0.0.0.0/0 gateway=192.168.1.1 routing-mark=TO_WAN_DIRECT check-gateway=ping
add dst-address=0.0.0.0/0 gateway=192.168.2.1 routing-mark=TO_WAN_HOME check-gateway=ping
add dst-address=0.0.0.0/0 gateway=192.168.1.1 distance=1 check-gateway=ping
add dst-address=0.0.0.0/0 gateway=192.168.2.1 distance=2 check-gateway=ping
```

**SNAT**

```
/ ip firewall nat 
add chain=srcnat out-interface=WAN-CT-DIRECT action=masquerade
add chain=srcnat out-interface=WAN-CT-HOME action=masquerade
```

**策略路由**

防止策略路由导致的路由环路，处理 WAN 的局部网络

```
/ ip firewall mangle
add chain=prerouting dst-address=192.168.1.0/24  action=accept in-interface=LAN
add chain=prerouting dst-address=192.168.2.0/24  action=accept in-interface=LAN
```

