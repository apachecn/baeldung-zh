# Spring MVC 中使用 FreeMarker 的介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/freemarker-in-spring-mvc-tutorial>

## 1。概述

FreeMarker 是一个基于 Java 的模板引擎，来自 Apache 软件基金会。像其他模板引擎一样，FreeMarker 被设计成在遵循 MVC 模式的应用程序中支持 HTML 网页。本教程展示了如何**配置 FreeMarker 以用于 Spring MVC** 作为 JSP 的替代。

本文不会讨论 Spring MVC 使用的基础。要深入了解这一点，请参考本文。此外，这并不是要详细了解 FreeMarker 的广泛功能。有关 FreeMarker 用法和语法的更多信息，请访问[的网站](https://web.archive.org/web/20220812065141/https://freemarker.incubator.apache.org/)。

## 2。Maven 依赖关系

由于这是一个基于 Maven 的项目，我们首先将所需的依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.23</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>${spring.version}</version>
</dependency>
```

## 3。配置

现在让我们深入了解项目的配置。这是一个基于注释的 Spring 项目，所以我们不会演示基于 XML 的配置。

### 3.1。弹簧腹板配置

让我们创建一个类来配置 web 组件。为此，我们需要用`@EnableWebMvc`、`@Configuration`和`@ComponentScan`来注释这个类。

```java
@EnableWebMvc
@Configuration
@ComponentScan({"com.baeldung.freemarker"})
public class SpringWebConfig extends WebMvcConfigurerAdapter {
    // All web configuration will go here.
}
```

### 3.2。配置`ViewResolver`

Spring MVC 框架提供了 **`ViewResolver`** 接口，将视图名称映射到实际的视图。我们将创建一个`**FreeMarkerViewResolver**`的实例，它属于 **spring-webmvc** 依赖关系。

该对象需要用将在运行时使用的必需值进行配置。例如，我们将配置视图解析器对以`**.ftl**`结尾的视图使用 FreeMarker:

```java
@Bean 
public FreeMarkerViewResolver freemarkerViewResolver() { 
    FreeMarkerViewResolver resolver = new FreeMarkerViewResolver(); 
    resolver.setCache(true); 
    resolver.setPrefix(""); 
    resolver.setSuffix(".ftl"); 
    return resolver; 
}
```

此外，请注意，我们还可以在这里控制缓存模式——这应该只在调试和开发时禁用。

### 3.3。FreeMarker 模板路径配置

接下来，我们将设置模板路径，该路径指示模板在 web 上下文中的位置:

```java
@Bean 
public FreeMarkerConfigurer freemarkerConfig() { 
    FreeMarkerConfigurer freeMarkerConfigurer = new FreeMarkerConfigurer(); 
    freeMarkerConfigurer.setTemplateLoaderPath("/WEB-INF/views/ftl/");
    return freeMarkerConfigurer; 
}
```

### 3.4。弹簧控制器配置

现在我们可以使用一个 Spring 控制器**来处理一个 FreeMarker 模板用于显示**。这只是一个传统的弹簧控制器:

```java
@RequestMapping(value = "/cars", method = RequestMethod.GET)
public String init(@ModelAttribute("model") ModelMap model) {
    model.addAttribute("carList", carList);
    return "index";
}
```

先前定义的`FreeMarkerViewResolver`和路径配置将负责将视图名称`index`转换成正确的 FreeMarker 视图。

## 4。FreeMarker HTML 模板

### 4.1。创建简单的 HTML 模板视图

现在是时候用 FreeMarker 创建一个 **HTML 模板了。在我们的示例中，我们向模型添加了一个汽车列表。FreeMarker 可以访问这个列表，并通过遍历它的内容来显示它。**

当对`/cars` URI 发出请求时，Spring 将使用提供给它的模型来处理模板。在我们的模板中， **`#list`指令**指示 FreeMarker 应该遍历模型中的`carList`对象，使用`car`引用当前元素并呈现该块中的内容。

下面的代码还包括**FreeMarker**表达式来引用`carList`中每个元素的属性；例如，为了显示当前汽车元素的`make`属性，我们使用了表达式`${car.make}`。

```java
<div id="header">
  <h2>FreeMarker Spring MVC Hello World</h2>
</div>
<div id="content">
  <fieldset>
    <legend>Add Car</legend>
    <form name="car" action="add" method="post">
      Make : <input type="text" name="make" /><br/>
      Model: <input type="text" name="model" /><br/>
      <input type="submit" value="Save" />
    </form>
  </fieldset>
  <br/>
  <table class="datatable">
    <tr>
      <th>Make</th>
      <th>Model</th>
    </tr>
    <#list model["carList"] as car>
      <tr>
        <td>${car.make}</td>
        <td>${car.model}</td>
      </tr>
    </#list>
  </table>
</div>
```

用 CSS 对输出进行样式化后，处理后的 FreeMarker 模板生成一个表单和汽车列表:

[![browser_localhost-300x235](img/8e6a67d535c1e8bea8527c38797a5805.png)](/web/20220812065141/https://www.baeldung.com/wp-content/uploads/2016/07/browser_localhost-300x235-1.png)

## 5.Spring Boot

如果我们使用 Spring Boot，我们可以简单地导入`spring-boot-starter-freemarker`依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
    <version>2.3.4.RELEASE</version>
</dependency>
```

然后，我们只需在`src/main/resources/templates`下添加模板文件。Spring Boot 负责其他默认配置，如`FreeMarkerConfigurer`和`FreeMarkerViewResolver`。

## 6。结论

在本文中，我们讨论了如何在 Spring MVC 应用程序中集成 **FreeMarker。** FreeMarker 的功能远远超出了我们所展示的范围，因此请访问 [Apache FreeMarker 网站](https://web.archive.org/web/20220812065141/https://freemarker.incubator.apache.org/)了解更多关于其使用的详细信息。

本文中的示例代码可以在 [Github](https://web.archive.org/web/20220812065141/https://github.com/eugenp/tutorials/tree/master/spring-freemarker) 上的一个项目中获得。