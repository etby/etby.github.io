---
title: 翻译《Flutter：从网络缓存懒加载数据》
tags:
  - Flutter
categories: Translate
date: 2018-04-02 21:48:56
---


> 原文地址: https://proandroiddev.com/flutter-lazy-loading-data-from-network-with-caching-b7486de57f11

![](https://cdn-images-1.medium.com/max/2000/1*in7MRIAKfRn-qDgJKc9XVw.jpeg)

首先，我不是很熟悉 Flutter，因此在Flutter专家眼中许多地方也许比较幼稚、低效或不够优雅。不论如何， 我们都是从那个阶段开始的，所以让我们从脏脏的地方开始。

让我们开始一个练习, 目标是一个非常典型的任务 : 从网络懒加载一个集合数据并且显示到它们到手机的列表上. 数据可以从后端获取, 有一些微小的限制:

  1. 假如我们需要展示碰巧在一个'页面'中的 #2 和 #3 , 我们不能发送两次相同的请求.
  2. 假如我们已经加载了一个特殊的条目, 当他滑出列表之后再次展示 (或者也许这个View已经被RecycleView回收) , 然后短时间内需要再次展示之前的条目, 这时候我不想发送其他的网络请求. 是的, 我们需要一些缓存.

我们必须怎么做呢? 假设有这样一个API, 通过它能够简单的拉取页面数据:

```
import 'dart:async';
import 'dart:convert';
import 'dart:io';
import 'package:flute/model/Products.dart';

class Api {
  final String API_KEY = "<api_key>";
  final String BASE_URL =
      "https://company.com/_ah/api/service/v1/";

  @override
  Future<Products> getProducts(int pageNumber, int pageSize) async {
    final url = "${BASE_URL}products/$API_KEY/$pageNumber/$pageSize";
    final httpClient = new HttpClient();

    try {
      var request = await httpClient.getUrl(Uri.parse(url));
      var response = await request.close();

      if (response.statusCode == HttpStatus.OK) {
        var json = await response.transform(UTF8.decoder).join();
        var data = JSON.decode(json);

        return Products.fromMap(data); // We'll see it soon
      } else {
        log("Failed http call."); // Perhaps handle it somehow
      }
    } catch (exception) {
      log(exception.toString());
    }
    return null;
  }
}
```

它很简洁, 只有一个公开的异步方法. 现在让我们创建能够解析JSON响应填充自己的一个产品类和产品页对象模型:

```
class Product {
  final String productId;
  final String productName;
  final String price;
  final String productImage;
  // Perhaps some more

  const Product({
    this.productId,
    this.productName,
    this.price,
    this.productImage
  });

  Product.fromMap(Map<String, dynamic>  map) :
    productId = map['productId'],
    productName = map['productName'],
    price = map['price'],
    productImage = map['productImage'],
}
```

... 和 …

```
class Products {
  final List<Product> products;
  final int totalProducts;
  final int pageNumber;
  final int pageSize;

  Products({this.products, this.totalProducts, this.pageNumber, this.pageSize});

  Products.fromMap(Map<String, dynamic> map)
      : products = new List<Product>.from(map['products'].map((product) => new Product.fromMap(product))),
        totalProducts = map['totalProducts'],
        pageNumber = map['pageNumber'],
        pageSize = map['pageSize'];
}
```

现在, 我们有必要在页面中刷新数据, 但是我们应用的其他部分需要知道这个吗? 答案肯定是不需要, 我们必须有一个通过索引获取数据的抽象层,  让他来判断是否需要进行网络请求, 或者使用本地可供使用的缓存数据. 我们将这个层叫做Repository; 让我们开始为Repository本身定义接口, 并且我保证我们会更接近一些有趣的东西,  我会定义: 

```
abstract class Repository {
  Future<Product> getProduct(int index);
}
```

和 Cache:

```
abstract class Cache<T> {
  Future<T> get(int index);
  put(int index, T object);
}
```

现在, 定义分页和缓存的抽象逻辑. 这很简单 - 每次在App中请求其他分页的产品数据， 我们首先在缓存中查找，如果存在, 我们就使用它. 假如不存在并且我们没有正在向后端发送请求, 我们将请求它并且返回一个 Feture 对象给调用的地方. 然后当这个页面的数据从服务器返回之后, 我们完成所有阻塞的请求并且解析这个页面的数据. 听起来很复杂? 我保证实际其实很简单:

```

class CachingRepository extends Repository {
  final int pageSize;
  final Cache<Product> cache;
  final Api api = Api();

  final pagesInProgress = Set<int>();
  final pagesCompleted = Set<int>();
  final completers = HashMap<int, Set<Completer>>();

  int totalProducts;

  CachingRepository({this.pageSize, this.cache});

  @override
  Future<Product> getProduct(int index) {
    final pageIndex = pageIndexFromProductIndex(index);

    if (pagesCompleted.contains(pageIndex)) {
      return cache.get(index);
    } else {
      if (!pagesInProgress.contains(pageIndex)) {
        pagesInProgress.add(pageIndex);
        var future = api.getProducts(pageIndex, pageSize);
        future.asStream().listen(onData);
      }
      return buildFuture(index);
    }
  }

  Future<Product> buildFuture(int index) {
    var completer = Completer<Product>();

    if (completers[index] == null) {
      completers[index] = Set<Completer>();
    }
    completers[index].add(completer);

    return completer.future;
  }

  void onData(Products products) {
    if (products != null) {
      totalProducts = products.totalProducts;
      pagesInProgress.remove(products.pageNumber);
      pagesCompleted.add(products.pageNumber);

      for (int i = 0; i < pageSize; i++) {
        int index = products.pageSize * products.pageNumber + i;
        Product product = products.products[i];

        cache.put(index, product);
        Set<Completer> comps = completers[index];

        if (comps != null) {
          for (var completer in comps) {
            completer.complete(product);
          }
          comps.clear();
        }
      }
    } else {
      log("CachingRepository.onData(null)!!!");
    }
  }

  int pageIndexFromProductIndex(int productIndex) {
    return productIndex ~/ pageSize;
  }
}
```

每次我们想展示一个产品的时候, 我们会设法获取它; 可是等等! 我们的 Repository 返回的不是一个 Product, 它是一个 Future 对象, 所以我们应该如何在屏幕上展示它? 或许, 可以在等待服务器响应的时候监听他的流? 这是一个选择, 但是在Flutter上我们有更好的选择, 它叫FutureBuilder, 它为我们提供了所有的细节 :

```
Widget _buildProductRow(Future<Product> productFuture) {
      return new FutureBuilder<Product>(
        future: productFuture,
        builder: (BuildContext context, AsyncSnapshot<Product> snapshot) {
          if (snapshot.hasData) {
            return _buildProductCard(snapshot.data);
          } else {
            return new LinearProgressIndicator();
          }
        },
      );
    }

Widget _buildProductCard(Product product) {
    return GestureDetector(
      onTap: () {
        showProductDetails();
      },
      child: new Row(children: [
        new Container(
          height: 64.0,
          width: 64.0,
          child: new Image.network(product.productImage),
        ),
        new Flexible(
          child: new Text(
            product.productName,
            maxLines: 3,
            overflow: TextOverflow.ellipsis,
            style: new TextStyle(fontWeight: FontWeight.bold),
          ),
        ),
        new Container(
            child: new Text(product.price)),
      ]),
    );
  }
```

加入假数据, 让我们看看:

![](https://cdn-images-1.medium.com/max/3480/1*q_NW5tWwCuhDCtrpm71U2A.png)

当然, 这是一个快速简陋的解决方案. 没有任何错误处理. 有些地方能做的更好. 但是, 它至少能够工作, 所有代码能在这儿找到:

 [https://github.com/chelomin/flute/tree/Medium_v1](https://github.com/chelomin/flute/tree/Medium_v1)

感谢阅读, 欢迎任何的反馈和夸奖!
