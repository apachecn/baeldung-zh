# RxJava 一个可观察的多订户

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-multiple-subscribers-observable>

## 1.概观

多个订阅者的默认行为并不总是令人满意的。在本文中，我们将介绍如何改变这种行为，并以适当的方式处理多个订阅者。

但是首先，让我们看看多个订户的默认行为。

## 2.默认行为

假设我们有以下的`Observable`:

```java
private static Observable getObservable() {
    return Observable.create(subscriber -> {
        subscriber.onNext(gettingValue(1));
        subscriber.onNext(gettingValue(2));

        subscriber.add(Subscriptions.create(() -> {
            LOGGER.info("Clear resources");
        }));
    });
}
```

一旦`Subscriber` s 订阅，就会发出两个元素。

在我们的示例中，我们有两个`Subscriber`:

```java
LOGGER.info("Subscribing");

Subscription s1 = obs.subscribe(i -> LOGGER.info("subscriber#1 is printing " + i));
Subscription s2 = obs.subscribe(i -> LOGGER.info("subscriber#2 is printing " + i));

s1.unsubscribe();
s2.unsubscribe();
```

想象一下，获取每个元素是一项代价高昂的操作——例如，它可能包括密集的计算或打开 URL 连接。

为了简单起见，我们只返回一个数字:

```java
private static Integer gettingValue(int i) {
    LOGGER.info("Getting " + i);
    return i;
}
```

以下是输出:

```java
Subscribing
Getting 1
subscriber#1 is printing 1
Getting 2
subscriber#1 is printing 2
Getting 1
subscriber#2 is printing 1
Getting 2
subscriber#2 is printing 2
Clear resources
Clear resources
```

正如我们所见，**获取每个元素以及清除资源默认执行两次**——每个`Subscriber`一次。这不是我们想要的。`ConnectableObservable`类有助于解决这个问题。

## 3.`ConnectableObservable`

