# 探索球衣请求参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jersey-request-parameters>

## 1.介绍

Jersey 是一个用于创建 RESTful web 服务的流行 Java 框架。

在本教程中，我们将探索如何通过一个简单的 Jersey 项目读取不同的请求参数类型。

## 2.项目设置

使用 Maven 原型，我们将能够为我们的文章生成一个工作项目:

```java
mvn archetype:generate -DarchetypeArtifactId=jersey-quickstart-grizzly2
  -DarchetypeGroupId=org.glassfish.jersey.archetypes -DinteractiveMode=false
  -DgroupId=com.example -DartifactId=simple-service -Dpackage=com.example
  -DarchetypeVersion=2.28
```

生成的 Jersey 项目将在 Grizzly 容器上运行。

现在，默认情况下，我们应用程序的端点将是`[http://localhost:8080/myapp](https://web.archive.org/web/20221026123345/http://localhost:8080/myapp).`

让我们添加一个`items`资源，我们将在实验中使用它:

```java
@Path("items")
public class ItemsController {
    // our endpoints are defined here
}
```

顺便说一句，注意[运动衫也可以很好地配合弹簧控制器](/web/20221026123345/https://www.baeldung.com/jersey-rest-api-with-spring)。

## 3.带注释的参数类型

因此，在我们实际读取任何请求参数之前，让我们澄清一些规则。**允许的参数类型有:**

*   原始类型，如`float` 和`char`
*   具有带单个`String`参数的构造函数的类型
*   类型，有 `fromString` 或 `valueOf`静态方法；对于那些，一个 单个 `String` 自变量是必选的
*   集合——如上述类型的`List`、`Set`和`SortedSet –`

此外，我们可以**注册一个`ParamConverterProvider`** JAX-RS 扩展 SPI 的实现。返回类型必须是能够从`String`转换为类型的 `ParamConverter`实例。

## 4.饼干

我们可以使用`@CookieParam`注释在 Jersey 方法中解析 [cookie 值](/web/20221026123345/https://www.baeldung.com/cookies-java):

```java
@GET
public String jsessionid(@CookieParam("JSESSIONId") String jsessionId) {
    return "Cookie parameter value is [" + jsessionId+ "]";
}
```

If we start up our container, we can [cURL](/web/20221026123345/https://www.baeldung.com/curl-rest) this endpoint to see the response:

```java
> curl --cookie "JSESSIONID=5BDA743FEBD1BAEFED12ECE124330923" http://localhost:8080/myapp/items
Cookie parameter value is [5BDA743FEBD1BAEFED12ECE124330923]
```

## 5.头球

或者，我们可以用的`@HeaderParam`注释来解析[的 HTTP 头](https://web.archive.org/web/20221026123345/https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers):

```java
@GET
public String contentType(@HeaderParam("Content-Type") String contentType) {
    return "Header parameter value is [" + contentType+ "]";
}
```

让我们再测试一次:

```java
> curl --header "Content-Type: text/html" http://localhost:8080/myapp/items
Header parameter value is [text/html]
```

## 6.路径参数

尤其是对于 RESTful APIs，在路径中包含信息是很常见的。

我们可以用`@PathParam`提取路径元素:

```java
@GET
@Path("/{id}")
public String itemId(@PathParam("id") Integer id) {
    return "Path parameter value is [" + id + "]";
}
```

让我们发送另一个值为`3`的`curl`命令:

```java
> curl http://localhost:8080/myapp/items/3
Path parameter value is [3]
```

## 7.查询参数

我们通常在 RESTful APIs 中使用查询参数来获取可选信息。

要读取这些值，我们可以使用`@QueryParam`注释:

```java
@GET
public String itemName(@QueryParam("name") String name) {
    return "Query parameter value is [" + name + "]";
}
```

所以，现在我们可以用`curl`进行测试，就像以前一样:

```java
> curl http://localhost:8080/myapp/items?name=Toaster
Query parameter value if [Toaster]
```

## 8.表单参数

For reading parameters from a form submission, we’ll use the `@FormParam` annotation:

```java
@POST
public String itemShipment(@FormParam("deliveryAddress") String deliveryAddress, 
  @FormParam("quantity") Long quantity) {
    return "Form parameters are [deliveryAddress=" + deliveryAddress+ ", quantity=" + quantity + "]";
}
```

我们还需要设置适当的`Content-Type`来模仿表单提交动作。让我们使用`-d`标志:来设置表单参数

```java
> curl -X POST -H 'Content-Type:application/x-www-form-urlencoded' \
  -d 'deliveryAddress=Washington nr 4&quantity;=5' \
  http://localhost:8080/myapp/items
Form parameters are [deliveryAddress=Washington nr 4, quantity=5]
```

## 9.矩阵参数

**[矩阵参数](/web/20221026123345/https://www.baeldung.com/spring-mvc-matrix-variables)是一个更灵活的查询参数，因为它们可以添加到 URL 中的任何位置。**

比如在`http://localhost:8080/myapp;name=value/items`中，矩阵参数是`name`。

为了读取这些值，我们可以使用可用的`@MatrixParam`注释:

```java
@GET
public String itemColors(@MatrixParam("colors") List<String> colors) {
    return "Matrix parameter values are " + Arrays.toString(colors.toArray());
}
```

现在我们将再次测试我们的端点:

```java
> curl http://localhost:8080/myapp/items;colors=blue,red
Matrix parameter values are [blue,red]
```

## 10.Bean 参数

最后，我们将检查如何使用 bean 参数组合请求参数。澄清一下，[bean 参数](/web/20221026123345/https://www.baeldung.com/jersey-bean-validation)实际上是一个组合了不同类型的请求参数的对象。

我们将在这里使用一个头参数、一个路径和一个表单:

```java
public class ItemOrder {
    @HeaderParam("coupon")
    private String coupon;

    @PathParam("itemId")
    private Long itemId;

    @FormParam("total")
    private Double total;

    //getter and setter

    @Override
    public String toString() {
        return "ItemOrder {coupon=" + coupon + ", itemId=" + itemId + ", total=" + total + '}';
    }
}
```

此外，为了获得这样的参数组合，我们将使用`@BeanParam`注释:

```java
@POST
@Path("/{itemId}")
public String itemOrder(@BeanParam ItemOrder itemOrder) {
    return itemOrder.toString();
}
```

在`curl`命令中，我们添加了这三种类型的参数，最终我们将得到一个`ItemOrder` 对象:

```java
> curl -X POST -H 'Content-Type:application/x-www-form-urlencoded' \
  --header 'coupon:FREE10p' \
  -d total=70 \
  http://localhost:8080/myapp/items/28711
ItemOrder {coupon=FREE10p, itemId=28711, total=70}
```

## 11.结论

总而言之，我们为 Jersey 项目创建了一个简单的设置，帮助我们探索如何使用 Jersey 从请求中读取不同的参数。

Github 上的[提供了所有这些代码片段的实现。](https://web.archive.org/web/20221026123345/https://github.com/eugenp/tutorials/tree/master/jersey)