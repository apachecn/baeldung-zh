# JDeferred 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdeferred>

## 1。概述

[`JDeferred`](https://web.archive.org/web/20221205135244/http://jdeferred.org/) 是一个小型的`Java`库(也支持`Groovy`)，用于实现异步拓扑，无需编写样板代码。这个框架的灵感来自于 [`Jquery's Promise/Ajax`](https://web.archive.org/web/20221205135244/https://api.jquery.com/jquery.ajax/) 特性和 [`Android's Deferred Object`](https://web.archive.org/web/20221205135244/https://github.com/CodeAndMagic/android-deferred-object) 模式。

在本教程中，我们将展示如何使用`JDeferred`及其不同的实用程序。

## 2。Maven 依赖关系

我们可以在任何应用程序中使用`JDeferred`,方法是将下面的依赖项添加到我们的`pom.xml:`中

```java
<dependency>
    <groupId>org.jdeferred</groupId>
    <artifactId>jdeferred-core</artifactId>
    <version>1.2.6</version>
</dependency>
```

我们可以在[中央 Maven 资源库](https://web.archive.org/web/20221205135244/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.jdeferred%22%20AND%20a%3A%22jdeferred-core%22)中查看最新版本的`JDeferred`项目。

## 3。承诺

让我们看一个简单的用例，调用一个容易出错的同步`REST` `API`调用，并根据 API 返回的数据执行一些任务。

在简单的 JQuery 中，上述场景可以通过以下方式解决:

```java
$.ajax("/GetEmployees")
    .done(
        function() {
            alert( "success" );
        }
     )
    .fail(
        function() {
            alert( "error" );
        }
     )
    .always(
        function() {
            alert( "complete" );
        }
    );
```

类似地，`JDeferred`带有`[Promise](https://web.archive.org/web/20221205135244/https://github.com/jdeferred/jdeferred/blob/master/subprojects/jdeferred-core/src/main/java/org/jdeferred2/Promise.java)`和 [`Deferred`](https://web.archive.org/web/20221205135244/https://github.com/jdeferred/jdeferred/blob/master/subprojects/jdeferred-core/src/main/java/org/jdeferred2/Deferred.java) 接口，这些接口在相应的对象上注册一个线程独立的钩子，该钩子根据对象状态触发不同的可定制动作。

这里，`Deferred`充当触发器，`Promise`充当观察者。

我们可以很容易地创建这种类型的异步工作流:

```java
Deferred<String, String, String> deferred
  = new DeferredObject<>();
Promise<String, String, String> promise = deferred.promise();

promise.done(result -> System.out.println("Job done"))
  .fail(rejection -> System.out.println("Job fail"))
  .progress(progress -> System.out.println("Job is in progress"))
  .always((state, result, rejection) -> 
    System.out.println("Job execution started"));

deferred.resolve("msg");
deferred.notify("notice");
deferred.reject("oops");
```

这里，每种方法都有不同的语义:

*   `done()`–仅当延迟对象上的未决操作成功完成时触发
*   `fail()`–在对延迟对象执行未决操作时，当出现某些异常时触发
*   `progress()`–延迟对象上的挂起操作开始执行时触发
*   `always()`–不管延迟对象的状态如何都会触发

默认情况下，延期对象的状态可以是`PENDING/REJECTED/RESOLVED`。我们可以使用 **`deferred.state()`** 的方法来检查状态。

这里要注意的一点是，**一旦一个延迟对象的状态被更改为`RESOLVED,`，我们就不能对该对象执行`reject`操作。**

同样，一旦对象的状态被更改为`REJECTED,`，我们就不能对该对象执行`resolve`或`notify`操作。任何违反都会导致`IllegalStateExeption`。

## 4。过滤器

在检索最终结果之前，我们可以用 [`DoneFilter`](https://web.archive.org/web/20221205135244/https://github.com/jdeferred/jdeferred/blob/master/subprojects/jdeferred-core/src/main/java/org/jdeferred2/DoneFilter.java) 对延期对象进行过滤。

一旦过滤完成，我们将得到线程安全的延迟对象:

```java
private static String modifiedMsg;

static String filter(String msg) {
    Deferred<String, ?, ?> d = new DeferredObject<>();
    Promise<String, ?, ?> p = d.promise();
    Promise<String, ?, ?> filtered = p.then((result) > {
        modifiedMsg = "Hello "  result;
    });

    filtered.done(r > System.out.println("filtering done"));

    d.resolve(msg);
    return modifiedMsg;
}
```

## 5。管道

与过滤器类似，`JDeferred`提供了 [`DonePipe`](https://web.archive.org/web/20221205135244/https://github.com/jdeferred/jdeferred/blob/master/subprojects/jdeferred-core/src/main/java/org/jdeferred2/DonePipe.java) 接口，以便在解决了延迟的对象挂起操作后执行复杂的过滤后操作。

```java
public enum Result { 
    SUCCESS, FAILURE 
}; 

private static Result status; 

public static Result validate(int num) { 
    Deferred<Integer, ?, ?> d = new DeferredObject<>(); 
    Promise<Integer, ?, ?> p = d.promise(); 

    p.then((DonePipe<Integer, Integer, Exception, Void>) result > {
        public Deferred<Integer, Exception, Void> pipeDone(Integer result) {
            if (result < 90) {
                return new DeferredObject<Integer, Exception, Void>()
                  .resolve(result);
            } else {
                return new DeferredObject<Integer, Exception, Void>()
                  .reject(new Exception("Unacceptable value"));
            }
    }).done(r > status = Result.SUCCESS )
      .fail(r > status = Result.FAILURE );

    d.resolve(num);
    return status;
}
```

这里，基于实际结果的值，我们抛出了一个异常来拒绝结果。

## 6。延期管理器

在实时场景中，我们需要处理由多个承诺观察到的多个延迟对象。在这种情况下，很难分别管理多个承诺。

这就是为什么`JDeferred`带有 [`DeferredManager`](https://web.archive.org/web/20221205135244/https://github.com/jdeferred/jdeferred/blob/master/subprojects/jdeferred-core/src/main/java/org/jdeferred2/DeferredManager.java) 接口，为所有的承诺创建一个公共的观察者。因此，使用这个共同的观察者，我们可以为所有的承诺创建共同的行动:

```java
Deferred<String, String, String> deferred = new DeferredObject<>();
DeferredManager dm = new DefaultDeferredManager();
Promise<String, String, String> p1 = deferred.promise(), 
  p2 = deferred.promise(), 
  p3 = deferred.promise();
dm.when(p1, p2, p3)
  .done(result -> ... )
  .fail(result -> ... );
deferred.resolve("Hello Baeldung");
```

我们也可以将带有自定义线程池的 [`ExecutorService`](https://web.archive.org/web/20221205135244/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ExecutorService.html) 分配给`DeferredManager`:

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
DeferredManager dm = new DefaultDeferredManager(executor);
```

事实上，我们完全可以忽略`Promise`的使用，可以直接定义 [`Callable`](https://web.archive.org/web/20221205135244/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Callable.html) 接口来完成任务:

```java
DeferredManager dm = new DefaultDeferredManager();
dm.when(() -> {
    // return something and raise an exception to interrupt the task
}).done(result -> ... )
  .fail(e -> ... );
```

## 7 .**。线程安全动作**

虽然，大多数时候我们需要处理异步工作流，有些时候我们需要等待所有并行任务的结果。

在这种情况下，我们只能使用`Object`的`wait()`方法来**等待所有延迟的任务完成**:

```java
DeferredManager dm = new DefaultDeferredManager();
Deferred<String, String, String> deferred = new DeferredObject<>();
Promise<String, String, String> p1 = deferred.promise();
Promise<String, String, String> p = dm
  .when(p1)
  .done(result -> ... )
  .fail(result -> ... );

synchronized (p) {
    while (p.isPending()) {
        try {
            p.wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

deferred.resolve("Hello Baeldung");
```

或者，我们可以使用`Promise`接口的`waitSafely()`方法来实现同样的功能。

```java
try {
    p.waitSafely();
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

尽管上述两种方法执行的内容几乎相同，但使用第二种方法总是可取的，因为第二个过程不需要同步。

## 8。安卓集成

`JDeferred`可以使用 Android Maven 插件轻松与 Android 应用程序集成。

对于 APKLIB 构建，我们需要在`pom.xml`中添加以下依赖项:

```java
<dependency>
    <groupId>org.jdeferred</groupId>
    <artifactId>jdeferred-android</artifactId>
    <version>1.2.6</version>
    <type>apklib</type>
</dependency>
```

对于`AAR`构建，我们需要在`pom.xml`中添加以下依赖项:

```java
<dependency>
    <groupId>org.jdeferred</groupId>
    <artifactId>jdeferred-android-aar</artifactId>
    <version>1.2.6</version>
    <type>aar</type>
</dependency>
```

## 9。结论

在本教程中，我们探索了关于`JDeferred`，以及它的不同实用程序。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221205135244/https://github.com/eugenp/tutorials/tree/master/libraries-4)