# Derive4J 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/derive4j>

## 1.介绍

Derive4J 是一个注释处理器，支持 Java 8 中的各种功能概念。

在本教程中，我们将介绍 Derive4J 和该框架支持的最重要的概念:

*   代数数据类型
*   结构模式匹配
*   一流的懒惰

## 2.Maven 依赖性

要使用 Derive4J，我们需要将[依赖项](https://web.archive.org/web/20221128043333/https://search.maven.org/search?q=g:org.derive4j%20AND%20a:derive4j)包含到我们的项目中:

```java
<dependency>
    <groupId>org.derive4j</groupId>
    <artifactId>derive4j</artifactId>
    <version>1.1.0</version>
    <optional>true</optional>
</dependency>
```

## 3.代数数据类型

### 3.1.描述

代数数据类型(ADT)是一种复合类型——它们是其他类型或泛型的组合。

ADT 通常分为两大类:

*   总和
*   产品

代数数据类型默认出现在许多语言中，比如 Haskell 和 Scala。

### 3.2.总和类型

**Sum 是表示逻辑 OR 运算的数据类型。**这意味着它可以是一件事或另一件事，但不能两者都是。简单来说，sum 类型就是一组不同的 cases。“sum”这个名称来源于这样一个事实，即不同值的总数就是事例的总数。

`Enum `是 Java 中最接近 sum 类型的东西。`Enum`有一组可能的值，但一次只能有一个。但是，**在 Java 中我们不能将任何额外的数据与`Enum`相关联，这是代数数据类型相对于`Enum`的主要优势。**

### 3.3.产品类型

**Product 是表示逻辑 AND 运算的数据类型。**是几个值的组合。

`Class`在 Java 中可以认为是一个产品类型。产品类型由其字段的组合来定义。

我们可以在这篇维基百科文章中找到更多关于 ADTs [的信息。](https://web.archive.org/web/20221128043333/https://en.wikipedia.org/wiki/Algebraic_data_type)

### 3.4.使用

常用的代数数据类型之一是`Either. `,我们可以把`Either`看作是一个更复杂的`Optional`,当有可能丢失值或者操作可能导致异常时，可以使用它。

我们需要用至少一个抽象方法来注释一个`abstract class`或`interface`，Derive4J 将使用该方法来生成我们的 ADT 的结构。

为了在 Derive4J 中创建`Either` 数据类型，我们需要创建一个`interface`:

```java
@Data
interface Either<A, B> {
    <X> X match(Function<A, X> left, Function<B, X> right);
}
```

我们的`interface`用`@Data`注释，这将允许 Derive4J 为我们生成适当的代码。生成的代码包含工厂方法、惰性构造函数和各种其他方法。

默认情况下，生成的代码获得带注释的名称`class`，但是是复数形式。但是，有可能通过`inClass`参数进行配置。

现在，我们可以使用生成的代码来创建`Either` ADT，并验证它是否正常工作:

```java
public void testEitherIsCreatedFromRight() {
    Either<Exception, String> either = Eithers.right("Okay");
    Optional<Exception> leftOptional = Eithers.getLeft(either);
    Optional<String> rightOptional = Eithers.getRight(either);
    Assertions.assertThat(leftOptional).isEmpty();
    Assertions.assertThat(rightOptional).hasValue("Okay");
}
```

我们还可以使用生成的`match() `方法来执行一个函数，这取决于`Either` 的哪一侧存在:

```java
public void testEitherIsMatchedWithRight() {
    Either<Exception, String> either = Eithers.right("Okay");
    Function<Exception, String> leftFunction = Mockito.mock(Function.class);
    Function<String, String> rightFunction = Mockito.mock(Function.class);
    either.match(leftFunction, rightFunction);
    Mockito.verify(rightFunction, Mockito.times(1)).apply("Okay");
    Mockito.verify(leftFunction, Mockito.times(0)).apply(Mockito.any(Exception.class));
}
```

## 4.模式匹配

通过使用代数数据类型实现的特性之一是**模式匹配。**

**模式匹配是根据模式检查值的机制。**基本上，模式匹配是一个更强大的`[switch](/web/20221128043333/https://www.baeldung.com/java-switch)`语句，但是对匹配类型没有限制，也不要求模式是常量。更多信息，我们可以查看[这篇关于模式匹配](https://web.archive.org/web/20221128043333/https://en.wikipedia.org/wiki/Pattern_matching)的维基百科文章。

为了使用模式匹配，我们将创建一个类来模拟 HTTP 请求。用户将能够使用给定的 HTTP 方法之一:

*   得到
*   邮政
*   删除
*   放

让我们将我们的请求类建模为 Derive4J 中的 ADT，从`HTTPRequest`接口开始:

```java
@Data
interface HTTPRequest {
    interface Cases<R>{
        R GET(String path);
        R POST(String path);
        R PUT(String path);
        R DELETE(String path);
    }

    <R> R match(Cases<R> method);
}
```

生成的类`HttpRequests`(注意复数形式)，现在将允许我们基于请求的类型执行模式匹配。

为此，我们将创建一个非常简单的`HTTPServer `类，它将根据请求的类型用不同的`Status`进行响应。

首先，让我们创建一个简单的`HTTPResponse `类，它将作为服务器对客户端的响应:

```java
public class HTTPResponse {
    int statusCode;
    String responseBody;

    public HTTPResponse(int statusCode, String responseBody) {
        this.statusCode = statusCode;
        this.responseBody = responseBody;
    }
}
```

然后，我们可以创建使用模式匹配来发送正确响应的服务器:

```java
public class HTTPServer {
    public static String GET_RESPONSE_BODY = "Success!";
    public static String PUT_RESPONSE_BODY = "Resource Created!";
    public static String POST_RESPONSE_BODY = "Resource Updated!";
    public static String DELETE_RESPONSE_BODY = "Resource Deleted!";

    public HTTPResponse acceptRequest(HTTPRequest request) {
        return HTTPRequests.caseOf(request)
          .GET((path) -> new HTTPResponse(200, GET_RESPONSE_BODY))
          .POST((path,body) -> new HTTPResponse(201, POST_RESPONSE_BODY))
          .PUT((path,body) -> new HTTPResponse(200, PUT_RESPONSE_BODY))
          .DELETE(path -> new HTTPResponse(200, DELETE_RESPONSE_BODY));
    }
}
```

我们的`class`的`acceptRequest() `方法对请求的类型使用模式匹配，并将根据请求的类型返回不同的响应:

```java
@Test
public void whenRequestReachesServer_thenProperResponseIsReturned() {
    HTTPServer server = new HTTPServer();
    HTTPRequest postRequest = HTTPRequests.POST("http://test.com/post", "Resource");
    HTTPResponse response = server.acceptRequest(postRequest);
    Assert.assertEquals(201, response.getStatusCode());
    Assert.assertEquals(HTTPServer.POST_RESPONSE_BODY, response.getResponseBody());
}
```

## 5.一流的懒惰

Derive4J 允许我们引入惰性的概念，这意味着我们的对象在我们对它们执行一个操作之前不会被初始化。让我们将`interface`声明为`LazyRequest `，并将生成的类配置为命名为`LazyRequestImpl`:

```java
@Data(value = @Derive(
  inClass = "{ClassName}Impl",
  make = {Make.lazyConstructor, Make.constructors}
))
public interface LazyRequest {
    interface Cases<R>{
        R GET(String path);
        R POST(String path, String body);
        R PUT(String path, String body);
        R DELETE(String path);
    }

    <R> R match(LazyRequest.Cases<R> method);
}
```

我们现在可以验证生成的惰性构造函数是否正常工作:

```java
@Test
public void whenRequestIsReferenced_thenRequestIsLazilyContructed() {
    LazyRequestSupplier mockSupplier = Mockito.spy(new LazyRequestSupplier());
    LazyRequest request = LazyRequestImpl.lazy(() -> mockSupplier.get());
    Mockito.verify(mockSupplier, Mockito.times(0)).get();
    Assert.assertEquals(LazyRequestImpl.getPath(request), "http://test.com/get");
    Mockito.verify(mockSupplier, Mockito.times(1)).get();
}

class LazyRequestSupplier implements Supplier<LazyRequest> {
    @Override
    public LazyRequest get() {
        return LazyRequestImpl.GET("http://test.com/get");
    }
}
```

我们可以在 Scala 文档中找到更多关于一流懒惰和例子[的信息。](https://web.archive.org/web/20221128043333/https://www.scala-lang.org/blog/2017/11/28/view-based-collections.html)

## 6.结论

在本教程中，我们介绍了 Derive4J 库，并使用它来实现一些函数概念，如代数数据类型和模式匹配，这些在 Java 中通常是不可用的。

我们可以在官方 Derive4J 文档中找到关于该库的更多信息[。](https://web.archive.org/web/20221128043333/https://github.com/derive4j/derive4j)

和往常一样，所有代码样本都可以在 GitHub 上找到[。](https://web.archive.org/web/20221128043333/https://github.com/eugenp/tutorials/tree/master/libraries-data-2)