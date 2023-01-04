# Vertx 和 RxJava 集成示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vertx-rx-java>

## 1。概述

[`RxJava`](https://web.archive.org/web/20221128112217/https://github.com/ReactiveX/RxJava) 是一个用于创建异步和基于事件的程序的流行库，它从`[Reactive Extensions](https://web.archive.org/web/20221128112217/http://reactivex.io/)`计划提出的主要思想中获得灵感。

`Eclipse`旗下的项目`[Vert.x](https://web.archive.org/web/20221128112217/http://vertx.io/)`提供了几个从头开始设计的组件，以充分利用反应范式。

一起使用，它们可以证明是任何需要反应的`Java`程序的有效基础。

在本文中，我们将加载一个包含城市名称列表的文件，并打印出每个城市从日出到日落的一天有多长。

我们将使用公开发布的数据[www.metaweather.com](https://web.archive.org/web/20221128112217/https://www.metaweather.com/api/)`REST API`——来计算白昼的长度，而`RxJava`和`Vert.x`则以一种纯粹被动的方式来计算。

## 2。Maven 依赖关系

先来导入`vertx-rx-java2`:

```java
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-rx-java2</artifactId>
    <version>3.5.0-Beta1</version>
</dependency> 
```

在撰写本文时，`Vert.x`和更新的`RxJava 2`之间的集成仅作为测试版可用，然而，对于我们正在构建的程序来说，它足够稳定。

注意, `io.vertx:vertx-rx-java2`依赖于`io.reactivex.rxjava2:rxjava`,所以不需要显式导入任何与`RxJava`相关的包。

与`RxJava`整合的最新版本`Vert.x`可以在 [Maven Central](https://web.archive.org/web/20221128112217/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22vertx-rx-java2%22) 上找到。

## 3。设置

和每个使用`Vert.x,`的应用程序一样，我们将开始创建`vertx`对象，这是所有`Vert.x`特性的主要入口点:

```java
Vertx vertx = io.vertx.reactivex.core.Vertx.vertx();
```

`vertx-rx-java2`库提供了两个类:`io.vertx.core.Vertx`和`io.vertx.reactivex.core.Vertx`。虽然第一个是唯一基于`Vert.x`的应用程序的通常入口点，但后者是我们用来与`RxJava`集成的入口点。

我们继续定义稍后会用到的对象:

```java
FileSystem fileSystem = vertx.fileSystem();
HttpClient httpClient = vertx.createHttpClient();
```

`Vert.x`的`FileSystem`以被动的方式访问文件系统，而`Vert.x`的`HttpClient`为`HTTP`做同样的事情。

## 4。反应链

在反应式上下文中，很容易将几个更简单的反应式操作符连接起来，以获得有意义的计算。

让我们为我们的例子这样做:

```java
fileSystem
  .rxReadFile("cities.txt").toFlowable()
  .flatMap(buffer -> Flowable.fromArray(buffer.toString().split("\\r?\\n")))
  .flatMap(city -> searchByCityName(httpClient, city))
  .flatMap(HttpClientResponse::toFlowable)
  .map(extractingWoeid())
  .flatMap(cityId -> getDataByPlaceId(httpClient, cityId))
  .flatMap(toBufferFlowable())
  .map(Buffer::toJsonObject)
  .map(toCityAndDayLength())
  .subscribe(System.out::println, Throwable::printStackTrace);
```

现在让我们探索一下每个逻辑代码块是如何工作的。

## 5。城市名称

第一步是读取包含城市名称列表的文件，每行一个名称:

```java
fileSystem
 .rxReadFile("cities.txt").toFlowable()
 .flatMap(buffer -> Flowable.fromArray(buffer.toString().split("\\r?\\n"))) 
```

方法 `rxReadFile()`被动地读取一个文件并返回一个`RxJava`的`Single<Buffer>`。因此我们得到了我们正在寻找的集成:来自`RxJava`的数据结构中`Vert.x`的异步性。

只有一个文件，所以我们将得到一个包含文件全部内容的`Buffer`的单次发射。我们将输入转换成一个`RxJava`的`Flowable`，并且我们将文件的行平面映射成一个`Flowable`，它为每个城市名发出一个事件。

## 6。JSON 城市描述符

有了城市名，下一步是使用`Metaweather REST API`来获取该城市的标识符代码。这个标识符将用于获取城市的日出和日落时间。让我们继续调用链:

让我们继续调用链:

```java
.flatMap(city -> searchByCityName(httpClient, city))
.flatMap(HttpClientResponse::toFlowable) 
```

`searchByCityName()`方法使用我们在第一步中创建的`HttpClient`——来调用给出城市标识符的`REST`服务。然后通过第二个`flatMap(),`，我们得到包含响应的`Buffer`。

让我们完成这一步，编写`searchByCityName()`的主体:

```java
Flowable<HttpClientResponse> searchByCityName(HttpClient httpClient, String cityName) {
    HttpClientRequest req = httpClient.get(
        new RequestOptions()
          .setHost("www.metaweather.com")
          .setPort(443)
          .setSsl(true)
          .setURI(format("/api/location/search/?query=%s", cityName)));
    return req
      .toFlowable()
      .doOnSubscribe(subscription -> req.end());
}
```

`Vert.x`的`HttpClient`返回一个`RxJava`的`Flowable`，它发出被动的`HTTP`响应。这是在`Buffers`中依次发出响应拆分的正文。

我们为正确的 URL 创建了一个新的反应式请求，但是我们注意到 **`Vert.x`需要调用`HttpClientRequest.end()`方法**来发送请求，并且在成功调用`end()`之前还需要至少一个订阅。

实现这一点的一个解决方案是，一旦消费者订阅，就使用`RxJava`的 [`doOnSubscribe()`](https://web.archive.org/web/20221128112217/http://reactivex.io/documentation/operators/do.html) 来调用`end()`。

## 7。城市标识符

我们现在只需要获取返回的`JSON`对象的`woeid`属性的值，该值通过一个自定义方法唯一地标识了城市:

```java
.map(extractingWoeid())
```

`extractingWoeid()`方法返回一个函数，该函数从包含在`REST`服务响应中的`JSON`中提取城市标识符:

```java
private static Function<Buffer, Long> extractingWoeid() {
    return cityBuffer -> cityBuffer
      .toJsonArray()
      .getJsonObject(0)
      .getLong("woeid");
}
```

注意，我们可以使用由`Buffer`提供的便捷的`toJson…()`方法来快速访问我们需要的属性。

## 8。城市详情

让我们继续反应链，从`REST API`中检索我们需要的细节:

```java
.flatMap(cityId -> getDataByPlaceId(httpClient, cityId))
.flatMap(toBufferFlowable())
```

让我们详细介绍一下`getDataByPlaceId()`方法:

```java
static Flowable<HttpClientResponse> getDataByPlaceId(
  HttpClient httpClient, long placeId) {

    return autoPerformingReq(
      httpClient,
      format("/api/location/%s/", placeId));
}
```

这里，我们使用了与上一步相同的方法。`getDataByPlaceId()`返回一个`Flowable<HttpClientResponse>`。反过来，`HttpClientResponse`将会以块的形式发出`API`响应，如果它比这几个字节长的话。

使用`toBufferFlowable()`方法，我们将响应块缩减为单个块，以访问完整的 JSON 对象:

```java
static Function<HttpClientResponse, Publisher<? extends Buffer>>
  toBufferFlowable() {
    return response -> response
      .toObservable()
      .reduce(
        Buffer.buffer(),
        Buffer::appendBuffer).toFlowable();
}
```

## 9。日落和日出时间

让我们继续添加到反应链中，从`JSON`对象中检索我们感兴趣的信息:

```java
.map(toCityAndDayLength())
```

让我们来编写`toCityAndDayLength()`方法:

```java
static Function<JsonObject, CityAndDayLength> toCityAndDayLength() {
    return json -> {
        ZonedDateTime sunRise = ZonedDateTime.parse(json.getString("sun_rise"));
        ZonedDateTime sunSet = ZonedDateTime.parse(json.getString("sun_set"));
        String cityName = json.getString("title");
        return new CityAndDayLength(
          cityName, sunSet.toEpochSecond() - sunRise.toEpochSecond());
    };
}
```

它返回一个函数，该函数映射包含在`JSON`中的信息以创建一个`POJO`，该函数简单地计算日出和日落之间的时间(以小时为单位)。

## 10。订阅

反应链完成。我们现在可以使用一个处理程序来订阅结果`Flowable`，该处理程序打印出发出的`CityAndDayLength`的实例，或者错误情况下的堆栈跟踪:

```java
.subscribe(
  System.out::println, 
  Throwable::printStackTrace)
```

当我们运行应用程序时，我们可以看到这样的结果，这取决于列表中包含的城市和应用程序运行的日期:

```java
In Chicago there are 13.3 hours of light.
In Milan there are 13.5 hours of light.
In Cairo there are 12.9 hours of light.
In Moscow there are 14.1 hours of light.
In Santiago there are 11.3 hours of light.
In Auckland there are 11.2 hours of light.
```

城市可能会以不同于文件中指定的顺序出现，因为对`HTTP API`的所有请求都是异步执行的。

## 11。结论

在本文中，我们看到了将`Vert.x`反应式模块与`RxJava`提供的操作符和逻辑结构混合在一起是多么容易。

我们构建的反应链虽然很长，但展示了它如何让复杂的场景变得相当容易编写。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128112217/https://github.com/eugenp/tutorials/tree/master/vertx-modules/vertx-and-rxjava)