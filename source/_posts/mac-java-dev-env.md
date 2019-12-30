---
title: Mac 下的 Java 开发环境配置
date: 2019-11-25 20:54:43
tags:
- Java
---

## 安装 JAVA

直接安装最新版本 （ oracle jdk ），其他版本的可从官网下载安装

```
brew cask install java
```

安装选定版本 （ openjdk ）

```
brew tap homebrew/cask-versions
brew tap adoptopenjdk/openjdk
brew update
brew cask install adoptopenjdk/openjdk/adoptopenjdk8 
# 也可以安装 9/10/11/12/13 等
```

## 安装编译工具

安装 Maven

```
brew install maven
```

安装 Gradle

```
brew install gradle
```

