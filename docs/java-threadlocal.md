# Java 中的 ThreadLocal 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-threadlocal>

## 1。概述

在本教程中，我们将关注来自`java.lang` 包的`[ThreadLocal](https://web.archive.org/web/20220625075228/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ThreadLocal.html)`构造。这使我们能够为当前线程单独存储数据，并简单地将其包装在一个特殊类型的对象中。

## 2。`ThreadLocal` API

`TheadLocal` 构造允许我们存储数据，这些数据只能由特定线程的**访问**。****

假设我们想要一个与特定线程绑定的`Integer`值:

```java
ThreadLocal<Integer> threadLocalValue = new ThreadLocal<>();
```

接下来，当我们想从一个线程中使用这个值时，我们只需要调用一个`get()`或`set()`方法。简单地说，我们可以想象`ThreadLocal`用线程作为键将数据存储在地图中。

因此，当我们在`threadLocalValue`上调用`get()`方法时，我们将获得请求线程的`Integer`值:

```java
threadLocalValue.set(1);
Integer result = threadLocalValue.get();
```

我们可以通过使用`withInitial()`静态方法并向其传递一个供应商来构造`ThreadLocal`的实例:

```java
ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 1);
```

要从`ThreadLocal`中删除值，我们可以调用`remove()`方法:

```java
threadLocal.remove();
```

为了了解如何正确使用`ThreadLocal`，我们将首先看一个不使用`ThreadLocal`的例子，然后我们将重写我们的例子来利用这个结构。

## 3。将用户数据存储在地图中

让我们考虑一个程序，它需要为每个给定的用户 id 存储特定于用户的`Context`数据:

```java
public class Context {
    private String userName;

    public Context(String userName) {
        this.userName = userName;
    }
}
```

我们希望每个用户 id 有一个线程。我们将创建一个实现`Runnable`接口的`SharedMapWithUserContext`类。`run()`方法中的实现通过`UserRepository`类调用一些数据库，该类为给定的`userId`返回一个`Context`对象。

接下来，我们将上下文存储在由`userId`键控的`ConcurentHashMap`中:

```java
public class SharedMapWithUserContext implements Runnable {

    public static Map<Integer, Context> userContextPerUserId
      = new ConcurrentHashMap<>();
    private Integer userId;
    private UserRepository userRepository = new UserRepository();

    @Override
    public void run() {
        String userName = userRepository.getUserNameForUserId(userId);
        userContextPerUserId.put(userId, new Context(userName));
    }

    // standard constructor
}
```

通过为两个不同的`userIds,`创建并启动两个线程，并断言我们在`userContextPerUserId`映射中有两个条目，我们可以很容易地测试我们的代码:

```java
SharedMapWithUserContext firstUser = new SharedMapWithUserContext(1);
SharedMapWithUserContext secondUser = new SharedMapWithUserContext(2);
new Thread(firstUser).start();
new Thread(secondUser).start();

assertEquals(SharedMapWithUserContext.userContextPerUserId.size(), 2);
```

## 4。将用户数据存储在`ThreadLocal`

我们可以重写我们的示例，使用一个`ThreadLocal`来存储用户`Context` 实例。每个线程都有自己的`ThreadLocal`实例。

当使用`ThreadLocal`时，我们需要非常小心，因为每个`ThreadLocal`实例都与一个特定的线程相关联。在我们的例子中，我们为每个特定的`userId`都有一个专用的线程，这个线程是我们创建的，所以我们对它有完全的控制权。

`run()`方法将获取用户上下文，并使用`set()`方法将其存储到`ThreadLocal`变量中:

```java
public class ThreadLocalWithUserContext implements Runnable {

    private static ThreadLocal<Context> userContext 
      = new ThreadLocal<>();
    private Integer userId;
    private UserRepository userRepository = new UserRepository();

    @Override
    public void run() {
        String userName = userRepository.getUserNameForUserId(userId);
        userContext.set(new Context(userName));
        System.out.println("thread context for given userId: " 
          + userId + " is: " + userContext.get());
    }

    // standard constructor
}
```

我们可以通过启动两个线程来测试它，这两个线程将针对给定的`userId`执行动作:

```java
ThreadLocalWithUserContext firstUser 
  = new ThreadLocalWithUserContext(1);
ThreadLocalWithUserContext secondUser 
  = new ThreadLocalWithUserContext(2);
new Thread(firstUser).start();
new Thread(secondUser).start();
```

运行这段代码后，我们将在标准输出中看到 `ThreadLocal`是为每个给定线程设置的:

```java
thread context for given userId: 1 is: Context{userNameSecret='18a78f8e-24d2-4abf-91d6-79eaa198123f'}
thread context for given userId: 2 is: Context{userNameSecret='e19f6a0a-253e-423e-8b2b-bca1f471ae5c'}
```

我们可以看到每个用户都有自己的`Context`。

## 5。`ThreadLocal`sī线程池

`ThreadLocal`提供一个易于使用的 API 来限制每个线程的一些值。这是在 Java 中实现[线程安全](/web/20220625075228/https://www.baeldung.com/java-thread-safety)的合理方式。然而，**当我们一起使用`ThreadLocal`** **s 和[线程池](/web/20220625075228/https://www.baeldung.com/thread-pool-java-and-guava)时，我们应该格外小心。**

为了更好地理解这个可能的警告，让我们考虑以下场景:

1.  首先，应用程序从池中借用一个线程。
2.  然后，它将一些线程限制的值存储到当前线程的`ThreadLocal`中。
3.  一旦当前执行完成，应用程序将借用的线程返回到池中。
4.  过了一会儿，应用程序借用同一个线程来处理另一个请求。
5.  **由于应用程序上次没有执行必要的清理，它可能会为新请求重新使用相同的`ThreadLocal`数据。**

这可能会在高度并发的应用程序中导致令人惊讶的后果。

解决这个问题的一个方法是，一旦我们用完了，就手动删除每个`ThreadLocal`。因为这种方法需要严格的代码审查，所以很容易出错。

### 5.1.扩展`ThreadPoolExecutor`

事实证明，**可以扩展`ThreadPoolExecutor`类，并为 [`beforeExecute()`](https://web.archive.org/web/20220625075228/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html#beforeExecute(java.lang.Thread,java.lang.Runnable)) 和 [`afterExecute()`](https://web.archive.org/web/20220625075228/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html#afterExecute(java.lang.Runnable,java.lang.Throwable)) 方法提供一个定制的钩子实现。**线程池将在使用借用的线程运行任何东西之前调用`beforeExecute()`方法。另一方面，它会在执行我们的逻辑后调用`afterExecute()`方法。

因此，我们可以扩展`ThreadPoolExecutor`类并删除`afterExecute()`方法中的`ThreadLocal`数据:

```java
public class ThreadLocalAwareThreadPool extends ThreadPoolExecutor {

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        // Call remove on each ThreadLocal
    }
}
```

如果我们将请求提交给这个`[ExecutorService](https://web.archive.org/web/20220625075228/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ExecutorService.html)`的实现，那么我们可以确定使用`ThreadLocal`和线程池不会给我们的应用程序带来安全隐患。

## 6。结论

在这篇简短的文章中，我们研究了`ThreadLocal` 构造。我们实现了使用线程间共享的`ConcurrentHashMap` 来存储与特定`userId.` 相关联的上下文的逻辑，然后我们重写了我们的示例，以利用`ThreadLocal` 来存储与特定`userId`和特定线程相关联的数据。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220625075228/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced)