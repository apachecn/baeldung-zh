# RxJava 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rx-java>

## 1。概述

在本文中，我们将重点关注在 Java 中使用反应式扩展(Rx)来组合和消费数据序列。

乍一看，该 API 可能看起来类似于 Java 8 Streams，但实际上，它更加灵活和流畅，使其成为一种强大的编程范式。

如果你想了解更多关于 RxJava 的内容，请查看这篇文章。

## 2。设置

为了在我们的 [Maven](https://web.archive.org/web/20220627173244/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.reactivex%22) 项目中使用 RxJava，我们需要向我们的`pom.xml:`添加以下依赖项

```
<dependency>
    <groupId>io.reactivex</groupId>
    <artifactId>rxjava</artifactId>
    <version>${rx.java.version}</version>
</dependency>
```

或者，对于 Gradle 项目:

```
compile 'io.reactivex.rxjava:rxjava:x.y.z'
```

## 3。功能反应概念

一方面， **`functional programming`是通过组合纯功能、避免共享状态、可变数据和副作用来构建软件的过程。**

另一方面， **`reactive programming`是一种异步编程范例，关注数据流和变化的传播。**

总之，`functional reactive programming`形成了功能性和反应性技术的组合，可以代表事件驱动编程的一种优雅方法——其值随着时间的推移而变化，并且消费者在数据到来时做出反应。

这项技术将核心原则的不同实现结合在一起，一些作者提出了一个文档，定义了描述新型应用程序的通用词汇。

### 3.1。反应宣言

[Reactive Manifesto](https://web.archive.org/web/20220627173244/http://www.reactivemanifesto.org/) 是一个在线文档，为软件开发行业中的应用程序制定了一个高标准。简而言之，反应式系统是:

*   响应迅速——系统应该及时响应
*   消息驱动——系统应该在组件之间使用异步消息传递来确保松耦合
*   弹性–系统应在高负载下保持响应
*   弹性–当某些组件出现故障时，系统应保持响应

## 4。可观测量

使用`Rx:`时，需要了解两种关键类型

*   `Observable` 表示可以从数据源获取数据的任何对象，并且其状态可能会引起其他对象的兴趣
*   `observer` 是任何希望在另一个对象的状态改变时得到通知的对象

一个`observer`订阅一个`Observable`序列。**序列一次向`observer`发送一个项目。**

`observer`在处理下一个之前处理每一个。如果许多事件异步进入，它们必须存储在队列中或被丢弃。

在`Rx`中，一个`observer`将永远不会在一个项目无序的情况下被调用，也不会在对前一个项目的回调返回之前被调用。

### 4.1。`Observable`种类

有两种类型:

*   支持异步执行，并允许在事件流中的任何点取消订阅。在本文中，我们将主要关注这种类型

*   所有的`onNext`观察者调用将是同步的，并且不可能在事件流中间取消订阅。我们总是可以将一个`Observable`转换成一个`Blocking Observable`，使用方法`toBlocking:`

```
BlockingObservable<String> blockingObservable = observable.toBlocking();
```

### 4.2。操作员

**An `operator`是一个将一个`O` `bservable`(源)作为第一个参数并返回另一个`Observable`(目的)的函数。然后，对于源 observable 发出的每个项目，它将对该项目应用一个函数，然后在目的地`Observable`发出结果。**

`Operators`可以链接在一起，创建基于特定标准过滤事件的复杂数据流。多个运算符可以应用于同一个`observable`。

一个`Observable`发射物品的速度比一个`operator`或者`observer`消耗物品的速度快，这种情况并不难出现。你可以在这里阅读更多关于背压[的内容。](/web/20220627173244/https://www.baeldung.com/rxjava-backpressure)

### 4.3。创建可观察对象

基本操作符`just` 产生一个`Observable`，它在完成之前发出一个通用实例，字符串`“Hello”.` 当我们想从`Observable`中获取信息时，我们实现一个`observer`接口，然后在期望的`Observable:`上调用 subscribe

```
Observable<String> observable = Observable.just("Hello");
observable.subscribe(s -> result = s);

assertTrue(result.equals("Hello"));
```

### 4.4。`OnNext, OnError,`和`OnCompleted`

在`observer`接口上有三个我们想要了解的方法:

1.  每次有新的事件发布到附加的`Observable`时，`OnNext` 在我们的`observer`上被调用。这是我们对每个事件执行一些操作的方法
2.  当与一个`Observable`相关联的事件序列完成时，`OnCompleted` 被调用，这表明我们不应该期望对我们的观察者有更多的`onNext`调用
3.  当在`RxJava`框架代码或我们的事件处理代码中抛出未处理的异常时，调用`OnError`

`Observables` `subscribe`方法的返回值是一个`subscribe`接口:

```
String[] letters = {"a", "b", "c", "d", "e", "f", "g"};
Observable<String> observable = Observable.from(letters);
observable.subscribe(
  i -> result += i,  //OnNext
  Throwable::printStackTrace, //OnError
  () -> result += "_Completed" //OnCompleted
);
assertTrue(result.equals("abcdefg_Completed"));
```

## 5。可观察转换和条件运算符

### 5.1。`Map`

m `ap operator` 通过对每个项目应用一个函数来转换由一个`Observable`发出的项目。

假设有一个声明的字符串数组，其中包含字母表中的一些字母，我们希望以大写形式打印它们:

```
Observable.from(letters)
  .map(String::toUpperCase)
  .subscribe(letter -> result += letter);
assertTrue(result.equals("ABCDEFG"));
```

每当我们以嵌套的`Observables.`结束时，`The flatMap`可以用来展平`Observables`

更多关于`map` 和`flatMap` 区别的细节可以在[这里](/web/20220627173244/https://www.baeldung.com/java-difference-map-and-flatmap)找到。

假设我们有一个从字符串列表中返回`Observable<String>` 的方法。现在我们将根据`Subscriber` 看到的内容为新`Observable` 的每个字符串打印标题列表:

```
Observable<String> getTitle() {
    return Observable.from(titleList);
}
Observable.just("book1", "book2")
  .flatMap(s -> getTitle())
  .subscribe(l -> result += l);

assertTrue(result.equals("titletitle"));
```

### 5.2。`Scan`

`scan operator a`对由`Observable`顺序发出的每个项目应用一个函数，并发出每个连续的值。

它允许我们将状态从一个事件延续到另一个事件:

```
String[] letters = {"a", "b", "c"};
Observable.from(letters)
  .scan(new StringBuilder(), StringBuilder::append)
  .subscribe(total -> result += total.toString());
assertTrue(result.equals("aababc"));
```

### 5.3。`GroupBy`

`Group by` 操作符允许我们将输入`Observable`中的事件分类成输出类别。

假设我们创建了一个从 0 到 10 的整数数组，然后应用`group by`将它们分成类别`even`和`odd`:

```
Observable.from(numbers)
  .groupBy(i -> 0 == (i % 2) ? "EVEN" : "ODD")
  .subscribe(group ->
    group.subscribe((number) -> {
        if (group.getKey().toString().equals("EVEN")) {
            EVEN[0] += number;
        } else {
            ODD[0] += number;
        }
    })
  );
assertTrue(EVEN[0].equals("0246810"));
assertTrue(ODD[0].equals("13579"));
```

### 5.4。`Filter`

操作员`filter` 只从`observable`中发出那些通过`predicate`测试的物品。

因此，让我们在一个整数数组中过滤奇数:

```
Observable.from(numbers)
  .filter(i -> (i % 2 == 1))
  .subscribe(i -> result += i);

assertTrue(result.equals("13579"));
```

### 5.5。条件运算符

`DefaultIfEmpty` 从源`Observable`发出项目，如果源`Observable`为空，则发出默认项目:

```
Observable.empty()
  .defaultIfEmpty("Observable is empty")
  .subscribe(s -> result += s);

assertTrue(result.equals("Observable is empty"));
```

以下代码发出字母表'`a'` 的第一个字母，因为数组`letters` 不为空，这是它在第一个位置包含的内容:

```
Observable.from(letters)
  .defaultIfEmpty("Observable is empty")
  .first()
  .subscribe(s -> result += s);

assertTrue(result.equals("a"));
```

`TakeWhile` 操作员在指定条件变为假后丢弃由`Observable`发出的项目；

```
Observable.from(numbers)
  .takeWhile(i -> i < 5)
  .subscribe(s -> sum[0] += s);

assertTrue(sum[0] == 10);
```

当然，还有更多其他运营商可以满足我们的需求，如`Contain, SkipWhile, SkipUntil, TakeUntil,` 等。

## 6。可连接的可观测量

**一个`ConnectableObservable`类似于一个普通的`Observable`，除了当它被订阅时它不开始发射项目，而是只有当`connect`操作符被应用到它时才开始发射项目。**

这样，我们可以在`Observable`开始发送项目之前，等待所有预期的观察者订阅`Observable`:

```
String[] result = {""};
ConnectableObservable<Long> connectable
  = Observable.interval(200, TimeUnit.MILLISECONDS).publish();
connectable.subscribe(i -> result[0] += i);
assertFalse(result[0].equals("01"));

connectable.connect();
Thread.sleep(500);

assertTrue(result[0].equals("01"));
```

## 7.单一的

`Single` 就像一个`Observable`，它不是发出一系列的值，而是发出一个值或一个错误通知。

有了这个数据源，我们只能用两种方法订阅:

*   `OnSuccess` 返回一个`Single`,它也调用我们指定的方法
*   `OnError` 还返回一个`Single`,它会立即通知订阅者一个错误

```
String[] result = {""};
Single<String> single = Observable.just("Hello")
  .toSingle()
  .doOnSuccess(i -> result[0] += i)
  .doOnError(error -> {
      throw new RuntimeException(error.getMessage());
  });
single.subscribe();

assertTrue(result[0].equals("Hello"));
```

## 8。主题

一个`Subject` 同时是两个元素，一个`subscriber`和一个`observable`。作为订阅者，一个主题可以用来发布来自多个可观察对象的事件。

因为它也是可观察的，来自多个订阅者的事件可以作为它的事件重新发送给任何观察它的人。

在下一个示例中，我们将了解观察者如何能够看到他们订阅后发生的事件:

```
Integer subscriber1 = 0;
Integer subscriber2 = 0;
Observer<Integer> getFirstObserver() {
    return new Observer<Integer>() {
        @Override
        public void onNext(Integer value) {
           subscriber1 += value;
        }

        @Override
        public void onError(Throwable e) {
            System.out.println("error");
        }

        @Override
        public void onCompleted() {
            System.out.println("Subscriber1 completed");
        }
    };
}

Observer<Integer> getSecondObserver() {
    return new Observer<Integer>() {
        @Override
        public void onNext(Integer value) {
            subscriber2 += value;
        }

        @Override
        public void onError(Throwable e) {
            System.out.println("error");
        }

        @Override
        public void onCompleted() {
            System.out.println("Subscriber2 completed");
        }
    };
}

PublishSubject<Integer> subject = PublishSubject.create(); 
subject.subscribe(getFirstObserver()); 
subject.onNext(1); 
subject.onNext(2); 
subject.onNext(3); 
subject.subscribe(getSecondObserver()); 
subject.onNext(4); 
subject.onCompleted();

assertTrue(subscriber1 + subscriber2 == 14)
```

## 9。资源管理

操作允许我们将资源，如 JDBC 数据库连接、网络连接或打开的文件与我们的观察对象相关联。

在这里，我们在评论中介绍了实现这一目标所需的步骤，以及一个实施示例:

```
String[] result = {""};
Observable<Character> values = Observable.using(
  () -> "MyResource",
  r -> {
      return Observable.create(o -> {
          for (Character c : r.toCharArray()) {
              o.onNext(c);
          }
          o.onCompleted();
      });
  },
  r -> System.out.println("Disposed: " + r)
);
values.subscribe(
  v -> result[0] += v,
  e -> result[0] += e
);
assertTrue(result[0].equals("MyResource"));
```

## 10。结论

在本文中，我们讨论了如何使用 RxJava 库，以及如何探索它最重要的特性。

该项目的完整源代码，包括这里使用的所有代码样本，可以在 Github 上找到[。](https://web.archive.org/web/20220627173244/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-core)