# Spring MVC +百里香 3.0:新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-thymeleaf-3>

## 1。简介

[百里叶](https://web.archive.org/web/20220810180959/http://www.thymeleaf.org/)是一个 Java 模板引擎，用于处理和创建 HTML、XML、JavaScript、CSS 和纯文本。关于百里香和春天的介绍，请看[这篇文章](/web/20220810180959/https://www.baeldung.com/thymeleaf-in-spring-mvc)。

在本文中，我们将讨论百里叶应用程序 Spring MVC 中百里叶 3.0 的新特性。版本 3 提供了新的特性和许多改进。更具体地说，我们将讨论自然处理和 Javascript 内联的主题。

百里叶 3.0 包括三种新的文本模板模式:`TEXT`、`JAVASCRIPT`和`CSS` ——分别用于处理普通、JavaScript 和 CSS 模板。

## 2。Maven 依赖关系

首先，让我们看看将百里香叶与 Spring 集成所需的配置；`thymeleaf-spring`我们的依赖项中需要库:

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

注意，对于 Spring 4 项目，必须使用`thymeleaf-spring4`库来代替`thymeleaf-spring5`。最新版本的依赖可以在[这里](https://web.archive.org/web/20220810180959/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.thymeleaf%22%20AND%20a%3A%22thymeleaf-spring5%22)找到。

## 3。Java 百里香配置

首先，我们需要配置新的模板引擎、视图和模板解析器。为了做到这一点，我们需要更新 Java 配置类

为了做到这一点，我们需要更新 Java config 类，这里创建了[和](/web/20220810180959/https://www.baeldung.com/thymeleaf-in-spring-mvc)。除了新类型的解析器，我们的模板还实现了 Spring 接口`ApplicationContextAware`:

```java
@Configuration
@EnableWebMvc
@ComponentScan({ "com.baeldung.thymeleaf" })
public class WebMVCConfig implements WebMvcConfigurer, ApplicationContextAware {

    private ApplicationContext applicationContext;

    // Java setter

    @Bean
    public ViewResolver htmlViewResolver() {
        ThymeleafViewResolver resolver = new ThymeleafViewResolver();
        resolver.setTemplateEngine(templateEngine(htmlTemplateResolver()));
        resolver.setContentType("text/html");
        resolver.setCharacterEncoding("UTF-8");
        resolver.setViewNames(ArrayUtil.array("*.html"));
        return resolver;
    }

    @Bean
    public ViewResolver javascriptViewResolver() {
        ThymeleafViewResolver resolver = new ThymeleafViewResolver();
        resolver.setTemplateEngine(templateEngine(javascriptTemplateResolver()));
        resolver.setContentType("application/javascript");
        resolver.setCharacterEncoding("UTF-8");
        resolver.setViewNames(ArrayUtil.array("*.js"));
        return resolver;
    }

    @Bean
    public ViewResolver plainViewResolver() {
        ThymeleafViewResolver resolver = new ThymeleafViewResolver();
        resolver.setTemplateEngine(templateEngine(plainTemplateResolver()));
        resolver.setContentType("text/plain");
        resolver.setCharacterEncoding("UTF-8");
        resolver.setViewNames(ArrayUtil.array("*.txt"));
        return resolver;
    }
} 
```

正如我们在上面看到的，我们创建了三个不同的视图解析器——一个用于 HTML 视图，一个用于 Javascript 文件，一个用于纯文本文件。百里香将通过检查文件扩展名来区分它们——分别是`.html`、`.js`和`.txt`。

我们还创建了静态的`ArrayUtil`类，以便使用方法`array()`创建所需的带有视图名称的`String[]`数组。

在本课程的下一部分，我们需要配置模板引擎:

```java
private ISpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
    SpringTemplateEngine engine = new SpringTemplateEngine();
    engine.setTemplateResolver(templateResolver);
    return engine;
} 
```

最后，我们需要创建三个独立的模板解析器:

```java
private ITemplateResolver htmlTemplateResolver() {
    SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
    resolver.setApplicationContext(applicationContext);
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setCacheable(false);
    resolver.setTemplateMode(TemplateMode.HTML);
    return resolver;
}

private ITemplateResolver javascriptTemplateResolver() {
    SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
    resolver.setApplicationContext(applicationContext);
    resolver.setPrefix("/WEB-INF/js/");
    resolver.setCacheable(false);
    resolver.setTemplateMode(TemplateMode.JAVASCRIPT);
    return resolver;
}

private ITemplateResolver plainTemplateResolver() {
    SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
    resolver.setApplicationContext(applicationContext);
    resolver.setPrefix("/WEB-INF/txt/");
    resolver.setCacheable(false);
    resolver.setTemplateMode(TemplateMode.TEXT);
    return resolver;
}
```

请注意，测试最好使用非缓存模板，这就是为什么建议使用`setCacheable(false)`方法。

Javascript 模板会存放在`/WEB-INF/js/`文件夹，纯文本文件存放在`/WEB-INF/txt/`文件夹，最后 HTML 文件的路径是`/WEB-INF/html`。

## 4。弹簧控制器配置

为了测试我们的新配置，我们创建了以下 Spring 控制器:

```java
@Controller
public class InliningController {

    @RequestMapping(value = "/html", method = RequestMethod.GET)
    public String getExampleHTML(Model model) {
        model.addAttribute("title", "Baeldung");
        model.addAttribute("description", "<strong>Thymeleaf</strong> tutorial");
        return "inliningExample.html";
    }

    @RequestMapping(value = "/js", method = RequestMethod.GET)
    public String getExampleJS(Model model) {
        model.addAttribute("students", StudentUtils.buildStudents());
        return "studentCheck.js";
    }

    @RequestMapping(value = "/plain", method = RequestMethod.GET)
    public String getExamplePlain(Model model) {
        model.addAttribute("username", SecurityContextHolder.getContext()
          .getAuthentication().getName());
        model.addAttribute("students", StudentUtils.buildStudents());
        return "studentsList.txt";
    }
}
```

在 HTML 文件示例中，我们将展示如何使用新的内联特性，包括转义 HTML 标签和不转义 HTML 标签。

对于 js 示例，我们将生成一个 AJAX 请求，该请求将加载包含学生信息的 JS 文件。请注意，我们在`StudentUtils`类中使用简单的`buildStudents()`方法，来自这篇[文章](/web/20220810180959/https://www.baeldung.com/thymeleaf-in-spring-mvc)。

最后，在纯文本示例中，我们将学生信息显示为文本文件。使用纯文本模板模式的典型示例可以用于发送纯文本电子邮件。

作为附加功能，我们将使用`SecurityContextHolder`来获取登录的用户名。

## 5。Html/Js/Text 示例文件

本教程的最后一部分是创建三种不同类型的文件，并测试新的百里香叶功能的用法。让我们从 HTML 文件开始:

```java
<!DOCTYPE html>
<html  xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Inlining example</title>
</head>
<body>
    <p>Title of tutorial: [[${title}]]</p>
    <p>Description: [(${description})]</p>
</body>
</html>
```

在这个文件中，我们使用了两种不同的方法。为了显示标题，我们使用转义语法，这将删除所有 HTML 标签，导致只显示文本。在描述的情况下，我们使用非转义语法来保存 HTML 标签。最终结果将如下所示:

```java
<p>Title of tutorial: Baeldung</p>
<p>Description: <strong>Thymeleaf</strong> tutorial</p>
```

这当然会被我们的浏览器解析，用粗体显示单词百里香。

接下来，我们继续测试 js 模板的特性:

```java
var count = [[${students.size()}]];
alert("Number of students in group: " + count); 
```

模板模式下的属性将被 JavaScript 取消转义。这将导致创建 js 警报。我们通过使用 jQuery AJAX 在 listStudents.html 文件中加载此警报:

```java
<script>
    $(document).ready(function() {
        $.ajax({
            url : "/spring-thymeleaf/js",
            });
        });
</script>
```

最后，但不是最不重要的功能，我们想测试的是纯文本文件生成。我们创建了包含以下内容的 studentsList.txt 文件:

```java
Dear [(${username})],

This is the list of our students:
[# th:each="s : ${students}"]
   - [(${s.name})]. ID: [(${s.id})]
[/]
Thanks,
The Baeldung University
```

注意，与标记模板模式一样，标准方言只包括一个可处理的元素(`[# …])`和一组可处理的属性(`th:text, th:utext, th:if, th:unless, th:each`等)。).结果将是一个文本文件，我们可以在电子邮件中使用，就像在第 3 节末尾提到的那样。

**怎么考？**我们的建议是先使用浏览器，然后检查现有的 JUnit 测试。

## 6.**Spring Boot 百里香叶**

`Spring Boot`通过添加 [`spring-boot-starter-thymeleaf`](https://web.archive.org/web/20220810180959/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-thymeleaf%22) 依赖关系为`Thymeleaf`提供自动配置:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <version>2.3.3.RELEASE</version>
</dependency> 
```

不需要显式配置。默认情况下，HTML 文件应该放在`resources/templates `位置。

## 7。结论

在本文中，我们讨论了在百里香框架中实现的新特性，重点是 3.0 版。

本教程的完整实现可以在 GitHub 项目中找到——这是一个基于 Eclipse 的项目，很容易在每个现代互联网浏览器中测试。

最后，如果您计划将一个项目从版本 2 迁移到这个最新版本，请看迁移指南中的[。请注意，您现有的百里香模板与百里香 3.0 几乎 100%兼容，因此您只需在配置中做一些修改。](https://web.archive.org/web/20220810180959/http://www.thymeleaf.org/doc/articles/thymeleaf3migration.html)