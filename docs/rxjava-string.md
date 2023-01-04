# 可观察到的 xJava

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-string>

## 1。`StringObservable`简介

使用`RxJava`中的`String`序列可能具有挑战性；幸运的是`[RxJavaString](https://web.archive.org/web/20221128050212/https://github.com/ReactiveX/RxJavaString)`为我们提供了所有必需的设施。

在本文中，我们将讨论包含一些有用的`String`操作符的`StringObservable`。因此，在开始之前，建议先看一下[简介](/web/20221128050212/https://www.baeldung.com/rx-java) [简介](/web/20221128050212/https://www.baeldung.com/rx-java) [至](/web/20221128050212/https://www.baeldung.com/rx-java) [RxJ](/web/20221128050212/https://www.baeldung.com/rx-java) [ava](/web/20221128050212/https://www.baeldung.com/rx-java) 。

## 2。Maven 设置

首先，让我们将`RxJavaString`包括在我们的依赖项中:

```
<dependency>
  <groupId>io.reactivex</groupId>
  <artifactId>rxjava-string</artifactId>
  <version>1.1.1</version>
</dependency>
```

最新版本的 rxjava `-string`可以在 [Maven Central](https://web.archive.org/web/20221128050212/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.reactivex%22%20AND%20a%3A%22rxjava-string%22) 上获得。

## 3。`StringObservable`

**`StringObservable`是一个方便的操作符，用于表示潜在的无限序列的编码`Strings`** `.`

操作符`from`读取一个输入流，创建一个`Observable`,它发出以字符为界的字节数组序列:

![Reading an infinite stream](img/10abb32c35b0454dab48f8c46428ead3.png)

由[react vex . io](https://web.archive.org/web/20221128050212/http://reactivex.io/)，在[下使用 CC-BY](https://web.archive.org/web/20221128050212/https://creativecommons.org/licenses/by/3.0/)

我们可以使用一个`from`操作符从一个`InputStream`直接创建一个`Observable`:

```
TestSubscriber testSubscriber = new TestSubscriber();
ByteArrayInputStream is = new ByteArrayInputStream("Lorem ipsum loream, Lorem ipsum lore".getBytes());
Observable<byte[]> observableByteStream = StringObservable.from(is);

// emits 8 byte array items
observableByteStream.subscribe(testSubscriber);
```

## 4。将字节转换成`Strings`

可以使用`decode`和`encode` 操作符对来自不同字符集的无限序列进行编码/解码。

顾名思义，这些函数将简单地创建一个发出字节数组编码或解码序列的`Observable`或`Strings,`，因此，如果我们需要处理不同字符集中的`Strings`，我们可以使用

 **解码一个字节数组`Observable`:

```
TestSubscriber testSubscriber = new TestSubscriber();
ByteArrayInputStream is = new ByteArrayInputStream(
  "Lorem ipsum loream, Lorem ipsum lore".getBytes());
Observable<byte[]> byteArrayObservable = StringObservable.from(is);
Observable<String> stringObservable = StringObservable
  .decode(byteArrayObservable, StandardCharsets.UTF_8);

// emits UTF-8 decoded strings,"Lorem ipsum loream, Lorem ipsum lore"
stringObservable.subscribe(testSubscriber);
```

## 5。`Strings`分裂

`StringObservable`也有一些方便的操作符来拆分`String`序列:`split`和`byLine,`都创建了一个新的`Observable`，它按照一种模式来分块输入数据输出项目；

![StringObservable.split splitting strings in rxjava](img/f2ae6641499765d7489b5d9849d4ec5e.png)

由[react vex . io](https://web.archive.org/web/20221128050212/http://reactivex.io/)

```
TestSubscriber testSubscriber = new TestSubscriber();
Observable<String> sourceObservable = Observable.just("Lorem ipsum loream,Lorem ipsum ", "lore");
Observable<String> splittedObservable = StringObservable.split(sourceObservable, ",");

// emits 2 strings "Lorem ipsum loream", "Lorem ipsum lore"
splittedObservable.subscribe(testSubscriber);
```

## 6。连接字符串

与前一节的操作符互补的是`join`和`stringConcat`，它们连接来自`String` `Observable`的项目，发出一个给定分隔符的单个字符串。

另外，请注意，这些将在发出输出之前消耗所有项目。

![StringObservable.join joining strings](img/cbdf8a219eaefe307124aa9d9a5868c3.png)

由[react vex . io](https://web.archive.org/web/20221128050212/http://reactivex.io/)

```
TestSubscriber testSubscriber = new TestSubscriber();
Observable<String> sourceObservable = Observable.just("Lorem ipsum loream", "Lorem ipsum lore");
Observable<String> joinedObservable = StringObservable.join(sourceObservable, ",");

// emits single string "Lorem ipsum loream,Lorem ipsum lore"
joinedObservable.subscribe(testSubscriber);
```

## 7。结论

对`StringObservable`的简要介绍展示了使用`RxJavaString.`操纵`String`的几个用例

本教程中的例子和其他关于如何使用`StringObservable` 操作符的例子可以在 GitHub 的[中找到。](https://web.archive.org/web/20221128050212/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-observables)**