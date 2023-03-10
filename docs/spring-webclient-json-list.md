# 使用 WebClient 获取 JSON 对象列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webclient-json-list>

## 1.概观

我们的服务经常与其他 REST 服务通信以获取信息。

从 Spring 5 开始，我们开始使用 [`WebClient`](/web/20220525124321/https://www.baeldung.com/spring-5-webclient) 以一种被动的、非阻塞的方式来执行这些请求。`WebClient`是新的`WebFlux`框架的一部分，构建在`Project Reactor`之上。它有一个流畅的、反应式的 API，并且在其底层实现中使用 HTTP 协议。

当我们发出 web 请求时，数据通常以 JSON 的形式返回。`WebClient`可以为我们转换这个。

在本文中，我们将了解如何使用`WebClient`将 JSON 数组转换成`Object`的 Java `Array`、 [POJO 的`Array`和 POJO 的`List`。](/web/20220525124321/https://www.baeldung.com/java-pojo-class#what-is-a-pojo)

## 2.属国

为了使用`WebClient,`,我们需要给我们的`pom.xml:` 添加几个依赖项

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.projectreactor</groupId>
    <artifactId>reactor-spring</artifactId>
    <version>1.0.1.RELEASE</version>
</dependency>
```

## 3.JSON、POJO 和服务

让我们从一个端点`http://localhost:8080/readers`开始，它以 JSON 数组的形式返回一个读者列表，其中包含他们最喜欢的书籍:

```java
[{
    "id": 1,
    "name": "reader1",
    "favouriteBook": { 
        "author": "Milan Kundera",
        "title": "The Unbearable Lightness of Being"
    } }, {
    "id": 2,
    "name": "reader2"
    "favouriteBook": { 
        "author": "Douglas Adams",
        "title": "The Hitchhiker's Guide to the Galaxy"
    }
}]
```

我们将需要相应的`Reader` 和`Book` 类来处理数据:

```java
public class Reader {
    private int id;
    private String name;
    private Book favouriteBook;

    // getters and setters..
}
```

```java
public class Book {
    private final String author;
    private final String title;

   // getters and setters..
}
```

对于我们的接口实现，我们写了一个`ReaderConsumerServiceImpl` ，用`WebClient` 作为它的依赖:

```java
public class ReaderConsumerServiceImpl implements ReaderConsumerService {

    private final WebClient webClient;

    public ReaderConsumerServiceImpl(WebClient webclient) {
        this.webclient = webclient;
    }

    // ...
}
```

## 4.映射 JSON 对象的一个`List`

当我们从 REST 请求中接收到一个 JSON 数组时，有多种方法可以将其转换成 Java 集合。让我们看看各种选项，看看处理返回的数据有多容易。我们将着眼于提取读者最喜爱的书籍。

### 4.1.`Mono`对`Flux`

`Project Reactor `已经介绍了`Publisher: **Mono**`和`**Flux**`的两个实现。

当我们需要处理零个到多个或者潜在的无限个结果时,`Flux<T>`非常有用。我们可以把 Twitter feed 看作一个例子。

当我们知道结果是一次返回的时候——就像在我们的用例中一样——我们可以使用`Mono<T>`。

### 4.2.`WebClient`用`Object`排列

首先，让我们用`WebClient.get`进行`GET`调用，并使用一个`Object[]`类型的 `Mono` 来收集响应:

```java
Mono<Object[]> response = webClient.get()
  .accept(MediaType.APPLICATION_JSON)
  .retrieve()
  .bodyToMono(Object[].class).log();
```

接下来，让我们将尸体提取到我们的`Object`数组中:

```java
Object[] objects = response.block();
```

这里实际的`Object`是一个包含我们数据的任意结构。让我们把它转换成一个`Reader`对象的数组。

为此，我们需要一个`ObjectMapper`:

```java
ObjectMapper mapper = new ObjectMapper();
```

这里，我们内联声明了它，尽管这通常是作为类的`private static final`成员来完成的。

最后，我们准备提取读者最喜欢的书籍，并将其收集到一个列表中:

```java
return Arrays.stream(objects)
  .map(object -> mapper.convertValue(object, Reader.class))
  .map(Reader::getFavouriteBook)
  .collect(Collectors.toList());
```

当我们要求 [Jackson 反序列化器](/web/20220525124321/https://www.baeldung.com/jackson-object-mapper-tutorial)产生`Object`作为目标类型时，它实际上**将 JSON 反序列化成一系列`LinkedHashMap`对象**。用`convertValue`进行后期处理效率很低。如果我们在反序列化期间向 Jackson 提供我们想要的类型，就可以避免这种情况。

### 4.3.`WebClient`用`Reader` 排列

我们可以向我们的`WebClient`提供`Reader[]`而不是`Object[]`:

```java
Mono<Reader[]> response = webClient.get()
  .accept(MediaType.APPLICATION_JSON)
  .retrieve()
  .bodyToMono(Reader[].class).log();
Reader[] readers = response.block();
return Arrays.stream(readers)
  .map(Reader:getFavouriteBook)
  .collect(Collectors.toList());
```

在这里，我们可以观察到我们不再需要`ObjectMapper.convertValue`。然而，我们仍然需要做额外的转换来使用 Java `Stream` API，并让我们的代码使用`List`。

### 4.4.`WebClient`同`Reader` `List`

如果我们希望 Jackson 产生一个由 `Reader`组成的`List`而不是一个数组，我们需要描述我们想要创建的`List`。为此，我们向该方法提供了一个由[匿名内部类](/web/20220525124321/https://www.baeldung.com/java-anonymous-classes)生成的`ParameterizedTypeReference` :

```java
Mono<List<Reader>> response = webClient.get()
  .accept(MediaType.APPLICATION_JSON)
  .retrieve()
  .bodyToMono(new ParameterizedTypeReference<List<Reader>>() {});
List<Reader> readers = response.block();

return readers.stream()
  .map(Reader::getFavouriteBook)
  .collect(Collectors.toList());
```

这给了我们可以利用的`List`。

让我们更深入地了解一下**为什么我们需要使用** `**ParameterizedTypeReference**.`

当类型信息在运行时可用时，Spring 的 WebClient 可以很容易地将 JSON 反序列化为一个`Reader.class`。

然而，对于泛型，如果我们试图使用`List<Reader>.class`，就会出现[类型擦除](/web/20220525124321/https://www.baeldung.com/java-type-erasure)。因此，Jackson 将无法确定泛型的类型参数。

通过使用`ParameterizedTypeReference`，我们可以克服这个问题。将它实例化为匿名内部类利用了这样一个事实，即泛型类的子类包含编译时类型信息，这些信息不会被类型擦除，并且可以通过反射使用。

## 5.结论

在本教程中，我们看到了使用`WebClient` `.` 处理 JSON 对象的三种不同方式，我们看到了指定`Object`数组类型的方式以及我们自己的自定义类。

然后我们学习了如何通过使用`ParameterizedTypeReference`来提供产生`List`的信息类型。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220525124321/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-client)