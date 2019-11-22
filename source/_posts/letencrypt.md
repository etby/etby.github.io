---
title: 使用 Let‘s Encrypt 签发免费 SSL 证书
date: 2019-11-22 14:13:44
tags:
- DevOps
- Linux
- SSL
---

## 安装

### 添加 PPA 仓库

```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
```

### 安装 Certbot

```
sudo apt-get install certbot
```

## 签发证书

使用以下命令签发证书

```
sudo certbot certonly --manual --preferred-challenges dns -d gaobo.name
```

`certonly --manual` 代表手动签发证书，不进行其他配置工作。

`--preferred-challenges dns` 代表验证方式使用 DNS 进行域名所有权的验证，也可以选择 `http` 方式进行验证。

`-d gaobo.name` 则是域名配置，后面可以多次使用 `-d DOMAIN` 来配置多个域名。

### 修改 DNS 解析

在执行上面的命令之后，会有几步相关的操作让你确认。之后会给你一段口令让你设置到 DNS 解析上：

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.gaobo.name with the following value:

aO0HEtcVvHPPGY2a8JHEzjIXcDj8QD0kWb48Xmbh3g8

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

按照提示设置好之后，按回车键即可。

### 成功签发

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/gaobo.name/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/gaobo.name/privkey.pem
   Your cert will expire on 2020-02-20. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```

出现上面的提示之后就代表成功签发了，接下来只需要在 Web 服务器中进行配置即可。

## 服务器配置

### NGINX

如下所示，配置 `ssl_certificate` 和 `ssl_certificate_key` 即可。

```
server {
    listen 443 ssl;
    server_name gaobo.name;

    ssl_certificate /etc/letsencrypt/live/gaobo.name/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/gaobo.name/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8002;
        include proxy_params;
    }
}
```



## 参考

https://certbot.eff.org/docs/using.html#certbot-commands