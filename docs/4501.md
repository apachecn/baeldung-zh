# 新泽西的 Bean 验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jersey-bean-validation>

## 1。概述

在本教程中，我们将看看使用开源框架 [Jersey](https://web.archive.org/web/20220627172045/https://jersey.github.io/) 的 Bean 验证。

正如我们在以前的文章中已经看到的，Jersey 是一个用于开发 RESTful Web 服务的开源框架。在关于如何[用 Jersey 和 Spring 创建 API 的介绍中，我们可以获得更多关于 Jersey 的细节。](/web/20220627172045/https://www.baeldung.com/jersey-rest-api-with-spring)

## 2。比恩在新泽西验证

**验证是验证某些数据符合一个或多个预定义约束**的过程。当然，在大多数应用中，这是一个非常常见的用例。

Java Bean [验证](https://web.archive.org/web/20220627172045/https://beanvalidation.org/)框架(JSR-380)已经成为处理这种 Java 操作的事实上的标准。回顾一下 Java Bean 验证的基础知识，请参考我们之前的[教程](/web/20220627172045/https://www.baeldung.com/javax-validation)。

**Jersey 包含一个支持 Bean 验证的扩展模块**。要在我们的应用程序中使用这一功能，我们首先需要配置它。在下一节中，我们将看到如何配置我们的应用程序。

## 3。应用程序设置

现在，让我们在优秀的 [Jersey MVC Support](/web/20220627172045/https://www.baeldung.com/jersey-mvc) 文章中的简单水果 API 示例的基础上进行构建。

### 3.1。Maven 依赖关系

首先，让我们将 Bean 验证依赖项添加到我们的`pom.xml`:

```
<dependency>
    <groupId>org.glassfish.jersey.ext</groupId>
    <artifactId>jersey-bean-validation</artifactId>
    <version>2.27</version>
</dependency>
```

我们可以从 [Maven Central](https://web.archive.org/web/20220627172045/https://search.maven.org/classic/#search%7Cga%7C1%7Cjersey-bean-validation) 获得最新版本。

### 3.2。配置服务器

在 Jersey 中，我们通常在自定义资源配置类中注册我们想要使用的扩展特性。

但是，对于 bean 验证扩展，不需要进行这种注册。幸运的是，这是 Jersey 框架自动注册的少数扩展之一。

最后，为了向客户机**发送验证错误，我们将向自定义资源配置**添加一个服务器属性:

```
public ViewApplicationConfig() {
    packages("com.baeldung.jersey.server");
    property(ServerProperties.BV_SEND_ERROR_IN_RESPONSE, true);
} 
```

## 4。验证 JAX 遥感资源方法

在这一节中，我们将解释使用约束注释来验证输入参数的两种不同方式:

*   使用内置 Bean 验证 API 约束
*   创建自定义约束和验证器

### 4.1。使用内置约束注释

让我们从查看内置约束注释开始:

```
@POST
@Path("/create")
@Consumes(MediaType.APPLICATION_FORM_URLENCODED)
public void createFruit(
    @NotNull(message = "Fruit name must not be null") @FormParam("name") String name, 
    @NotNull(message = "Fruit colour must not be null") @FormParam("colour") String colour) {

    Fruit fruit = new Fruit(name, colour);
    SimpleStorageService.storeFruit(fruit);
} 
```

在这个例子中，我们使用两个表单参数`name`和`colour`创建了一个新的`Fruit`。我们使用`@NotNull`注释，它已经是 Bean 验证 API 的一部分。

这给我们的表单参数强加了一个简单的 not null 约束。**如果其中一个参数是`null`，那么注释中声明的消息将被返回**。

自然，我们将用一个单元测试来证明这一点:

```
@Test
public void givenCreateFruit_whenFormContainsNullParam_thenResponseCodeIsBadRequest() {
    Form form = new Form();
    form.param("name", "apple");
    form.param("colour", null);
    Response response = target("fruit/create").request(MediaType.APPLICATION_FORM_URLENCODED)
        .post(Entity.form(form));

    assertEquals("Http Response should be 400 ", 400, response.getStatus());
    assertThat(response.readEntity(String.class), containsString("Fruit colour must not be null"));
} 
```

在上面的例子中，**我们使用`JerseyTest`支持类来测试我们的水果资源**。我们发送一个空的 POST 请求，并检查响应是否包含预期的消息。

要获得内置验证约束的列表，请看文档中的[。](https://web.archive.org/web/20220627172045/https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#section-builtin-constraints)

### 4.2。定义自定义约束注释

有时我们需要施加更复杂的约束。我们可以通过定义自己的自定义注释来做到这一点。

使用我们简单的水果 API 示例，假设我们需要验证所有水果都有有效的序列号:

```
@PUT
@Path("/update")
@Consumes("application/x-www-form-urlencoded")
public void updateFruit(@SerialNumber @FormParam("serial") String serial) {
    //...
} 
```

在这个例子中，参数`serial`必须满足由`@SerialNumber`定义的约束，我们接下来将定义这些约束。

我们将首先定义约束注释:

```
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = { SerialNumber.Validator.class })
    public @interface SerialNumber {

    String message()

    default "Fruit serial number is not valid";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
} 
```

接下来，我们将定义验证器类`SerialNumber.Validator`:

```
public class Validator implements ConstraintValidator<SerialNumber, String> {
    @Override
    public void initialize(SerialNumber serial) {
    }

    @Override
    public boolean isValid(String serial, 
        ConstraintValidatorContext constraintValidatorContext) {

        String serialNumRegex = "^\\d{3}-\\d{3}-\\d{4}$";
        return Pattern.matches(serialNumRegex, serial);
    }
} 
```

这里的关键点是`Validator`类必须 实现 `ConstraintValidator`其中`T`是我们想要 验证 的值类型，在我们的例子中是`String` 。

**最后，我们然后在`isValid`方法**中实现我们的自定义验证逻辑。

## 5。资源验证

**此外，Bean 验证 API 还允许我们使用`@Valid`注释**来验证对象。

在下一节中，我们将解释使用该注释验证资源类的两种不同方式:

*   首先，请求资源验证
*   第二，响应资源验证

让我们从给我们的`Fruit`对象添加`@Min`注释开始:

```
@XmlRootElement
public class Fruit {

    @Min(value = 10, message = "Fruit weight must be 10 or greater")
    private Integer weight;
    //...
} 
```

### 5.1。请求资源验证

首先，我们将在我们的`FruitResource` 类中使用`@Valid`来启用验证:

```
@POST
@Path("/create")
@Consumes("application/json")
public void createFruit(@Valid Fruit fruit) {
    SimpleStorageService.storeFruit(fruit);
} 
```

在上面的例子中，**如果我们试图创建一个权重小于 10 的水果，我们将得到一个验证错误。**

### 5.2。响应资源验证

同样，在下一个示例中，我们将看到如何验证响应资源:

```
@GET
@Valid
@Produces("application/json")
@Path("/search/{name}")
public Fruit findFruitByName(@PathParam("name") String name) {
    return SimpleStorageService.findByName(name);
}
```

请注意，我们如何使用相同的`@Valid`注释。**但是这次我们在资源方法级别使用它来确保响应是有效的。**

## 6。自定义异常处理程序

在这最后一部分中，我们将简要地看一下如何创建一个定制的异常处理程序。如果我们违反了一个特定的约束，当我们想要返回一个定制的响应时，这是很有用的。

让我们从定义我们的`FruitExceptionMapper`开始:

```
public class FruitExceptionMapper implements ExceptionMapper<ConstraintViolationException> {

    @Override
    public Response toResponse(ConstraintViolationException exception) {
        return Response.status(Response.Status.BAD_REQUEST)
            .entity(prepareMessage(exception))
            .type("text/plain")
            .build();
    }

    private String prepareMessage(ConstraintViolationException exception) {
        StringBuilder message = new StringBuilder();
        for (ConstraintViolation<?> cv : exception.getConstraintViolations()) {
            message.append(cv.getPropertyPath() + " " + cv.getMessage() + "\n");
        }
        return message.toString();
    }
}
```

首先，我们定义一个定制的异常映射提供者。为了做到这一点，我们使用一个`ConstraintViolationException`实现了`ExceptionMapper`接口。

**因此，我们将看到，当这个异常 被抛出 时，我们的自定义异常映射器实例的`toResponse`方法将被调用。**

此外，在这个简单的例子中，我们遍历所有违规，并在响应中追加要发送回的每个属性和消息。

**接下来，为了使用我们的定制异常映射器，我们需要注册我们的提供者**:

```
@Override
protected Application configure() {
    ViewApplicationConfig config = new ViewApplicationConfig();
    config.register(FruitExceptionMapper.class);
    return config;
}
```

最后，我们添加一个端点来返回一个无效的`Fruit` ,以显示运行中的异常处理器:

```
@GET
@Produces(MediaType.TEXT_HTML)
@Path("/exception")
@Valid
public Fruit exception() {
    Fruit fruit = new Fruit();
    fruit.setName("a");
    fruit.setColour("b");
    return fruit;
} 
```

## 7。结论

总之，在本教程中，我们已经探索了 Jersey Bean 验证 API 扩展。

首先，我们从介绍如何在 Jersey 中使用 Bean 验证 API 开始。此外，我们还了解了如何配置一个示例 web 应用程序。

最后，我们看了几种用 Jersey 进行验证的方法，以及如何编写一个定制的异常处理程序。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220627172045/https://github.com/eugenp/tutorials/tree/master/jersey)