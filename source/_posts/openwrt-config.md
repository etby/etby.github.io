---
title: OpenWrt 路由器配置
date: 2020-01-05 14:44:09
tags:
- OpenWrt
---

## INIT

**Change Password**

```
password
```

## WAN

```
# Config Static IP
uci set network.wan.proto=static
uci set network.wan.ipaddr=10.10.10.5
uci set network.wan.netmask=255.255.255.0
uci set network.wan.gateway=10.10.10.1

# Review Config
uci export network

# Commit UCI Config, need restart network to working.
uci commit
```

## LAN

> Use edit config files way.

```
vi /etc/config/network

config interface 'lan'
        option type 'bridge'
        option ifname 'eth0'
        option proto 'static'
        option ipaddr '192.168.1.1'
        option netmask '255.255.255.0'
        option dns '10.10.10.1'
        option ip6assign '60'
```

## WLAN

```
# Enable Wireless Device
uci set wireless.@wifi-device[0].disabled=0

# Config Wireless
uci set wireless.@wifi-iface[0].mode=ap
uci set wireless.@wifi-iface[0].ssid=TSFuShui
uci set wireless.@wifi-iface[0].encryption=psk2
uci set wireless.@wifi-iface[0].key=88888888
uci set wireless.@wifi-iface[0].network=lan

# Review Config
uci export wireless

# Commit UCI Config, need restart network to working.
uci commit
```

## TEST

```
# Restart network to apply config
/etc/init.d/network restart

# If ping is ok, Good Net.
ping 1.1.1.1
```

## PACKAGE

```
# update package list
opkg update

# show upgrable pkg
opkg list-upgradable

# upgrade one pkg
opkg upgrade <pkg>

# upgrade all pkg
opkg list-upgradable | cut -f 1 -d ' ' | xargs opkg upgrade
```

