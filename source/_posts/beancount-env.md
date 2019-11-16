---
title: BeanCount 记账环境
date: 2019-11-15 15:12:15
tags:
- financial
- docker
---

## 简单介绍

beancount 是一种复式记账的语言，可以将交易记录在文本文件中，可以做很多的脚本处理，还有 Web 界面可以查看账目。

### 复式记账

基于会计恒等式产生的记账形式，每笔交易的结果至少被记录在一个借方和一个贷方的账户，且该笔交易的借贷双方总额相等，即 ”有借必有贷，借贷必相等“ 。

> 资产 = 负债 + 所有者权益
>
> 资产 = 负债 + 所有者权益 + （收入 - 费用）

复式记账可以做到对资金的强掌控，每一笔资金的来源和去向一清二楚。

## 本地安装

```
# 本文所有 python 环境都是 python3
pip3 install virtualenv

# 进入自己想放置环境配置的地方
python -m venv BEANCOUNT
source BEANCOUNT/bin/active
pip install beancount
pip install fava
```

安装完成即可开始记账，如何记账可以参考文章最下面的几篇博客。

## 网页访问

我是用 Docker 部署 fava 来使用的，一般小笔的交易直接在 fava 的编辑器中打开账本进行记账。

#### 建立部署文件夹

```
# 进入账本文件夹中执行
mkdir deploy && cd deploy
```

#### 记录依赖

```
# 进入 venv
source BEANCOUNT/bin/active
# 将正确的依赖配置记录下来
pip freeze > requirements.txt
```

#### Dockerfile

创建 Dockerfile 文件，然后输入以下内容

```
FROM python

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

ENV FAVA_HOST=0.0.0.0
VOLUME /data

CMD ["fava", "/data/main.bean"]
```

#### Docker Compose

创建 docker-compose.yml 文件，输入以下内容（具体配置可按自己的需求改变）

```
version: "3.3"
services:

  fava:
    build: .
    ports:
      - "127.0.0.1:8888:5000"
    volumes:
      - ..:/data
```

注意：

- 挂载的 volumes 中的 .. 是因为上面这些文件都在账本目录下的 deploy 文件夹中，.. 正好是账本文件夹。
- 主账本名称应该为 main.bean

#### 部署网站

```
docker-compose up -d --buil
```

之后可以访问： http://localhost:8888/ 查看账本是否部署成功

## 参考

- https://wzyboy.im/post/1063.html
- https://yuchi.me/post/beancount-intro/