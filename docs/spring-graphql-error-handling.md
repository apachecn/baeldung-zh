# 用 Spring Boot 处理 GraphQL 中的错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-graphql-error-handling>

## 1.概观

在本教程中，我们将学习 [GraphQL](/web/20220602150241/https://www.baeldung.com/graphql) 中的错误处理选项。我们将看看 GraphQL 规范是如何描述错误响应的。因此，我们将开发一个使用 Spring Boot 处理 GraphQL 错误的例子。

## 2.符合 GraphQL 规范的响应

按照 GraphQL 规范，收到的每个请求都必须返回一个格式良好的响应。这种格式良好的响应由来自相应的成功或不成功的请求操作的数据或错误的映射组成。此外，响应可能包含部分成功的结果数据和字段错误。

**响应图的关键组成部分是`errors`、`data`和`extensions`。**

响应中的`errors `部分描述了请求操作期间的任何故障。如果没有错误发生，`errors `组件不能出现在响应中。在下一节中，我们将研究规范中描述的不同种类的错误。

`data `部分描述了成功执行所请求操作的结果。如果操作是查询，则该组件是查询根操作类型的对象。另一方面，如果操作是一个变异，那么这个组件就是变异根操作类型的一个对象。

如果请求的操作甚至在执行之前就由于信息缺失、验证错误或语法错误而失败，那么`data `组件就不能出现在响应中。如果在操作执行过程中操作失败，结果不成功，那么`data`组件必须是`null`。

响应映射可能包含一个名为`extensions`的附加组件，它是一个映射对象。该组件便于实现者在响应中提供他们认为合适的其他定制内容。因此，对其内容格式没有额外的限制。

**如果响应中不存在`data`组件，那么`errors`组件必须存在，并且必须包含至少一个错误。**此外，还应指出失败的原因。

下面是一个 GraphQL 错误的例子:

```java
mutation {
  addVehicle(vin: "NDXT155NDFTV59834", year: 2021, make: "Toyota", model: "Camry", trim: "XLE",
             location: {zipcode: "75024", city: "Dallas", state: "TX"}) {
    vin
    year
    make
    model
    trim
  }
}
```

违反唯一约束时的错误响应如下所示:

```java
{
  "data": null,
  "errors": [
    {
      "errorType": "DataFetchingException",
      "locations": [
        {
          "line": 2,
          "column": 5,
          "sourceName": null
        }
      ],
      "message": "Failed to add vehicle. Vehicle with vin NDXT155NDFTV59834 already present.",
      "path": [
        "addVehicle"
      ],
      "extensions": {
        "vin": "NDXT155NDFTV59834"
      }
    }
  ]
}
```

## 3.符合 GraphQL 规范的错误响应组件

响应中的`errors` 部分是一个非空的错误列表，每个错误都是一个映射。

### 3.1.请求错误

**顾名思义，如果请求本身有问题，请求错误可能会在操作执行之前发生。**这可能是由于请求数据解析失败、请求文档验证、不支持的操作或无效的请求值。

当请求错误发生时，这表明执行还没有开始，这意味着响应中的`data`部分不能出现在响应中。换句话说，响应只包含`errors`部分。

让我们看一个演示无效输入语法的例子:

```java
query {
  searchByVin(vin: "error) {
    vin
    year
    make
    model
    trim
  }
}
```

下面是对语法错误的请求错误响应，在本例中是缺少引号:

```java
{
  "data": null,
  "errors": [
    {
      "message": "Invalid Syntax",
      "locations": [
        {
          "line": 5,
          "column": 8,
          "sourceName": null
        }
      ],
      "errorType": "InvalidSyntax",
      "path": null,
      "extensions": null
    }
  ]
}
```

### 3.2.现场误差

**字段错误，顾名思义，可能是由于未能将值强制转换为预期类型，或者是在特定字段的值解析过程中出现内部错误。**表示在执行所请求的操作过程中出现现场错误。

如果出现字段错误，请求操作的**继续执行并返回部分结果**，这意味着响应的`data `部分必须与`errors `部分中的所有字段错误一起出现。

让我们看另一个例子:

```java
query {
  searchAll {
    vin
    year
    make
    model
    trim
  }
}
```

这一次，我们包含了 vehicle `trim`字段，根据我们的 GraphQL 模式，它应该是不可空的。

然而，其中一辆车的信息有一个空值`trim`,所以我们只得到部分数据——那些`trim`值不为空的车——以及错误:

```java
{
  "data": {
    "searchAll": [
      null,
      {
        "vin": "JTKKU4B41C1023346",
        "year": 2012,
        "make": "Toyota",
        "model": "Scion",
        "trim": "Xd"
      },
      {
        "vin": "1G1JC1444PZ215071",
        "year": 2000,
        "make": "Chevrolet",
        "model": "CAVALIER VL",
        "trim": "RS"
      }
    ]
  },
  "errors": [
    {
      "message": "Cannot return null for non-nullable type: 'String' within parent 'Vehicle' (/searchAll[0]/trim)",
      "path": [
        "searchAll",
        0,
        "trim"
      ],
      "errorType": "DataFetchingException",
      "locations": null,
      "extensions": null
    }
  ]
}
```

### 3.3.错误响应格式

正如我们前面看到的，响应中的`errors`是一个或多个错误的集合。**而且，每个错误都必须包含一个描述失败原因的关键字`message`，这样客户端开发人员就可以做出必要的更正来避免错误。**

**每个错误还可能包含一个名为`locations`** 的键，它是一个位置列表，指向所请求的 GraphQL 文档中与错误相关的一行。每个位置都是一个带有键的映射:分别是行和列，提供了相关元素的行号和开始列号。

**可能是错误一部分的另一个键称为`path`。**它提供了从根元素到响应中有错误的特定元素的值列表。如果字段值是列表，则`path`值可以是表示错误元素的字段名称或索引的字符串。如果错误与具有别名的字段有关，那么`path`中的值应该是别名。

### 3.4.处理现场错误

**无论字段错误是在可空字段还是不可空字段上出现，我们都应该像字段返回`null`一样处理它，并且错误必须添加到`errors`列表中。**

在可空字段的情况下，响应中的字段值将是`null`，但是`errors`必须包含描述失败原因和其他信息的字段错误，如前一节所述。

另一方面，父字段处理不可为空的字段错误。如果父字段不可为空，则错误处理会一直传播，直到到达可为空的父字段或根元素。

类似地，如果列表字段包含不可空的类型，并且一个或多个列表元素返回`null`，则整个列表解析为`null`。此外，如果包含列表字段的父字段不可为空，则错误处理会一直传播，直到到达可为空的父字段或根元素。

出于任何原因，**如果在解析过程中同一字段出现多个错误，那么对于该字段，我们必须在`errors`** 中只添加一个字段错误。

## 4.Spring Boot GraphQL 图书馆

我们的 Spring Boot 应用程序示例使用了`[graphql-spring-boot-starter](https://web.archive.org/web/20220602150241/https://search.maven.org/search?q=g:com.graphql-java%20AND%20a:graphql-spring-boot-starter) `模块，它引入了`graphql-java-servlet`和`graphql-java`。

我们还使用了 [`graphql-java-tools`](https://web.archive.org/web/20220602150241/https://search.maven.org/search?q=g:com.graphql-java%20AND%20a:graphql-java-tools) 模块，这有助于将 GraphQL 模式映射到现有的 Java 对象，对于单元测试，我们使用了 [`graphql-spring-boot-starter-test`](https://web.archive.org/web/20220602150241/https://search.maven.org/search?q=g:com.graphql-java%20AND%20a:graphql-spring-boot-starter-test) :

```java
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-spring-boot-starter</artifactId>
    <version>5.0.2</version>
</dependency>

<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java-tools</artifactId>
    <version>5.2.4</version>
</dependency>
```

对于测试，我们使用`graphql-spring-boot-starter-test`:

```java
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-spring-boot-starter-test</artifactId>
    <version>5.0.2</version>
    <scope>test</scope>
</dependency>
```

## 5.Spring Boot GraphQL 错误处理

在这一节中，我们将主要介绍 Spring Boot 应用程序本身中的 GraphQL 错误处理。我们不会讨论 GraphQL Java 和 GraphQL Spring Boot 应用程序的开发。

在我们的 Spring Boot 应用程序示例中，我们将根据位置或 VIN(车辆识别号)变异或查询车辆。通过这个例子，我们将看到实现错误处理的不同方法。

`graphql-java-servlet `模块提供了一个名为`GraphQLErrorHandler. `的接口，我们可以提供它的实现。

**在下面的小节中，我们将看到`graphql-java-servlet`模块如何使用来自`graphql-java `模块的组件来处理异常或错误。**

### 5.1.带有标准异常的 GraphQL 响应

通常，在 REST 应用程序中，我们通过扩展`RuntimeException`或`Throwable`来创建一个定制的运行时异常类:

```java
public class InvalidInputException extends RuntimeException {
    public InvalidInputException(String message) {
        super(message);
    }
}
```

使用这种方法，我们可以看到 GraphQL 引擎返回以下响应:

```java
{
  "data": null,
  "errors": [
    {
      "message": "Internal Server Error(s) while executing query",
      "path": null,
      "extensions": null
    }
  ]
}
```

在上面的错误响应中，我们可以看到它不包含错误的任何细节。

默认情况下，**任何自定义异常都由`SimpleDataFetcherExceptionHandler `类**处理[。它将原始异常连同源位置和执行路径(如果存在的话)包装到另一个名为`ExceptionWhileDataFetching.` 的异常中，然后将错误添加到`errors`集合中。反过来，`ExceptionWhileDataFetching`实现了`GraphQLError `接口。](https://web.archive.org/web/20220602150241/https://www.graphql-java.com/documentation/execution#exceptions-while-fetching-data)

在`SimpleDataFetcherExceptionHandler `处理程序之后，**另一个名为`DefaultGraphQLErrorHandler`的处理程序处理错误集合**。它将所有类型为`GraphQLError`的异常隔离为客户端错误。但是除此之外，它还为所有其他非客户端错误创建一个`GenericGraphQLError`异常(如果存在的话)。

在上面的例子中，`InvalidInputException`不是客户端错误，因为它只扩展了`RuntimeException`而没有实现`GraphQLError`。因此，`DefaultGraphQLErrorHandler `处理程序创建了一个`GenericGraphQLError`异常，代表带有内部服务器错误消息的`InvalidInputException`。

### 5.2.类型`GraphQLError`异常的 GraphQL 响应

现在让我们看看如果我们实现我们的定制异常，响应会是什么样子，因为`GraphQLError. ``GraphQLError`是一个接口，它允许我们通过实现`getExtensions()`方法来提供关于错误的更多信息。

让我们实现我们的自定义异常:

```java
public class AbstractGraphQLException extends RuntimeException implements GraphQLError {
    private Map<String, Object> parameters = new HashMap();

    public AbstractGraphQLException(String message) {
        super(message);
    }

    public AbstractGraphQLException(String message, Map<String, Object> additionParams) {
        this(message);
        if (additionParams != null) {
            parameters = additionParams;
        }
    }

    @Override
    public String getMessage() {
        return super.getMessage();
    }

    @Override
    public List<SourceLocation> getLocations() {
        return null;
    }

    @Override
    public ErrorType getErrorType() {
        return null;
    }

    @Override
    public Map<String, Object> getExtensions() {
        return this.parameters;
    }
}
```

```java
public class VehicleAlreadyPresentException extends AbstractGraphQLException {

     public VehicleAlreadyPresentException(String message) {
         super(message);
     }

    public VehicleAlreadyPresentException(String message, Map<String, Object> additionParams) {
        super(message, additionParams);
    }
}
```

正如我们在上面的代码片段中看到的，我们已经为`getLocations()`和`getErrorType()`方法返回了`null`，因为默认包装异常`ExceptionWhileDataFetching`只调用了我们自定义包装异常的`getMesssage()`和`getExtensions()`方法。

正如我们在前面章节中看到的，`SimpleDataFetcherExceptionHandler`类处理数据获取错误。**让我们看看`graphql-java`库如何帮助我们设置`path`、`locations`和错误`type`。**

下面的代码片段显示了 GraphQL 引擎执行使用`DataFetcherExceptionHandlerParameters`类来设置错误字段的位置和路径。这些值作为构造函数参数传递给`ExceptionWhileDataFetching`:

```java
...
public void accept(DataFetcherExceptionHandlerParameters handlerParameters) {
        Throwable exception = handlerParameters.getException();
        SourceLocation sourceLocation = handlerParameters.getField().getSourceLocation();
        ExecutionPath path = handlerParameters.getPath();

        ExceptionWhileDataFetching error = new ExceptionWhileDataFetching(path, exception, sourceLocation);
        handlerParameters.getExecutionContext().addError(error);
        log.warn(error.getMessage(), exception);
}
...
```

让我们来看一个来自`ExceptionWhileDataFetching` 类的片段。在这里，我们可以看到错误类型是`DataFetchingException`:

```java
...
@Override
public List<SourceLocation> getLocations() {
    return locations;
}

@Override
public List<Object> getPath() {
    return path;
}

@Override
public ErrorType getErrorType() {
    return ErrorType.DataFetchingException;
}
...
```

## 6.结论

在本教程中，我们学习了不同类型的 GraphQL 错误。我们还研究了如何根据规范格式化 GraphQL 错误。后来，我们在 Spring Boot 应用程序中实现了错误处理。

请注意， [Spring](https://web.archive.org/web/20220602150241/https://docs.spring.io/spring-graphql/docs/1.0.0-M5/reference/html/) 团队与 [GraphQL Java](https://web.archive.org/web/20220602150241/https://www.graphql-java.com/) 团队合作，正在用 GraphQL 为 Spring Boot 开发一个新的库`spring-boot-starter-graphql`。它仍然处于里程碑式的发布阶段，还没有正式发布。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220602150241/https://github.com/eugenp/tutorials/tree/master/graphql/graphql-error-handling)