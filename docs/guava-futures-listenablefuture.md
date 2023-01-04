# 番石榴的未来和 ListenableFuture

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-futures-listenablefuture>

## 1.介绍

Guava 为我们提供了比默认 Java`Future.` 更丰富的 API，让我们看看如何利用这一点。

## 2.`Future`、`ListenableFuture`和`Futures`

让我们简单看看这些不同的类是什么，以及它们之间的关系。

### 2.1.`Future`

因为`Java 5, `我们可以用`[java.util.concurrent.Future](/web/20221101111241/https://www.baeldung.com/java-future "Future") `来表示异步任务。

一个`Future`允许我们访问已经完成或者可能在未来完成的任务的结果，以及对取消它们的支持。

### 2.2.`ListenableFuture`

使用`java.util.concurrent.Future`时缺少的一个特性是添加监听器以在完成时运行的能力，这是大多数流行的异步框架提供的一个常见特性。

**Guava 通过允许我们将监听器**连接到它的`[com.google.common.util.concurrent.ListenableFuture](https://web.archive.org/web/20221101111241/https://guava.dev/releases/29.0-jre/api/docs/com/google/common/util/concurrent/ListenableFuture.html "ListenableFuture").`来解决这个问题

### 2.3.`Futures`

番石榴为我们提供了便利类 [`com.google.common.util.concurrent.Futures`](https://web.archive.org/web/20221101111241/https://guava.dev/releases/29.0-jre/api/docs/com/google/common/util/concurrent/Futures "Futures") ，让我们更容易使用它们的`ListenableFuture.`

这个类提供了与`ListenableFuture,`交互的各种方式，其中之一是对添加成功/失败回调的**支持，并允许我们通过聚合或转换来协调多个未来。**

## 3.简单用法

现在让我们看看如何以最简单的方式使用`ListenableFuture`;创建和添加回调。

### 3.1.正在创建`ListenableFuture`

**我们获得`ListenableFuture`的最简单的方法是提交一个任务给`ListeningExecutorService`** (就像我们如何使用一个普通的`ExecutorService `来获得一个普通的`Future`):

```java
ExecutorService execService = Executors.newSingleThreadExecutor();
ListeningExecutorService lExecService = MoreExecutors.listeningDecorator(execService);

ListenableFuture<Integer> asyncTask = lExecService.submit(() -> {
    TimeUnit.MILLISECONDS.sleep(500); // long running task
    return 5;
});
```

注意我们是如何使用`MoreExecutors `类将`ExecutorService`装饰成`ListeningExecutorService. `的，我们可以参考[线程池在 Guava](/web/20221101111241/https://www.baeldung.com/thread-pool-java-and-guava#Implementation "Introduction to Thread Pools in Java") 中的实现来了解更多关于`MoreExecutors`的信息。

如果我们已经有了一个返回`Future`的 API，我们需要将它转换成`ListenableFuture`，这很容易通过初始化它的具体实现`ListenableFutureTask:`来完成

```java
// old api
public FutureTask<String> fetchConfigTask(String configKey) {
    return new FutureTask<>(() -> {
        TimeUnit.MILLISECONDS.sleep(500);
        return String.format("%s.%d", configKey, new Random().nextInt(Integer.MAX_VALUE));
    });
}

// new api
public ListenableFutureTask<String> fetchConfigListenableTask(String configKey) {
    return ListenableFutureTask.create(() -> {
        TimeUnit.MILLISECONDS.sleep(500);
        return String.format("%s.%d", configKey, new Random().nextInt(Integer.MAX_VALUE));
    });
}
```

我们需要意识到，除非我们将这些任务提交给一个`Executor.` **，否则它们不会运行。直接与`ListenableFutureTask` 交互并不常见，只在极少数情况下使用**(例如:实现我们自己的`ExecutorService`)。实际用法参考番石榴的`[AbstractListeningExecutorService](https://web.archive.org/web/20221101111241/https://github.com/google/guava/blob/v18.0/guava/src/com/google/common/util/concurrent/AbstractListeningExecutorService.java "AbstractListeningExecutorService")`。

如果我们的异步任务不能使用`ListeningExecutorService`或提供的`Futures`工具方法，我们也可以使用`com.google.common.util.concurrent.SettableFuture`，我们需要手动设置未来值。对于更复杂的用法，我们还可以考虑`com.google.common.util.concurrent.AbstractFuture.`

### 3.2.添加监听器/回调

我们可以**向`ListenableFuture`添加监听器的一种方式是通过向`Futures.addCallback(),` 注册一个回调，当成功或失败发生时，为我们提供对结果或异常的访问:**

```java
Executor listeningExecutor = Executors.newSingleThreadExecutor();

ListenableFuture<Integer> asyncTask = new ListenableFutureService().succeedingTask()
Futures.addCallback(asyncTask, new FutureCallback<Integer>() {
    @Override
    public void onSuccess(Integer result) {
        // do on success
    }

    @Override
    public void onFailure(Throwable t) {
        // do on failure
    }
}, listeningExecutor);
```

我们还可以通过直接添加到`ListenableFuture.` 来**添加一个监听器。注意，这个监听器将在未来成功或异常完成时运行。另外，请注意，我们无法访问异步任务的结果:**

```java
Executor listeningExecutor = Executors.newSingleThreadExecutor();

int nextTask = 1;
Set<Integer> runningTasks = ConcurrentHashMap.newKeySet();
runningTasks.add(nextTask);

ListenableFuture<Integer> asyncTask = new ListenableFutureService().succeedingTask()
asyncTask.addListener(() -> runningTasks.remove(nextTask), listeningExecutor);
```

## 4.复杂用法

现在让我们看看如何在更复杂的场景中使用这些期货。

### 4.1.扇入

我们有时可能需要调用多个异步任务并收集它们的结果，这通常称为扇入操作。

番石榴为我们提供了两种方法。但是，我们应该根据我们的要求谨慎选择正确的方法。假设我们需要协调以下异步任务:

```java
ListenableFuture<String> task1 = service.fetchConfig("config.0");
ListenableFuture<String> task2 = service.fetchConfig("config.1");
ListenableFuture<String> task3 = service.fetchConfig("config.2");
```

**一种放大多种期货的方法是使用`Futures.allAsList()` 方法。这允许我们收集所有期货的结果，如果它们都成功，**按照提供的期货的顺序。如果这些未来中的任何一个失败了，那么整个结果就是失败的未来:

```java
ListenableFuture<List<String>> configsTask = Futures.allAsList(task1, task2, task3);
Futures.addCallback(configsTask, new FutureCallback<List<String>>() {
    @Override
    public void onSuccess(@Nullable List<String> configResults) {
        // do on all futures success
    }

    @Override
    public void onFailure(Throwable t) {
        // handle on at least one failure
    }
}, someExecutor);
```

**如果我们需要收集所有异步任务的结果，不管它们是否失败，我们可以使用`Futures.successfulAsList()`** 。这将返回一个列表，其结果将与传递到参数中的任务具有相同的顺序，并且失败的任务将有`null`被分配给它们在列表中各自的位置:

```java
ListenableFuture<List<String>> configsTask = Futures.successfulAsList(task1, task2, task3);
Futures.addCallback(configsTask, new FutureCallback<List<String>>() {
    @Override
    public void onSuccess(@Nullable List<String> configResults) {
        // handle results. If task2 failed, then configResults.get(1) == null
    }

    @Override
    public void onFailure(Throwable t) {
        // handle failure
    }
}, listeningExecutor);
```

在上面的用法中我们要小心的是，**如果未来的任务正常的成功返回`null`，它将与失败的任务无法区分(它也将结果设置为`null`)。**

### 4.2.带组合器的扇入

如果我们需要协调返回不同结果的多个未来，上述解决方案可能不够。在这种情况下，我们可以使用扇入操作的组合器变体来协调这种未来的混合。

类似于简单的扇入操作，**番石榴为我们提供了两种变体；一个在所有任务成功完成时成功，另一个在一些任务失败时成功，分别使用`Futures.whenAllSucceed()`和`Futures.whenAllComplete()`方法。**

让我们看看如何使用`Futures.whenAllSucceed()` 来组合来自多个未来的不同结果类型:

```java
ListenableFuture<Integer> cartIdTask = service.getCartId();
ListenableFuture<String> customerNameTask = service.getCustomerName();
ListenableFuture<List<String>> cartItemsTask = service.getCartItems();

ListenableFuture<CartInfo> cartInfoTask = Futures.whenAllSucceed(cartIdTask, customerNameTask, cartItemsTask)
    .call(() -> {
        int cartId = Futures.getDone(cartIdTask);
        String customerName = Futures.getDone(customerNameTask);
        List<String> cartItems = Futures.getDone(cartItemsTask);
        return new CartInfo(cartId, customerName, cartItems);
    }, someExecutor);

Futures.addCallback(cartInfoTask, new FutureCallback<CartInfo>() {
    @Override
    public void onSuccess(@Nullable CartInfo result) {
        //handle on all success and combination success
    }

    @Override
    public void onFailure(Throwable t) {
        //handle on either task fail or combination failed
    }
}, listeningExecService);
```

如果我们需要允许一些任务失败，我们可以使用`Futures.whenAllComplete()`。虽然语义与上面的大部分相似，但我们应该意识到，当`Futures.getDone() `被调用时，失败的期货将抛出一个`ExecutionException`。

### 4.3.转换

有时候我们需要转换一个曾经成功的未来的结果。番石榴为我们提供了两种方法来实现`Futures.transform()`和`Futures.lazyTransform()`。

让我们看看如何使用**来转换未来的结果。这个只要变换计算量不大就可以用:**

```java
ListenableFuture<List<String>> cartItemsTask = service.getCartItems();

Function<List<String>, Integer> itemCountFunc = cartItems -> {
    assertNotNull(cartItems);
    return cartItems.size();
};

ListenableFuture<Integer> itemCountTask = Futures.transform(cartItemsTask, itemCountFunc, listenExecService);
```

**我们也可以使用`Futures.lazyTransform()`** 将转换函数应用到`java.util.concurrent.Future.`中。我们需要记住，这个选项并不返回一个`ListenableFuture`，而是一个普通的`java.util.concurrent.Future`，并且转换函数在每次`get()`被调用时都适用。

### 4.4.连锁期货

我们可能会遇到我们的期货需要调用其他期货的情况。在这种情况下，番石榴为我们提供了`async()`变体来安全地将这些期货一个接一个地执行。

让我们看看如何**使用`Futures.submitAsync()` 从提交的`Callable `中调用一个未来:**

```java
AsyncCallable<String> asyncConfigTask = () -> {
    ListenableFuture<String> configTask = service.fetchConfig("config.a");
    TimeUnit.MILLISECONDS.sleep(500); //some long running task
    return configTask;
};

ListenableFuture<String> configTask = Futures.submitAsync(asyncConfigTask, executor);
```

如果我们想要真正的链接，一个未来的结果被输入到另一个未来的计算中，我们可以使用`Futures.transformAsync()` :

```java
ListenableFuture<String> usernameTask = service.generateUsername("john");
AsyncFunction<String, String> passwordFunc = username -> {
    ListenableFuture<String> generatePasswordTask = service.generatePassword(username);
    TimeUnit.MILLISECONDS.sleep(500); // some long running task
    return generatePasswordTask;
};

ListenableFuture<String> passwordTask = Futures.transformAsync(usernameTask, passwordFunc, executor);
```

Guava 还为我们提供了`Futures.scheduleAsync()`和`Futures.catchingAsync()` 来分别提交调度任务和提供错误恢复的回退任务。虽然它们适用于不同的场景，但我们不会讨论它们，因为它们与其他`async()`呼叫相似。

## 5.用法注意事项

现在，让我们来研究一些在期货交易中可能遇到的常见陷阱，以及如何避免它们。

### 5.1.工作执行者与倾听执行者

使用番石榴期货时，理解工作执行者和倾听执行者的区别是很重要的。例如，假设我们有一个获取配置的异步任务:

```java
public ListenableFuture<String> fetchConfig(String configKey) {
    return lExecService.submit(() -> {
        TimeUnit.MILLISECONDS.sleep(500);
        return String.format("%s.%d", configKey, new Random().nextInt(Integer.MAX_VALUE));
    });
}
```

我们还可以说，我们想给上述未来附加一个监听器:

```java
ListenableFuture<String> configsTask = service.fetchConfig("config.0");
Futures.addCallback(configsTask, someListener, listeningExecutor);
```

注意这里的`lExecService `是运行异步任务的执行器，而`listeningExecutor` 是调用监听器的执行器。

如上所述，**我们应该总是考虑分离这两个执行器，以避免我们的侦听器和工作器竞争相同的线程池资源。**共享同一个执行器可能会导致我们的重载任务使监听器执行饥饿。或者一个写得很糟糕的重量级听众最终阻碍了我们重要的重型任务。

### 5.2.小心使用`directExecutor()`

虽然我们可以在单元测试中使用`MoreExecutors.directExecutor()`和`MoreExecutors.newDirectExecutorService() `来更容易地处理异步执行，但是在产品代码中使用它们时我们应该小心。

当我们从上述方法中获得执行器时，我们提交给它的任何任务，无论是重量级的还是监听级的，都将在当前线程上执行。如果当前执行上下文需要高吞吐量，这可能是危险的。

比如使用一个`directExecutor `，在 UI 线程中向它提交一个重量级任务，会自动阻塞我们的 UI 线程。

我们还可能面临这样一种情况，我们的侦听器最终拖慢了我们所有其他侦听器(甚至是那些与`directExecutor`无关的侦听器)。这是因为 Guava 在各自的`Executors, `中执行`while`循环中的所有监听器，但是`directExecutor `将导致监听器在与`while`循环相同的线程中运行。

### 5.3.嵌套期货不好

当使用链式期货时，我们应该小心不要从另一个期货内部调用一个期货，以免创建嵌套期货:

```java
public ListenableFuture<String> generatePassword(String username) {
    return lExecService.submit(() -> {
        TimeUnit.MILLISECONDS.sleep(500);
        return username + "123";
    });
}

String firstName = "john";
ListenableFuture<ListenableFuture<String>> badTask = lExecService.submit(() -> {
    final String username = firstName.replaceAll("[^a-zA-Z]+", "")
        .concat("@service.com");
    return generatePassword(username);
});
```

**如果我们曾经看到有`ListenableFuture<ListenableFuture<V>>,` 的代码，那么我们应该知道这是一个写得很糟糕的未来**，因为外部未来的取消和完成可能会加速，而取消可能不会传播到内在未来。

如果我们看到上面的场景，我们应该总是使用`Futures.async()` 变体以一种连接的方式安全地解开这些连锁的未来。

### 5.4.小心使用`JdkFutureAdapters.listenInPoolThread()`

Guava 建议我们利用它的`ListenableFuture `的最好方法是将所有使用`Future`的代码转换成`ListenableFuture. `

如果这种转换在某些场景中不可行， **Guava 为我们提供了使用`JdkFutureAdapters.listenInPoolThread()`覆盖来完成这一转换的适配器。**虽然这似乎很有帮助，**番石榴警告我们，这些是重量级适配器，应该尽可能避免使用。**

## 6.结论

在本文中，我们已经看到了如何使用 Guava 的`ListenableFuture`来丰富我们对期货的使用，以及如何使用`Futures ` API 来简化这些期货。

我们也看到了一些我们在使用这些期货和所提供的执行者时可能会犯的常见错误。

和往常一样，GitHub 上的[提供了我们示例的完整源代码。](https://web.archive.org/web/20221101111241/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-concurrency)