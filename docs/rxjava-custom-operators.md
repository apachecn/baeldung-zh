# 在 RxJava 中实现自定义运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-custom-operators>

## 1。概述

在这个快速教程中，我们将展示如何使用 [RxJava](https://web.archive.org/web/20220625222813/https://github.com/ReactiveX/RxJava) 编写自定义操作符。

我们将讨论如何构建这个简单的操作符，以及一个转换器——作为一个类或者作为一个简单的函数。

## 2。Maven 配置

首先，我们需要确保在`pom.xml`中有`rxjava`依赖关系:

```java
<dependency>
    <groupId>io.reactivex</groupId>
    <artifactId>rxjava</artifactId>
    <version>1.3.0</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20220625222813/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.reactivex%22%20a%3A%22rxjava%22) 上查看`rxjava`的最新版本。

## 3。自定义操作员

**我们可以通过实现`[Operator](https://web.archive.org/web/20220625222813/http://reactivex.io/RxJava/1.x/javadoc/rx/Observable.Operator.html)`接口**来创建我们的自定义操作符，在下面的例子中，我们实现了一个简单的操作符，用于从`String`中删除非字母数字字符:

```java
public class ToCleanString implements Operator<String, String> {

    public static ToCleanString toCleanString() {
        return new ToCleanString();
    }

    private ToCleanString() {
        super();
    }

    @Override
    public Subscriber<? super String> call(final Subscriber<? super String> subscriber) {
        return new Subscriber<String>(subscriber) {
            @Override
            public void onCompleted() {
                if (!subscriber.isUnsubscribed()) {
                    subscriber.onCompleted();
                }
            }

            @Override
            public void onError(Throwable t) {
                if (!subscriber.isUnsubscribed()) {
                    subscriber.onError(t);
                }
            }

            @Override
            public void onNext(String item) {
                if (!subscriber.isUnsubscribed()) {
                    final String result = item.replaceAll("[^A-Za-z0-9]", "");
                    subscriber.onNext(result);
                }
            }
        };
    }
}
```

在上面的例子中，我们需要在应用我们的操作并向其发送项目之前检查订户是否被订阅，因为这是不必要的。

**我们还将实例创建限制为静态工厂方法，以在链接方法和使用静态导入时实现更加用户友好的可读性**。

现在，我们可以使用 [`lift`](https://web.archive.org/web/20220625222813/http://reactivex.io/RxJava/1.x/javadoc/rx/Observable.html#lift(rx.Observable.Operator)) 运算符轻松地将我们的自定义运算符与其他运算符链接起来:

```java
observable.lift(toCleanString())....
```

下面是对我们的自定义操作符的一个简单测试:

```java
@Test
public void whenUseCleanStringOperator_thenSuccess() {
    List<String> list = Arrays.asList("john_1", "tom-3");
    List<String> results = new ArrayList<>();
    Observable<String> observable = Observable
      .from(list)
      .lift(toCleanString());
    observable.subscribe(results::add);

    assertThat(results, notNullValue());
    assertThat(results, hasSize(2));
    assertThat(results, hasItems("john1", "tom3"));
}
```

## 4。变压器

我们也可以通过实现 [`Transformer`](https://web.archive.org/web/20220625222813/http://reactivex.io/RxJava/1.x/javadoc/rx/Observable.Transformer.html) 接口来创建我们的操作员:

```java
public class ToLength implements Transformer<String, Integer> {

    public static ToLength toLength() {
        return new ToLength();
    }

    private ToLength() {
        super();
    }

    @Override
    public Observable<Integer> call(Observable<String> source) {
        return source.map(String::length);
    }
}
```

注意，我们使用转换器`toLength`将我们的可观测值从`String`转换成它在`Integer`中的长度。

我们将需要一个 [`compose`](https://web.archive.org/web/20220625222813/http://reactivex.io/RxJava/1.x/javadoc/rx/Observable.html#compose(rx.Observable.Transformer)) 操作员来使用我们的变压器:

```java
observable.compose(toLength())...
```

这里有一个简单的测试:

```java
@Test
public void whenUseToLengthOperator_thenSuccess() {
    List<String> list = Arrays.asList("john", "tom");
    List<Integer> results = new ArrayList<>();
    Observable<Integer> observable = Observable
      .from(list)
      .compose(toLength());
    observable.subscribe(results::add);

    assertThat(results, notNullValue());
    assertThat(results, hasSize(2));
    assertThat(results, hasItems(4, 3));
}
```

`lift(Operator)`对可观察对象的订户进行操作，但`compose(Transformer)`对可观察对象本身进行操作。

当我们创建自定义操作符时，如果我们想对整个可观察对象进行操作，我们应该选择`Transformer`,如果我们想对可观察对象发出的项目进行操作，我们应该选择`Operator`

## 5。自定义运算符作为函数

我们可以将自定义操作符实现为一个函数，而不是`public class`:

```java
Operator<String, String> cleanStringFn = subscriber -> {
    return new Subscriber<String>(subscriber) {
        @Override
        public void onCompleted() {
            if (!subscriber.isUnsubscribed()) {
                subscriber.onCompleted();
            }
        }

        @Override
        public void onError(Throwable t) {
            if (!subscriber.isUnsubscribed()) {
                subscriber.onError(t);
            }
        }

        @Override
        public void onNext(String str) {
            if (!subscriber.isUnsubscribed()) {
                String result = str.replaceAll("[^A-Za-z0-9]", "");
                subscriber.onNext(result);
            }
        }
    };
};
```

这里有一个简单的测试:

```java
List<String> results = new ArrayList<>();
Observable.from(Arrays.asList("[[email protected]](/web/20220625222813/https://www.baeldung.com/cdn-cgi/l/email-protection)", "or-an?ge"))
  .lift(cleanStringFn)
  .subscribe(results::add);

assertThat(results, notNullValue());
assertThat(results, hasSize(2));
assertThat(results, hasItems("apple", "orange"));
```

类似地，以`Transformer`为例:

```java
@Test
public void whenUseFunctionTransformer_thenSuccess() {
    Transformer<String, Integer> toLengthFn = s -> s.map(String::length);

    List<Integer> results = new ArrayList<>();
    Observable.from(Arrays.asList("apple", "orange"))
      .compose(toLengthFn)
      .subscribe(results::add);

    assertThat(results, notNullValue());
    assertThat(results, hasSize(2));
    assertThat(results, hasItems(5, 6));
}
```

## 6。结论

在本文中，我们展示了如何编写 RxJava 操作符。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220625222813/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-operators)