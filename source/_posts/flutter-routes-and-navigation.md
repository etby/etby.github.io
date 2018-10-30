---
title: 翻译《Flutter：路由和导航》
date: 2018-04-04 22:57:00
categories: Translate
tags:
  - Flutter
---

> 原文地址: https://proandroiddev.com/flutter-routes-and-navigation-69f128a9ea8f

## Flutter: 路由和导航

今天我学习关于路由和导航。在[Flutter](https://flutter.io/)，导航是内置的。它很简单，快速并且易于使用。当阅读完这篇文章，你讲学习如何使用导航和在不同页面传递数据。跟随说明，让我们开始编码……

## 布局

让我们从基础开始。创建一个 [Flutter app](https://flutter.io/get-started/test-drive/#terminal) 然后去到 main.dart 并删除所有的东西。我们开始从Flutter导入材料设计库。下一步我设置我们的 main 函数。 在 main 中，我们创建一个新的 [MaterialApp](https://docs.flutter.io/flutter/material/MaterialApp-class.html) 。在 Material App 中，我们设置首页属性为 new Landing() 或者任何你喜欢的名字。我们如果没有创建这个类的话将会发生错误，所以让我们来创建它。

创建一个 Statless 部件, 叫做 Loading 或者你设置给首页属性的名称。在 Loading 中，复写 build 方法然后创建一个新的  [Scaffold](https://docs.flutter.io/flutter/material/Scaffold-class.html)。设置 body 为新的 LoadingBody()。你还得创建一个  [AppBar](https://docs.flutter.io/flutter/material/AppBar-class.html) 并且设置页面标题。

你的代码应该看起来像这样

{% gist f32e57f4e7f73011cb66a6b062c589ca  %}

现在去创建 body。创建一个 Stateful 部件叫 LandingBody。我们将继续重写这个部件的 build 方法。这儿你能自由地去创建属于你的布局。确保包含一个按钮让我们能够连接导航。我将创建一个圆形头像和一个图标按钮。他们都在一个蓝色背景的容器中。

{% gist 04a825625d1a7cc39137c0b03d53e5eb  %}

你的 app 现在应该看起来像这样

![](https://cdn-images-1.medium.com/max/2800/0*qlwJ-ibAAJ3nq0cV.png)

当首页创建好以后，我们将创建二个其他页面。创建两个新文件 page2.dart 和 page3.dart 在你的 lib 目录下。你可以创建任何你喜欢的布局， 这些页面都需要保护两个按钮，用来前进和后退。我将和上一个页面一样创建一个圆形头像和图标按钮在这两个页面。

Page 2

{% gist 9548617d91f190b9ed230486cb6620ab  %}

Page 3

{% gist f522a7a9ecb1031f557d2999f7a79779  %}

## 基础导航

当布局创建完成，我们现在能开始设置导航。首先，我们将设置一个基本导航在我们的 app 中移动，然后我们要在页面之间传递数据。

现在， 去 main.dart 并且 import 我们创建的两个页面。

{% gist 63c4a55205f4ae41b7fce77b4a1f1e61  %}

当两个页面导入之后，就该设置我们的路由了。MaterialApp 部件有个属性叫 [routes](https://docs.flutter.io/flutter/material/MaterialApp/routes.html)。我们将使用这个属性去制定我们的 app 中不同的路由。默认，这个部件分配 '/' 路由为首页。

路由属性包含一个字符串和一个 Widget Builder。这个字符串将是路由名和 Widget Builder 构造函数的参数 [MaterialPageRoute](https://docs.flutter.io/flutter/material/MaterialPageRoute-class.html) (它将用合适的动画和其他东西构建一个页面)

去设置它，导入这行代码到你的 MaterialApp 部件。

{% gist 400854e48f5ed28f0a9ca41038035f23  %}

你能看到，我们设置了导航名称并且为每一个路由创建了一个新页面。你也许注意到了，我们调用的部件是我们之前在不同页面中制定的类的名称。我们能这样做是因为我们导入了它们。

当导航的所有路由设置完成。在 Flutter，这个导航是管理所有路由的部件。它能增加一个新页面到屏幕和栈中，也能从栈里面删除一个页面返回上一个页面。在 main.dart ，我们将设置按压属性在我们的按钮上。

{% gist 0d2c2e38e9d736be2ed62e299a3b5306  %}

在上面的代码中， 我们有一个调用导航的函数。这个导航使用关键字导航进行调用。.of(context) 指定当前上下文。 .pushNamed() 告诉导航之后我们将导航的名字。如果正常，我们将导航到第二个页面。你也许注意到，传递给 .pushNamed() 和我们传递给路由设置的名称相同。

如果所有步骤都没错，当你点击你的按钮，你将去到第二个页面。

![](https://cdn-images-1.medium.com/max/2800/0*3J82-bfB4-vnBLAV.png)

现在你也许已经懂了如何回到上一个页面。完善这个，我们使用 pop 方法。在 page2.dart ，修改 你的按钮的onPressed 属性，之后，它看起来像下面的代码。

{% gist 3feb5a691c7ddc080f21083315ef9021  %}

既然我们已经到这里了，不妨配置第三个按钮让它导航到第三个页面。这是家庭作业，我将不会给你指导。如果你卡住了，留下评论我将尝试帮助你。如果你重新加载了 app，你的按钮一旦点击，应该带你到首页。

如果你去到第三页并且尝试使用 pop 返回 首页， 你将会注意到一个有趣的问题出现 :)。pop 方法将带你回到第二页，如果你尝试使用.pushNamed（）手动导航，则主页上会出现一个后退箭头。

![](https://cdn-images-1.medium.com/max/2800/0*J6CCehlnLcgbN7aE.png)

为了解决这个问题，并导航回栈中有多个页面的地方，请使用 pushNamedAndRemoveUntil。这将添加这个名字的路由并且移除其他所有导航。当其他导航被移除，这个返回按钮将消失。首先实现这个过程有点棘手，但在你使用它几次后变得容易。 在第3页上，修改现有的onPressed以反映下面的代码。

{% gist 2c6623c9d03c89e690b2d38e1f844726  %}

让我们分析这里发生的事情。我们像之前那样调用导航，但是将.pushNamedAndRemoveUntil添加到结尾而不是弹出或推送。PushNamedAndRemoveUntil 有两个参数，字符串类型的导航名称和路由类型的导航判断式。当判断式返回 false 时，所有先前的路由将从栈中删除，直到返回 true。在上面的代码中，你可以看到第一个参数，传入的是路由名称，然后是一个接收路由并且返回 false 的函数。第二个参数是一个判断式。当你从第三页面导航到首页时，页面顶部的箭头应该隐藏。

## 传递数据到不同页面

通过基础知识，让我们看看如何在两个页面之间传递数据。不同页面间传递数据需要我们手动使用 MaterialPageRoute 构建这个页面。在我们设置这个之前，让我们修改我们的首页和第二页面，当我们传递数据成功时，我们需要接收和显示。在 main.dart 中，我将在图标按钮上添加一个文字区域。提示文字为，它将请求用户输入他的姓名。我还设置每次输入文字时调用 onChange 属性。输入的值存储在叫 name 的变量中。

{% gist 219abbb0f402a4540406fcdb7f0b7891  %}

下一步，我们移动到第二页。我们将在这儿创建一个叫 name 的字符串变量，并且添加它到构造函数中。

{% gist 24e25976d2aa53107c675e1003c10168  %}

我们之后创建一个 [Text](https://docs.flutter.io/flutter/widgets/Text-class.html) 在圆形头像下面显示他。

{% gist 35da44861d0534da929fae812aa8072c  %}

设置完成之后，我们开始 MaterialPageRoute 上的工作。 返回 main.dart 修改 onPressed

从

{% gist 10b2c9e212033ba4ec39b42baaeca72d  %}

修改为

{% gist b5a71676c75948ee0681f0706250e6c3  %}

然后你将看到，MaterialPageRoute 接收一个有一个返回新部件的 build 方法的 builder 对象。在上面的例子中，我们创建了一个新的 Page2 并且将 TextField 中输入的名称传递给部件的 name 属性 (它来自部件的构造函数)。

如果你重新加载 app，你输入你的名称之后能在你导航到第二页的时候查看它。

这儿可以查看最终的产品

{% youtube Adf0374REVc  %}

## 结论

哇哦！今天我们学习了如何在 Flutter 中使用基本的导航。你看，它使用起来非常简单。但总的来说一些片段如果需要完全掌握还是需要一点小小的练习，练习是一种乐趣。如果你有任何问题，随便留下一条评论，我将在我有空闲的时候回复你。另外，这个项目的代码都在  [GitHub](https://github.com/Neevash/Flutter-Navigation-Playground). Clone，fork 和 修改 都随心所欲。感谢Vectorpocket，Kraphix和Freepik.com的Freepik为Circle Avatar中使用的惊人图片。

— [Nash](https://twitter.com/Nash_905?lang=en)
