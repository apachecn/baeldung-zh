# Apache Velocity 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-velocity>

## 1。概述

[Velocity](https://web.archive.org/web/20220909001729/https://velocity.apache.org/) 是一个基于 Java 的模板引擎。

它是一个开源的 web 框架，旨在用作 MVC 架构中的视图组件，并且它提供了一些现有技术(如 JSP)的替代方案。

Velocity 可用于生成 XML 文件、SQL、PostScript 和大多数其他基于文本的格式。

在本文中，我们将探讨如何使用它来创建动态网页。

## 2。速度如何工作

**速度的核心类是`VelocityEngine`。**

它使用数据模型和 velocity 模板协调读取、解析和生成内容的整个过程。

简而言之，对于任何典型的 velocity 应用程序，我们都需要遵循以下步骤:

*   初始化速度引擎
*   阅读模板
*   将数据模型放入上下文对象中
*   将模板与上下文数据合并，并呈现视图

**让我们按照这些简单的步骤来看一个例子**:

```
VelocityEngine velocityEngine = new VelocityEngine();
velocityEngine.init();

Template t = velocityEngine.getTemplate("index.vm");

VelocityContext context = new VelocityContext();
context.put("name", "World");

StringWriter writer = new StringWriter();
t.merge( context, writer );
```

## 3。Maven 依赖关系

为了使用 Velocity，我们需要向我们的 Maven 项目添加以下依赖项:

```
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity</artifactId>
    <version>1.7</version>
    </dependency>
<dependency>
     <groupId>org.apache.velocity</groupId>
     <artifactId>velocity-tools</artifactId>
     <version>2.0</version>
</dependency>
```

这两个依赖项的最新版本可以在这里找到: [velocity](https://web.archive.org/web/20220909001729/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.velocity%22%20AND%20a%3A%22velocity%22) 和 [velocity-tools](https://web.archive.org/web/20220909001729/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.velocity%22%20AND%20a%3A%22velocity-tools%22) 。

## 4。速度模板语言

Velocity 模板语言(VTL)通过使用 VTL 引用，提供了将动态内容合并到网页中的最简单、最干净的方法。

速度模板中的 VTL 参考以`$` 开始，用于获取与该参考相关的值。VTL 还提供了一组指令，可以用来操作 Java 代码的输出。这些指令以`#.` 开始

### 4.1。参考文献

速度、变量、属性和方法中有三种类型的引用:

*   **变量**–使用`#set`指令或从 Java 对象的字段返回的值在页面内定义:

    ```
    #set ($message="Hello World")
    ```

*   **属性**–指对象内的字段；它们也可以引用属性

    ```
    $customer.name
    ```

    的`getter`方法
*   **方法**–指 Java 对象上的方法:

    ```
    $customer.getName()
    ```

每个引用产生的最终值在呈现到最终输出时被转换为字符串。

### 4.2。指令

VTL 提供了一组丰富的指令:

*   **设置**–可用于设置参考值；该值可以赋给变量或属性引用:

    ```
    #set ($message = "Hello World")
    #set ($customer.name = "Brian Mcdonald")
    ```

*   **条件**–`#if, #elseif` 和 `#else` 指令提供了一种基于条件检查生成内容的方法:

    ```
    #if($employee.designation == "Manager")
        <h3> Manager </h3>
    #elseif($employee.designation == "Senior Developer")
        <h3> Senior Software Engineer </h3>
    #else
        <h3> Trainee </h3>
    #end
    ```

*   **循环**–`#foreach` 指令允许循环对象集合:

    ```
    <ul>
        #foreach($product in $productList)
            <li> $product </li>
        #end
    </ul>
    ```

*   **包含**–`#include` 元素提供将文件导入模板的能力:

    ```
    #include("one.gif","two.txt","three.html"...)
    ```

*   **解析**–`#parse`语句允许模板设计者导入另一个包含 VTL 的本地文件；Velocity 将解析内容并呈现它:

    ```
    #parse (Template)
    ```

*   **评估**–`#evaluate`指令可用于动态评估 VTL；这允许模板在渲染时评估一个`String`，例如国际化模板:

    ```
    #set($firstName = "David")
    #set($lastName = "Johnson")

    #set($dynamicsource = "$firstName$lastName")

    #evaluate($dynamicsource)
    ```

*   **break**–`#break` 指令停止对当前执行范围(即`#foreach`、`#parse`)的任何进一步渲染
*   **停止**–`#stop`指令停止模板的任何进一步渲染和执行。
*   **velocimacros** – `#macro` directive allows the template designer to define a repeated segment of VTL:

    ```
    #macro(tablerows)
        <tr>
            <td>
            </td>
        </tr>
    #end
    ```

    这个宏现在可以作为# `tablerows():`放在模板的任何地方

    ```
    #macro(tablerows $color $productList)
        #foreach($product in $productList)
            <tr>
                <td bgcolor=$color>$product.name</td>
            </tr>
        #end
    #end
    ```

### 4.3。其他功能

*   **数学**–一些内置的数学函数，可以在模板中使用:

    ```
    #set($percent = $number / 100)
    #set($remainder = $dividend % $divisor)
    ```

*   **测距算子**——可与`#set`和`#foreach:`

    ```
    #set($array = [0..10])

    #foreach($elem in $arr)
        $elem
    #end
    ```

    配合使用

## 5。速度 Servlet

Velocity 引擎的主要工作是基于模板生成内容。

该引擎本身不包含任何与 web 相关的功能。要实现 web 应用程序，我们需要使用 servlet 或基于 servlet 的框架。

Velocity 提供了一个现成的实现`VelocityViewServlet`，它是 velocity-tools 子项目的一部分。

为了利用`VelocityViewServlet,` 提供的内置功能，我们可以从`VelocityViewServlet` 扩展我们的 servlet 并覆盖`handleRequest()` 方法:

```
public class ProductServlet extends VelocityViewServlet {

    ProductService service = new ProductService();

    @Override
    public Template handleRequest(
      HttpServletRequest request, 
      HttpServletResponse response,
      Context context) throws Exception {

        List<Product> products = service.getProducts();
        context.put("products", products);

        return getTemplate("index.vm");
    }
}
```

## 6。配置

### 6.1。网络配置

现在让我们看看如何在`web.xml`中配置`VelocityViewServlet`。

我们需要指定可选的初始化参数，包括`velocity.properties`和`toolbox.xml`:

```
<web-app>
    <display-name>apache-velocity</display-name>
      //...

    <servlet>
        <servlet-name>velocity</servlet-name>
        <servlet-class>org.apache.velocity.tools.view.VelocityViewServlet</servlet-class>

        <init-param>
            <param-name>org.apache.velocity.properties</param-name>
            <param-value>/WEB-INF/velocity.properties</param-value>
        </init-param>
    </servlet>
        //...
</web-app> 
```

我们还需要为这个 servlet 指定映射。所有对 velocity 模板(`*.vm`)的请求都需要由 velocity servlet 提供服务:

```
<servlet-mapping>
    <servlet-name>velocityLayout</servlet-name>
    <url-pattern>*.vm</url-pattern>
</servlet-mapping>
```

### 6.2。资源加载器

Velocity 提供灵活的资源加载器系统。它允许一个或多个资源加载器同时运行:

*   `FileResourceLoader`
*   `JarResourceLoader`
*   `ClassPathResourceLoader`
*   `URLResourceLoader`
*   `DataSourceResourceLoader`
*   `WebappResourceLoader`

这些资源加载器在`velocity.properties:`中配置

```
resource.loader=webapp
webapp.resource.loader.class=org.apache.velocity.tools.view.WebappResourceLoader
webapp.resource.loader.path = 
webapp.resource.loader.cache = true
```

## 7。速度模板

Velocity 模板是编写所有视图生成逻辑的地方。这些页面使用 Velocity 模板语言(VTL)编写:

```
<html>
    ...
    <body>
        <center>
        ...
        <h2>$products.size() Products on Sale!</h2>
        <br/>
            We are proud to offer these fine products
            at these amazing prices.
        ...
        #set( $count = 1 )
        <table class="gridtable">
            <tr>
                <th>Serial #</th>
                <th>Product Name</th>
                <th>Price</th>
            </tr>
            #foreach( $product in $products )
            <tr>
                <td>$count)</td>
                <td>$product.getName()</td>
                <td>$product.getPrice()</td>
            </tr>
            #set( $count = $count + 1 )
            #end
        </table>
        <br/>
        </center>
    </body>
</html>
```

## 8。管理页面布局

Velocity 为基于 Velocity 工具的应用程序提供了简单的布局控件和可定制的错误屏幕。

`VelocityLayoutServlet` 封装了这个功能来呈现指定的布局。`VelocityLayoutServlet`是对`VelocityViewServlet.` 的扩展

### 8.1。网络配置

让我们看看如何配置`VelocityLayoutServlet.` ，servlet 被定义为拦截对 velocity 模板页面的请求，布局特定的属性在`velocity.properties`文件中定义:

```
<web-app>
    // ...
    <servlet>
        <servlet-name>velocityLayout</servlet-name>
        <servlet-class>org.apache.velocity.tools.view.VelocityLayoutServlet</servlet-class>

        <init-param>
            <param-name>org.apache.velocity.properties</param-name>
            <param-value>/WEB-INF/velocity.properties</param-value>
        </init-param>
    </servlet>
    // ...
    <servlet-mapping>
        <servlet-name>velocityLayout</servlet-name>
        <url-pattern>*.vm</url-pattern>
    </servlet-mapping>
    // ...
</web-app>
```

### 8.2。布局模板

布局模板定义了 velocity 页面的典型结构。默认情况下，`VelocityLayoutServlet`在布局文件夹下搜索`Default.vm` 。覆盖几个属性可以更改此位置:

```
tools.view.servlet.layout.directory = layout/
tools.view.servlet.layout.default.template = Default.vm 
```

布局文件由页眉模板、页脚模板和速度变量`$screen_content` 组成，该变量呈现所请求的速度页面的内容:

```
<html>
    <head>
        <title>Velocity</title>
    </head>
    <body>
        <div>
            #parse("/fragments/header.vm")
        </div>
        <div>
            <!-- View index.vm is inserted here -->
            $screen_content
        </div>
        <div>
            #parse("/fragments/footer.vm")
        </div>
    </body>
</html>
```

### 8.3。请求屏幕中的布局规格

特定屏幕的布局可以定义为页面开头的速度变量。这是通过在页面中添加这一行来实现的:

```
#set($layout = "MyOtherLayout.vm")
```

### 8.4。请求参数中的布局规格

我们可以在查询字符串`layout=MyOtherLayout.vm`中添加一个请求参数，VLS 将找到它并在该布局中呈现屏幕，而不是搜索默认布局。

### 8.5。错误屏幕

可以使用 velocity 布局实现定制的错误屏幕。`VelocityLayoutServlet`提供两个变量`$error_cause`和`$stack_trace`来表示异常细节。

可以在`velocity.properties` 文件中配置错误页面:

```
tools.view.servlet.error.template = Error.vm
```

## 9。结论

在本文中，我们已经了解了 Velocity 是如何成为渲染动态网页的有用工具。此外，我们已经看到了使用 velocity 提供的 servlets 的不同方式。

在 Baeldung 我们也有一篇关于 Spring MVC [的速度配置的文章。](/web/20220909001729/https://www.baeldung.com/spring-mvc-with-velocity)

本教程的完整代码可从 GitHub 上的[处获得。](https://web.archive.org/web/20220909001729/https://github.com/eugenp/tutorials/tree/master/apache-velocity)