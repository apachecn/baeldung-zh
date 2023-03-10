# 春季使用百里香叶简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-in-spring-mvc>

## 1。概述

[百里叶](https://web.archive.org/web/20221006104233/http://www.thymeleaf.org/)是一个 Java 模板引擎，用于处理和创建 HTML、XML、JavaScript、CSS 和文本。

在本教程中，我们将讨论如何将百里香与 Spring 一起使用，以及 Spring MVC 应用程序的视图层中的一些基本用例。

该库具有极强的可扩展性，其自然的模板功能确保我们可以在没有后端的情况下构建模板原型。与 JSP 等其他流行的模板引擎相比，这使得开发速度非常快。

## 2。百里香与春天的结合

首先，让我们看看与 Spring 集成所需的配置。集成需要`thymeleaf-spring`库。

我们将向 Maven POM 文件添加以下依赖项:

```java
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```

注意，对于 Spring 4 项目，我们必须使用`thymeleaf-spring4`库而不是`thymeleaf-spring5`。

`SpringTemplateEngine`类执行所有的配置步骤。

我们可以在 Java 配置文件中将该类配置为 bean:

```java
@Bean
@Description("Thymeleaf Template Resolver")
public ServletContextTemplateResolver templateResolver() {
    ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver();
    templateResolver.setPrefix("/WEB-INF/views/");
    templateResolver.setSuffix(".html");
    templateResolver.setTemplateMode("HTML5");

    return templateResolver;
}

@Bean
@Description("Thymeleaf Template Engine")
public SpringTemplateEngine templateEngine() {
    SpringTemplateEngine templateEngine = new SpringTemplateEngine();
    templateEngine.setTemplateResolver(templateResolver());
    templateEngine.setTemplateEngineMessageSource(messageSource());
    return templateEngine;
}
```

`templateResolver` bean 属性`prefix`和`suffix`分别表示视图页面在`webapp`目录中的位置和它们的文件扩展名。

Spring MVC 中的`ViewResolver`接口将控制器返回的视图名称映射到实际的视图对象。`ThymeleafViewResolver`实现了`ViewResolver`接口，它用于在给定视图名称的情况下决定渲染哪些百里香视图。

集成的最后一步是将`ThymeleafViewResolver`作为 bean 添加:

```java
@Bean
@Description("Thymeleaf View Resolver")
public ThymeleafViewResolver viewResolver() {
    ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
    viewResolver.setTemplateEngine(templateEngine());
    viewResolver.setOrder(1);
    return viewResolver;
}
```

## 3.**Spring Boot 百里香叶**

`Spring Boot`通过添加 [`spring-boot-starter-thymeleaf`](https://web.archive.org/web/20221006104233/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-thymeleaf%22) 依赖关系为`Thymeleaf`提供自动配置:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <version>2.3.3.RELEASE</version>
</dependency> 
```

不需要显式配置。默认情况下，HTML 文件应该放在`resources/templates `位置。

## 4。显示来自消息源(属性文件)的值

我们可以使用`th:text=”#{key}”`标签属性来显示属性文件中的值。

为此，我们需要将属性文件配置为一个`messageSource` bean:

```java
@Bean
@Description("Spring Message Resolver")
public ResourceBundleMessageSource messageSource() {
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    messageSource.setBasename("messages");
    return messageSource;
}
```

下面是显示与键`welcome.message`相关的值的百里香 HTML 代码:

```java
<span th:text="#{welcome.message}" />
```

## 5。显示模型属性

### 5.1。简单属性

我们可以使用`th:text=”${attributename}”`标签属性来显示模型属性的值。

让我们在控制器类中添加一个名为`serverTime`的模型属性:

```java
model.addAttribute("serverTime", dateFormat.format(new Date()));
```

下面是显示`serverTime`属性的值的 HTML 代码:

```java
Current time is <span th:text="${serverTime}" />
```

### 5.2。集合属性

如果模型属性是对象的集合，我们可以使用`th:each`标签属性来迭代它。

让我们定义一个具有两个字段`id`和`name`的`Student`模型类:

```java
public class Student implements Serializable {
    private Integer id;
    private String name;
    // standard getters and setters
}
```

现在我们将在控制器类中添加一个学生列表作为模型属性:

```java
List<Student> students = new ArrayList<Student>();
// logic to build student data
model.addAttribute("students", students);
```

最后，我们可以使用百里香模板代码迭代学生列表并显示所有字段值:

```java
<tbody>
    <tr th:each="student: ${students}">
        <td th:text="${student.id}" />
        <td th:text="${student.name}" />
    </tr>
</tbody>
```

## 6。条件评估

### 6.1。`if`和`unless`

如果满足条件，我们使用`th:if=”${condition}”`属性来显示视图的一部分。如果条件不满足，我们使用`th:unless=”${condition}”`属性来显示视图的一部分。

让我们给`Student`模型添加一个`gender`字段:

```java
public class Student implements Serializable {
    private Integer id;
    private String name;
    private Character gender;

    // standard getters and setters
}
```

假设这个字段有两个可能的值(M 或 F)来表示学生的性别。

如果我们希望显示单词“男性”或“女性”而不是单个字符，我们可以使用下面的百里香代码:

```java
<td>
    <span th:if="${student.gender} == 'M'" th:text="Male" /> 
    <span th:unless="${student.gender} == 'M'" th:text="Female" />
</td>
```

### 6.2。`switch` 和`case`

我们使用`th:switch`和`th:case`属性，通过 switch 语句结构有条件地显示内容。

让我们使用`th:switch`和`th:case`属性重写前面的代码:

```java
<td th:switch="${student.gender}">
    <span th:case="'M'" th:text="Male" /> 
    <span th:case="'F'" th:text="Female" />
</td>
```

## 7。处理用户输入

我们可以使用`th:action=”@{url}”`和`th:object=”${object}”`属性来处理表单输入。我们使用`th:action`来提供表单动作 URL，使用`th:object`来指定提交的表单数据将绑定到的对象。

使用`th:field=”*{name}”`属性映射各个字段，其中`name`是对象的匹配属性。

对于`Student`类，我们可以创建一个输入表单:

```java
<form action="#" th:action="@{/saveStudent}" th:object="${student}" method="post">
    <table border="1">
        <tr>
            <td><label th:text="#{msg.id}" /></td>
            <td><input type="number" th:field="*{id}" /></td>
        </tr>
        <tr>
            <td><label th:text="#{msg.name}" /></td>
            <td><input type="text" th:field="*{name}" /></td>
        </tr>
        <tr>
            <td><input type="submit" value="Submit" /></td>
        </tr>
    </table>
</form>
```

在上面的代码中，`/saveStudent`是表单动作 URL，`student`是保存提交的表单数据的对象。

`StudentController`类处理表单提交:

```java
@Controller
public class StudentController {
    @RequestMapping(value = "/saveStudent", method = RequestMethod.POST)
    public String saveStudent(@ModelAttribute Student student, BindingResult errors, Model model) {
        // logic to process input data
    }
}
```

`@RequestMapping`注释用表单中提供的 URL 映射控制器方法。带注释的方法`saveStudent()`为提交的表单执行所需的处理。最后，`@ModelAttribute`注释将表单字段绑定到`student`对象。

## 8。显示验证错误

我们可以使用`#fields.hasErrors()`函数来检查一个字段是否有任何验证错误。我们使用`#fields.errors()`函数来显示特定字段的错误。字段名是这两个函数的输入参数。

让我们看一下 HTML 代码来迭代并显示表单中每个字段的错误:

```java
<ul>
    <li th:each="err : ${#fields.errors('id')}" th:text="${err}" />
    <li th:each="err : ${#fields.errors('name')}" th:text="${err}" />
</ul>
```

上述函数接受通配符`*`或常量`all`来表示所有字段，而不是字段名。我们使用了`th:each`属性来迭代每个字段可能出现的多个错误。

下面是使用通配符`*`重写的前一段 HTML 代码:

```java
<ul>
    <li th:each="err : ${#fields.errors('*')}" th:text="${err}" />
</ul>
```

这里我们使用常数`all`:

```java
<ul>
    <li th:each="err : ${#fields.errors('all')}" th:text="${err}" />
</ul>
```

类似地，我们可以使用`global`常量显示 Spring 中的全局错误。

下面是显示全局错误的 HTML 代码:

```java
<ul>
    <li th:each="err : ${#fields.errors('global')}" th:text="${err}" />
</ul>
```

此外，我们可以使用`th:errors`属性来显示错误消息。

前面显示表单错误的代码可以使用`th:errors`属性重写:

```java
<ul>
    <li th:errors="*{id}" />
    <li th:errors="*{name}" />
</ul>
```

## 9。使用转换

我们使用双括号语法`{{}}`来格式化数据以便显示。这利用了在上下文文件的`conversionService` bean 中为该类型字段配置的`formatters`。

让我们看看如何格式化`Student`类中的 name 字段:

```java
<tr th:each="student: ${students}">
    <td th:text="${{student.name}}" />
</tr>
```

上面的代码使用了`NameFormatter`类，通过从`WebMvcConfigurer`接口覆盖`addFormatters()`方法来配置。

为此，我们的`@Configuration`类覆盖了`WebMvcConfigurerAdapter`类:

```java
@Configuration
public class WebMVCConfig extends WebMvcConfigurerAdapter {
    // ...
    @Override
    @Description("Custom Conversion Service")
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new NameFormatter());
    }
}
```

`NameFormatter`类实现了 Spring `Formatter`接口。

我们还可以使用 `#conversions` 实用程序来转换显示对象。效用函数的语法是`#conversions.convert(Object, Class)`，其中`Object`被转换为`Class`类型。

下面是如何显示去掉小数部分的`student`对象`percentage`字段:

```java
<tr th:each="student: ${students}">
    <td th:text="${#conversions.convert(student.percentage, 'Integer')}" />
</tr>
```

## 10。结论

在本文中，我们看到了如何在 Spring MVC 应用程序中集成和使用百里香。

我们还看到了如何显示字段、接受输入、显示验证错误以及转换数据以便显示的示例。

本文中显示的代码的工作版本可以在 [GitHub 库](https://web.archive.org/web/20221006104233/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf)中找到。