[`ConnectableObservable`](https://web.archive.org/web/20221205210906/http://reactivex.io/RxJava/javadoc/rx/observables/ConnectableObservable.html) 类允许与多个订阅者共享订阅，并且不需要多次执行底层操作。

但首先，让我们创建一个`ConnectableObservable`。

### 3.1.`publish()`

`publish()`方法就是从一个`Observable`创建一个`ConnectableObservable`的东西:

```java
ConnectableObservable obs = Observable.create(subscriber -> {
    subscriber.onNext(gettingValue(1));
    subscriber.onNext(gettingValue(2));
    subscriber.add(Subscriptions.create(() -> {
        LOGGER.info("Clear resources");
    }));
}).publish();
```

但是现在，它什么也做不了。让它起作用的是`connect()`方法。

### 3.2.`connect()`

**直到`ConnectableObservable`的`connect()`方法没有被调用`Observable`的`onSubcribe()`回调没有被触发**，即使有一些订阅者。

让我们来演示一下:

```java
LOGGER.info("Subscribing");
obs.subscribe(i -> LOGGER.info("subscriber #1 is printing " + i));
obs.subscribe(i -> LOGGER.info("subscriber #2 is printing " + i));
Thread.sleep(1000);
LOGGER.info("Connecting");
Subscription s = obs.connect();
s.unsubscribe();
```

我们订阅，然后在连接前等待一秒钟。输出是:

```java
Subscribing
Connecting
Getting 1
subscriber #1 is printing 1
subscriber #2 is printing 1
Getting 2
subscriber #1 is printing 2
subscriber #2 is printing 2
Clear resources
```

我们可以看到:

这种延迟是有益的——有时我们需要给所有的订阅者相同的元素序列，即使其中一个比另一个订阅得早。

### 3.3.可观测量的一致视图-`subscribe()`之后的`connect()`

这个用例无法在我们之前的`Observable`上演示，因为它运行起来很冷，而且两个订阅者都获得了整个元素序列。

相反，设想一个元素的发出不依赖于订阅的时刻，例如鼠标点击时发出的事件。现在还假设第二个`Subscriber`在第一个订阅之后订阅了第二个。

在这个例子中，第一个`Subscriber`将获得所有发出的元素，而第二个`Subscriber`将只接收一些元素。

另一方面，在正确的地方使用`connect()`方法可以让两个订阅者对`Observable`序列有相同的看法。

**热点的例子`Observable`**

让我们创造一个热点`Observable`。它将在鼠标点击`JFrame`时发射元素。

每个元素都将是单击的 x 坐标:

```java
private static Observable getObservable() {
    return Observable.create(subscriber -> {
        frame.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseClicked(MouseEvent e) {
                subscriber.onNext(e.getX());
            }
        });
        subscriber.add(Subscriptions.create(() {
            LOGGER.info("Clear resources");
            for (MouseListener listener : frame.getListeners(MouseListener.class)) {
                frame.removeMouseListener(listener);
            }
        }));
    });
} 
```

**热门的默认行为`Observable`**

现在，如果我们以第二个间隔一个接一个地订阅两个`Subscriber`，运行程序并开始单击，我们将看到第一个`Subscriber`将获得更多元素:

```java
public static void defaultBehaviour() throws InterruptedException {
    Observable obs = getObservable();

    LOGGER.info("subscribing #1");
    Subscription subscription1 = obs.subscribe((i) -> 
        LOGGER.info("subscriber#1 is printing x-coordinate " + i));
    Thread.sleep(1000);
    LOGGER.info("subscribing #2");
    Subscription subscription2 = obs.subscribe((i) -> 
        LOGGER.info("subscriber#2 is printing x-coordinate " + i));
    Thread.sleep(1000);
    LOGGER.info("unsubscribe#1");
    subscription1.unsubscribe();
    Thread.sleep(1000);
    LOGGER.info("unsubscribe#2");
    subscription2.unsubscribe();
}
```

```java
subscribing #1
subscriber#1 is printing x-coordinate 280
subscriber#1 is printing x-coordinate 242
subscribing #2
subscriber#1 is printing x-coordinate 343
subscriber#2 is printing x-coordinate 343
unsubscribe#1
clearing resources
unsubscribe#2
clearing resources
```

**`connect()`后`subscribe()`**

为了使两个订阅者得到相同的序列，我们将把这个`Observable`转换成`ConnectableObservable`，并在订阅后调用`connect()`两个`Subscriber`:

```java
public static void subscribeBeforeConnect() throws InterruptedException {

    ConnectableObservable obs = getObservable().publish();

    LOGGER.info("subscribing #1");
    Subscription subscription1 = obs.subscribe(
      i -> LOGGER.info("subscriber#1 is printing x-coordinate " + i));
    Thread.sleep(1000);
    LOGGER.info("subscribing #2");
    Subscription subscription2 = obs.subscribe(
      i ->  LOGGER.info("subscriber#2 is printing x-coordinate " + i));
    Thread.sleep(1000);
    LOGGER.info("connecting:");
    Subscription s = obs.connect();
    Thread.sleep(1000);
    LOGGER.info("unsubscribe connected");
    s.unsubscribe();
}
```

现在他们会得到相同的序列:

```java
subscribing #1
subscribing #2
connecting:
subscriber#1 is printing x-coordinate 317
subscriber#2 is printing x-coordinate 317
subscriber#1 is printing x-coordinate 364
subscriber#2 is printing x-coordinate 364
unsubscribe connected
clearing resources
```

因此，关键是等待所有订户都准备好的那一刻，然后调用 `connect()`。

例如，在 Spring 应用程序中，我们可以在应用程序启动时订阅所有组件，并在`onApplicationEvent()`中调用`connect()`。

但是让我们回到我们的例子；请注意，`connect()`方法之前的所有点击都会被忽略。如果我们不想错过元素，而是相反地处理它们，我们可以将`connect()`放在代码的前面，并迫使`Observable`在没有任何`Subscriber`的情况下产生事件。

### 3.4.在`subscribe()`之前没有`Subscriber`–`connect()`的情况下强制订阅

为了证明这一点，让我们纠正我们的例子:

```java
public static void connectBeforeSubscribe() throws InterruptedException {
    ConnectableObservable obs = getObservable()
      .doOnNext(x -> LOGGER.info("saving " + x)).publish();
    LOGGER.info("connecting:");
    Subscription s = obs.connect();
    Thread.sleep(1000);
    LOGGER.info("subscribing #1");
    obs.subscribe((i) -> LOGGER.info("subscriber#1 is printing x-coordinate " + i));
    Thread.sleep(1000);
    LOGGER.info("subscribing #2");
    obs.subscribe((i) -> LOGGER.info("subscriber#2 is printing x-coordinate " + i));
    Thread.sleep(1000);
    s.unsubscribe();
}
```

步骤相对简单:

*   首先，我们连接
*   然后我们等待一秒钟，订阅第一个`Subscriber`
*   最后，我们再等一秒，订阅第二个`Subscriber`

注意，我们已经添加了`doOnNext()`操作符。在这里，我们可以在数据库中存储元素，但在我们的代码中，我们只打印“保存…”。

如果我们启动代码并开始单击，我们将看到元素在`connect()`调用后立即被发出并处理:

```java
connecting:
saving 306
saving 248
subscribing #1
saving 377
subscriber#1 is printing x-coordinate 377
saving 295
subscriber#1 is printing x-coordinate 295
saving 206
subscriber#1 is printing x-coordinate 206
subscribing #2
saving 347
subscriber#1 is printing x-coordinate 347
subscriber#2 is printing x-coordinate 347
clearing resources
```

如果没有订阅者，元素仍然会被处理。

**因此`connect()`方法开始发出和处理元素，而不管是否有人订阅**，就好像有一个具有空动作的人工`Subscriber`消耗了元素。

如果一些真实的`Subscriber`订阅了，这个人工的中介只是向它们传播元素。

要取消订阅人工`Subscriber`我们执行:

```java
s.unsubscribe();
```

其中:

```java
Subscription s = obs.connect();
```

### 3.5.`autoConnect()`

**这个方法意味着 `connect()`不是在订阅之前或之后调用，而是在第一个`Subscriber`订阅**时自动调用。

使用这个方法，我们不能自己调用`connect()`，因为返回的对象是一个普通的`Observable`，它没有这个方法，但是使用了一个底层的`ConnectableObservable`:

```java
public static void autoConnectAndSubscribe() throws InterruptedException {
    Observable obs = getObservable()
    .doOnNext(x -> LOGGER.info("saving " + x)).publish().autoConnect();

    LOGGER.info("autoconnect()");
    Thread.sleep(1000);
    LOGGER.info("subscribing #1");
    Subscription s1 = obs.subscribe((i) -> 
        LOGGER.info("subscriber#1 is printing x-coordinate " + i));
    Thread.sleep(1000);
    LOGGER.info("subscribing #2");
    Subscription s2 = obs.subscribe((i) -> 
        LOGGER.info("subscriber#2 is printing x-coordinate " + i));

    Thread.sleep(1000);
    LOGGER.info("unsubscribe 1");
    s1.unsubscribe();
    Thread.sleep(1000);
    LOGGER.info("unsubscribe 2");
    s2.unsubscribe();
}
```

注意，我们不能同时退订人工`Subscriber`。我们可以取消订阅所有真实的`Subscribers`，但是虚拟的`Subscriber`仍然会处理事件。

为了理解这一点，让我们看看在最后一个订户退订之后发生了什么:

```java
subscribing #1
saving 296
subscriber#1 is printing x-coordinate 296
saving 329
subscriber#1 is printing x-coordinate 329
subscribing #2
saving 226
subscriber#1 is printing x-coordinate 226
subscriber#2 is printing x-coordinate 226
unsubscribe 1
saving 268
subscriber#2 is printing x-coordinate 268
saving 234
subscriber#2 is printing x-coordinate 234
unsubscribe 2
saving 278
saving 268
```

正如我们所看到的，清除资源并没有发生，在第二次取消订阅后，使用`doOnNext()`保存元素继续进行。这意味着人工`Subscriber`不退订而是继续消费元素。

### 3.6.`refCount()`

**`refCount()`与`autoConnect()`的相似之处在于，一旦第一个`Subscriber`订阅，连接也会自动发生。**

与`autoconnect()`不同，当最后一个`Subscriber`取消订阅时，断开连接也会自动发生:

```java
public static void refCountAndSubscribe() throws InterruptedException {
    Observable obs = getObservable()
      .doOnNext(x -> LOGGER.info("saving " + x)).publish().refCount();

    LOGGER.info("refcount()");
    Thread.sleep(1000);
    LOGGER.info("subscribing #1");
    Subscription subscription1 = obs.subscribe(
      i -> LOGGER.info("subscriber#1 is printing x-coordinate " + i));
    Thread.sleep(1000);
    LOGGER.info("subscribing #2");
    Subscription subscription2 = obs.subscribe(
      i -> LOGGER.info("subscriber#2 is printing x-coordinate " + i));

    Thread.sleep(1000);
    LOGGER.info("unsubscribe#1");
    subscription1.unsubscribe();
    Thread.sleep(1000);
    LOGGER.info("unsubscribe#2");
    subscription2.unsubscribe();
}
```

```java
refcount()
subscribing #1
saving 265
subscriber#1 is printing x-coordinate 265
saving 338
subscriber#1 is printing x-coordinate 338
subscribing #2
saving 203
subscriber#1 is printing x-coordinate 203
subscriber#2 is printing x-coordinate 203
unsubscribe#1
saving 294
subscriber#2 is printing x-coordinate 294
unsubscribe#2
clearing resources
```

## 4.结论

`ConnectableObservable`类有助于轻松处理多个订户。

它的方法看起来很相似，但是由于实现上的微妙之处，甚至方法的顺序也很重要，所以极大地改变了订阅者的行为。

本文中使用的所有示例的完整源代码可以在 GitHub 项目中找到。