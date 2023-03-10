# 带 RxJava 的 Ratpack

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ratpack-rxjava>

## 1.介绍

RxJava 是目前最流行的反应式编程库之一。

和 [`Ratpack`](/web/20221129014402/https://www.baeldung.com/ratpack) 是一个 Java 库的集合，用于创建基于 Netty 的精简而强大的 web 应用程序。

在本教程中，我们将讨论在 Ratpack 应用程序中加入 RxJava 来创建一个好的反应式 web 应用程序。

## 2.Maven 依赖性

现在，我们首先需要`[ratpack-core](https://web.archive.org/web/20221129014402/https://search.maven.org/search?q=g:io.ratpack%20AND%20a:ratpack-core) `和`[ratpack-rx](https://web.archive.org/web/20221129014402/https://search.maven.org/search?q=g:io.ratpack%20AND%20a:ratpack-rx) `依赖关系:

```java
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-core</artifactId>
    <version>1.6.0</version>
</dependency>
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-rx</artifactId>
    <version>1.6.0</version>
</dependency> 
```

顺便注意，`ratpack-rx`为我们导入了`rxjava` 依赖项。

## 3.初始设置

RxJava 使用其插件系统支持第三方库的集成。**所以，我们可以在 RxJava 的执行模型中融入不同的执行策略。**

Ratpack 通过`RxRatpack`插入这个执行模型，我们在启动时初始化它:

```java
RxRatpack.initialise(); 
```

现在，需要注意的是，每次 JVM 运行只需要调用**该方法一次。**

结果是**我们将能够将 RxJava 的`Observable`映射到 RxRatpack 的`Promise`类型**，反之亦然。

## 4.`Observable` s 到`Promise` s

我们可以将 RxJava 中的`Observable`转换成 Ratpack `Promise.`

**然而，有一点不匹配。**看，`Promise `发出单个值，但是`Observable `可以发出它们的流。

RxRatpack 通过提供两种不同的方法来处理这个问题:`promiseSingle()`和`promise().`

因此，假设我们有一个名为`MovieService` 的服务，它在`getMovie(). `上发出一个承诺，我们将使用`promiseSingle()`，因为我们知道它只会发出一次:

```java
Handler movieHandler = (ctx) -> {
    MovieService movieSvc = ctx.get(MovieService.class);
    Observable<Movie> movieObs = movieSvc.getMovie();
    RxRatpack.promiseSingle(movieObs)
      .then(movie -> ctx.render(Jackson.json(movie)));
};
```

另一方面，如果`getMovies()`可以返回电影结果流，我们将使用`promise()`:

```java
Handler moviesHandler = (ctx) -> {
    MovieService movieSvc = ctx.get(MovieService.class);
    Observable<Movie> movieObs = movieSvc.getMovies();
    RxRatpack.promise(movieObs)
      .then(movie -> ctx.render(Jackson.json(movie)));
};
```

然后，我们可以像平常一样将这些处理程序添加到 Ratpack 服务器中:

```java
RatpackServer.start(def -> def.registryOf(rSpec -> rSpec.add(MovieService.class, new MovieServiceImpl()))
  .handlers(chain -> chain
    .get("movie", movieHandler)
    .get("movies", moviesHandler)));
```

## 5.`Promise` s 到`Observable` s

相反，我们可以将 Ratpack 中的`Promise`类型映射回 RxJava `Observable`。****

**RxRatpack 又有两个方法:`observe()`和`observeEach().`**

在这种情况下，我们将想象我们有一个返回`Promise` s 而不是`Observable` s 的电影服务。

没有

```java
Handler moviePromiseHandler = ctx -> {
    MoviePromiseService promiseSvc = ctx.get(MoviePromiseService.class);
    Promise<Movie> moviePromise = promiseSvc.getMovie();
    RxRatpack.observe(moviePromise)
      .subscribe(movie -> ctx.render(Jackson.json(movie)));
};
```

当我们得到一个列表时，比如用`getMovies()`，我们会用`observeEach()`:

```java
Handler moviesPromiseHandler = ctx -> {
    MoviePromiseService promiseSvc = ctx.get(MoviePromiseService.class);
    Promise<List<Movie>> moviePromises = promiseSvc.getMovies();
    RxRatpack.observeEach(moviePromises)
        .toList()
        .subscribe(movie -> ctx.render(Jackson.json(movie)));
};
```

然后，我们可以像预期的那样添加处理程序:

```java
RatpackServer.start(def -> def.registryOf(regSpec -> regSpec
  .add(MoviePromiseService.class, new MoviePromiseServiceImpl()))
    .handlers(chain -> chain
      .get("movie", moviePromiseHandler)
      .get("movies", moviesPromiseHandler)));
```

## 6.并行处理

**RxRatpack 借助`fork()`和`forkEach()`方法支持并行性。**

它遵循一种我们已经见过的模式。

`fork()`采用单个`Observable`并且**将其并行化到不同的计算线程**上执行。然后，它会自动将数据绑定回最初的执行。

另一方面，`forkEach()`对由`Observable`的值流发出的每个元素做同样的事情。

让我们想象一下，我们想利用我们的电影名称，这是一个昂贵的操作。

简单地说，我们可以使用`forkEach()`将每个线程的执行卸载到一个线程池中:

```java
Observable<Movie> movieObs = movieSvc.getMovies();
Observable<String> upperCasedNames = movieObs.compose(RxRatpack::forkEach)
  .map(movie -> movie.getName().toUpperCase())
  .serialize();
```

## 7.隐式错误处理

最后，隐式错误处理是 RxJava 集成的关键特性之一。

默认情况下，RxJava 可观察序列会将任何异常转发给执行上下文异常处理程序。出于这个原因，错误处理程序不需要在可观察的序列中定义。

因此，**我们可以配置 Ratpack 来处理 RxJava 引发的这些错误。**

例如，我们希望在 HTTP 响应中输出每个错误。

注意，我们通过`Observable`抛出的异常被我们的`ServerErrorHandler`捕获并处理:

```java
RatpackServer.start(def -> def.registryOf(regSpec -> regSpec
  .add(ServerErrorHandler.class, (ctx, throwable) -> {
        ctx.render("Error caught by handler : " + throwable.getMessage());
    }))
  .handlers(chain -> chain
    .get("error", ctx -> {
        Observable.<String> error(new Exception("Error from observable")).subscribe(s -> {});
    })));
```

**注意，任何订阅者级别的错误处理都是优先的。**如果我们的`Observable` 想要做自己的错误处理，它可以，但是因为它不这样做，异常渗透到 Ratpack。

## 8.结论

在本文中，我们讨论了如何用 Ratpack 配置 RxJava。

我们研究了 RxJava 中的`Observables`到 Ratpack 中的`Promise`类型的转换，反之亦然。我们还研究了集成支持的并行性和隐式错误处理特性。

本文使用的所有代码示例都可以在 Github 上找到[。](https://web.archive.org/web/20221129014402/https://github.com/eugenp/tutorials/tree/master/web-modules/ratpack)