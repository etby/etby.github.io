---
title: Dart 语言关于导包的一点小坑
date: 2018-05-03 14:42:44
categories: OpenSource
tags:
- Flutter
---

## 问题

最近在用Flutter开发一个小一点的应用，有一些需要用到第三方框架的地方。但是当我按照文档接入之后，却出现了不该出现的问题。

在使用Fluro框架和Flutter_Redux框架的时候，存储在组件列表中的数据存储对象却无法找到。但是，越过App层的Fluro直接使用redux却可用。

在这还得吐槽一下Flutter，由于基础设施不完善，DEBUG起来非常麻烦，我也只能去求助大牛了。

## 解决过程

初步怀疑是flutter_redux框架的bug，然后就提了一个issue。具体：

https://github.com/brianegan/flutter_redux/issues/41

之后大牛回复我希望能提供一个demo，我就建了一个项目，按照错误代码去重建。结果无论如何都无法重现，但是我又不能将旧代码逻辑转移过来，这样也许会再次产生，比较产生bug的因素目前还不明确。

我就只能将私有仓库公开，之后提交给大牛，让他帮我看看了。

没想到大牛直接给我提了一个 pull request，感谢大牛。

## 解决方案

大牛给我的回答如下:

Hey there! The problem you're experiencing is a confusing issue with how Dart handles imports. By changing the imports to package-relative paths, I was able to get the app working.

General rule: always use import 'package:bungami_list:path/to/file rather than import 'path/to/file.dart

This is definitely a confusing part of Dart, and it might be helpful to read this thread on more info: dart-lang/sdk#32042 (comment)

大体是说，是因为import包的问题：dart针对于URI不同的库就算代码相同，也不会认为是同一个库。所有尽量使用import全路径的方式来进行导包，就算是在同一个项目中。更多的信息可以看这里：https://github.com/dart-lang/sdk/issues/32042#issuecomment-363107221

## 最后

经过这次的问题解决，我也获得了很多经验：

- 感谢开源社区
- 要多参与开源
- 当你有一些问题怀疑是由于开源库引起的，可以尝试去阅读源代码解决。如果源代码无法解决你的问题，可以直接提Issue给具体的项目，求职大牛帮忙。
- 开源的项目并不只需要解决代码方面的问题，使用方面的问题也是需要解决的。最好的方式是完整优秀的文档，对于无法文档化的问题，则集合大家的力量去解决。