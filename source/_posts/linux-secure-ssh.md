---
title: 配置安全的 SSHD
date: 2019-11-21 16:12:37
tags:
- Linux
- Security
---

## 修改配置文件

编辑 `/etc/ssh/sshd_config`


```
# 修改 SSH 端口为非常用端口
Port 22222
# 限制登陆过程的时间
LoginGraceTime 5
# 限制登陆过程重试次数
MaxAuthTries 1

# 禁止 Root 用户登陆
PermitRootLogin no 
# 禁止空密码
PermitEmptyPasswords no
# 禁止密码登陆
PasswordAuthentication no 

# 只允许特定用户登陆
AllowUsers etbyapt 
```

重启服务 `systemctl restart ssh`

## 隐藏端口

虽然我们能够更改端口号，和一些配置。但是，仍然有被扫描器扫描到的风险。如何能够让某个端口只在特定的情况下为我们自己开启，然后进行连接呢？我们可以使用 `knockd` 这个工具来完成这个工作。

knockd 可以检测某些特定端口的特定访问，然后根据配置的规则触发一些事件。事件触发之后就可以通过命令开启某些特定的端口，然后我们就可以进行连接了。

### 安装配置 knockd

```
sudo apt install knockd
```

编辑 `/etc/knockd.conf`

```
[options]
    LogFile       = /var/log/knockd.log
    Interface     = enp5s0  # 这里使用自己的网卡配置

[opencloseSSH]
    sequence      = 7000,8000,9000
    seq_timeout   = 5
    tcpflags      = syn
    start_command = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    cmd_timeout   = 30
    stop_command  = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
```

以上配置详解

```
sequence 配置端口号序列 可以使用 22:udp 代表udp的端口号
seq_timeout 连续访问上方序列的时间限制
tcpflags 包的 TCP 标志
start_command 完成序列之后执行的命令
cmd_timeout 结束命令的时间 
end_command 结束之后执行的命令 
由于防火墙对已经链接成功的包不进行过滤，所以链接之后再关闭包仍然能够通过
```

启动 knockd

```
systemctl enable knockd.service
systemctl start knockd.service
```

### 测试 knockd

测试需要安装 knock 

```
apt install knockd # 客户端和服务器在一个包里面
brew install knock
```

测试上面配置文件中的端口序列

```
$ knock  -v {IP} 7000 8000 9000

hitting tcp 10.10.10.100:7000
hitting tcp 10.10.10.100:8000
hitting tcp 10.10.10.100:9000
```

查看日志文件 `/var/log/knockd.log`

```
[2019-11-21 20:32] 10.10.10.133: opencloseSSH: OPEN SESAME
[2019-11-21 20:32] opencloseSSH: running command: /sbin/iptables -A INPUT -s 10.10.10.133 -p tcp --dport 22 -j ACCEPT

[2019-11-21 20:32] 10.10.10.133: opencloseSSH: command timeout
[2019-11-21 20:32] opencloseSSH: running command: /sbin/iptables -D INPUT -s 10.10.10.133 -p tcp --dport 22 -j ACCEPT
```

以上说明配置成功了，之后每次 SSH 之前需要先使用 knock 命令敲门之后再进行连接。

#### 注意

- knockd 所配置的敲门序列需要搭配 iptables 的包过滤才能真正发挥效果。

- knockd 的敲门模式不只是可以用来打开端口，也可以执行其他任何操作。

## 参考

https://www.ibm.com/developerworks/cn/aix/library/au-sshlocks/index.html

http://linux.vbird.org/linux_security/knockd.php