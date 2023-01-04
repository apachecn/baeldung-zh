# 属性文件中的@RequestMapping 值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-requestmapping-properties-file>

## 1.概观

在本教程中，**我们将看看如何在属性文件中设置`[@RequestMapping](/web/20221128043133/https://www.baeldung.com/spring-requestmapping)`值。**此外，我们将使用一个实际的例子来解释所有必要的配置。

首先，我们来定义一个基本的`@RequestMapping`及其配置。

## 2.@ `RequestMapping`基础知识

首先，**我们将用`@RequestMapping`创建并注释我们的类`WelcomeController`，以映射 web 请求**。这个类将分配我们的处理方法`getWelcomeMessage().`

所以，我们来定义一下:

```java
@RestController
@RequestMapping("/welcome")
public class WelcomeController {

   @GetMapping
   public String getWelcomeMessage() {
       return "Welcome to Baeldung!";
   }
}
```

另外，**有趣的是，我们将用`@GetMapping `来注释`getWelcomeMessage` `() `，以便只映射 GET 请求。**正如我们所看到的，我们使用了一个硬编码的字符串作为路径，静态地指示我们想要访问的路径。通过这种配置，我们可以很好地访问我们感兴趣的资源，如下所示:

```java
curl http://localhost:9006/welcome
> Welcome to Baeldung!
```

但是，如果我们想让路径依赖于一个配置参数呢？正如我们接下来将要看到的，我们可以利用`application.properties`。

## 3.@ `RequestMapping`和属性文件

首先，正如我们在[文档](https://web.archive.org/web/20221128043133/https://docs.spring.io/spring-framework/docs/3.2.16.RELEASE/spring-framework-reference/html/mvc.html)中所看到的，`@RequestMapping` 注释中的**模式支持${…}占位符来对应本地属性和/或系统属性以及环境变量**。这样，我们可以将属性文件链接到控制器。

一方面，我们需要创建属性文件本身。**我们将它放在`resources`文件夹中，命名为`application.properties`** 。然后，我们必须用我们选择的名称创建属性。在我们的例子中，我们将设置名称`welcome-controller.path`，并将我们想要的值设置为请求的端点。现在，我们的`application.properties`看起来像这样:

```java
welcome-controller.path=welcome
```

另一方面，**我们必须修改我们已经在`@RequestMapping`中静态建立的路径，以便它读取我们已经创建的新属性**:

```java
@RestController
@RequestMapping("/${welcome-controller.path}")
public class WelcomeController {
    @GetMapping
    public String getWelcomeMessage() {
        return "Welcome to Baeldung!";
    }
}
```

所以，通过这种方式，Spring 将能够映射端点的值，当用户访问这个 URL 时，这个方法将负责处理它。我们可以在下面看到相同的消息是如何与相同的请求一起显示的:

```java
curl http://localhost:9006/welcome 
> Welcome to Baeldung!
```

## 4.结论

在这篇短文中，**我们学习了如何在属性文件**中设置`@RequestMapping`值。此外，我们还创建了一个全功能示例，帮助我们理解所解释的概念。

GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221128043133/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-5)