# RxJava 中 Flatmap 和 Switchmap 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-flatmap-switchmap>

## 1。概述

RxJava 提供了各种操作符来将一个可观察对象发出的项目转换成其他可观察对象。最受欢迎的两个运算符是`flatMap`和`switchMap`。对于反应式编程的初学者来说，这两者之间的区别通常很难理解。

关于 RxJava 的介绍，参考[本文](/web/20220626195311/https://www.baeldung.com/rx-java)。

在本教程中，我们将通过一个简单的例子来理解其中的区别。

## 2。`flatMap`

**操作符`flatMap`使用提供的函数将从源可观测值返回的每一项转换成独立的可观测值，然后将所有的可观测值合并成一个可观测值**。观察值合并的顺序不能保证与源中的顺序相同`Observable.`

我们以一个搜索引擎为例。假设我们希望在键入单词的每个字符后立即显示搜索结果:

为了简单起见，我们将搜索查询输入作为单词列表。

此外，我们总是为每个单词返回两个搜索结果。

```java
// given
List<String> actualOutput = new ArrayList<>();
TestScheduler scheduler = new TestScheduler();
List<String> keywordToSearch = Arrays.asList("b", "bo", "boo", "book", "books");

// when
Observable.fromIterable(keywordToSearch)
  .flatMap(s -> Observable.just(s + " FirstResult", s + " SecondResult")
    .delay(10, TimeUnit.SECONDS, scheduler))
  .toList()
  .doOnSuccess(s -> actualOutput.addAll(s))
  .subscribe();

scheduler.advanceTimeBy(1, TimeUnit.MINUTES);

// then
assertThat(actualOutput, hasItems("b FirstResult", "b SecondResult",
  "boo FirstResult", "boo SecondResult",
  "bo FirstResult", "bo SecondResult",
  "book FirstResult", "book SecondResult",
  "books FirstResult", "books SecondResult"));
```

请注意，每次运行的顺序并不总是相同的。

## 3。`switchMap`

**`switchMap`运算符与`flatMap`类似，除了它只保留最新的可观测结果，而丢弃之前的结果。**

让我们改变一下我们的要求，我们希望只获得最后一个完整单词(在本例中是“books”)的搜索结果，而不是部分查询字符串。为了实现这一点，我们可以使用`switchMap`。

如果我们在上面的代码示例中用`switchMap`替换`flatMap`，下面的断言将是有效的:

```java
assertEquals(2, actualOutput.size());
assertThat(actualOutput, hasItems("books FirstResult", "books SecondResult"));
```

正如我们在这里看到的，我们只得到一个包含来自源可观察对象的最新输入项的单个可观察对象。所有以前的结果都被丢弃。

## 4。结论

综上所述，`switchMap`与`flatMap`的不同之处在于，它只保留将提供的函数应用到源可观测发出的最新项的输出，`flatMap`则保留所有结果，并以交错方式返回，不保证顺序。

和往常一样，本文中使用的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220626195311/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-core)