---
title: 翻译《必要的DiffUtil》
categories: Translate
tags:
  - Android
date: 2018-04-05 21:27:37
---

> 原文: https://proandroiddev.com/diffutil-is-a-must-797502bc1149


## DiffUtil 是必要的!

新数据的第一次到达对你的空白的**RecyclerView.Adapter**是不重要的，仅仅需要消费事件并且设置所有数据。我过去花费了许多时间寻找当你的适配器不为空的时候新数据到达时，不太麻烦更新数据的方法。我查看了很多地方和书籍都没有找到，重复了许多次也是一样。

在v7支持库  **RecyclerView** 支持库的 *24.2.0* 版本有一个很简单的类叫**DiffUtil**。它拯救了我的生命，它避免了你去某个地方和书籍查找。

## 上代码

**DiffUtil** 需要得到关于你的两个列表的更多基本信息，像*数量* 和 *如何比较条目*。**DiffUtil** 提供了**DiffUtil.Callback **让我们用来比较列表的基本信息。

在我的下一个技巧中，我将在我的**Adapter**中使用简单的**Product**对象。

{% gist 30822a7eeb01969b1b49f1cc36b0316a %}

**Callback** 是一个抽象的类，它需要去重写关于两个列表数量和条目如何通过索引比较的方法。我不熟悉内部实现，只能添加一些空检查(并省略了一些)。当然，列表可以是任何类型，生成**Callback**只是用来增强条目的比较。

你也许注意到了*getChangePayload() *方法，它并不是抽象的。这个方法将在*areItemsTheSame()* 返回**true**的时候被调用，但是 *areContentsTheSame()* 返回**false**的时候我们虽然谈论着相同的条目，但是里面的字段数据可能已经改变。基本上，这个方法返回列表中存在差异的原因。

在这，我使用一个**Bundle **去返回比一个多的关于比较的条目不相同的原因。

一旦我们有一个回调，这非常简单。

```
@Override
public void onNewProducts(List<Product> newProducts) {
    DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(new 				ProductListDiffCallback(mProducts, newProducts));
    diffResult.dispatchUpdatesTo(mProductAdapter);
}
```

**DiffResult **对象从差异计算完成之后返回，它将分配所有的改变到适配器中。

### Payloads

从*getChangePayload()*返回的对象将被使用在**DiffResult**的*notifyItemRangeChanged(position, count, payload)*方法中，之后会在适配器调用*onBindViewHolder(… List<Object> payloads)*时被传入。

```
    @Override
    public void onBindViewHolder(ProductViewHolder holder, int position, List<Object> payloads) {
        if(payloads.isEmpty()) return;
        else{
            Bundle o = (Bundle) payloads.get(0);
            for (String key : o.keySet()) {
                if(key.equals(KEY_DISCOUNT)){
                    //TODO lets update blink discount textView :)
                }else if(key.equals(KEY_PRICE)){
                    //TODO lets update and change price color for some time
                }else if(key.equals(KEY_REVIEWS_COUNT)){
                    //TODO just update the review count textview
                }
            }
        }
    }
```

### 巨大的列表

文档提示**DiffUtil**在巨大的数据集上会可能会花一点时间，建议将计算过程防止在后台线程。这有一个小小的使用 RxJava 的例子：

{% gist eb7cbb3122ff702081bb7cd14900f4b1 %}

从主线程分发**DiffResult **：

```
    @Override
    public void onNext(DiffUtil.DiffResult result) {
        result.dispatchUpdatesTo(mProductAdapter);
    }
```

## 结论:

**DiffUtil** 计算给定的两个列表的不同之处，旧列表是当前正在展示的，新列表是你刚刚获得的。**DiffResult**包含一些能够更新给你的适配器的数据。
