# Spring Web 应用程序中的 Flash 属性指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-web-flash-attributes>

## 1.概观

Web 应用程序通常依赖用户输入来满足它们的一些用例。因此，表单提交是为这类应用程序收集和处理数据的常用机制。

在本教程中，我们将了解到 **Spring 的 [flash 属性](https://web.archive.org/web/20220524021952/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/mvc/support/RedirectAttributes.html)如何安全可靠地帮助我们完成表单提交工作流**。

## 2.Flash 属性基础

在我们能够轻松使用 flash 属性之前，我们需要对表单提交工作流和一些关键的相关概念有一个相当好的理解。

### 2.1.发布/重定向/获取模式

设计 web 表单的一个简单方法是使用一个单独的 [HTTP POST](https://web.archive.org/web/20220524021952/https://en.wikipedia.org/wiki/POST_(HTTP)) 请求，该请求负责提交并通过其响应返回一个确认。然而，这样的设计暴露了重复处理 POST 请求的风险，以防用户最终刷新页面。

**为了减轻重复处理的问题，我们可以按照特定的顺序将工作流创建为一系列互连的请求——即 POST、REDIRECT 和 GET** 。简而言之，我们称之为表单提交的 [Post/Redirect/Get (PRG)](https://web.archive.org/web/20220524021952/https://en.wikipedia.org/wiki/Post/Redirect/Get) 模式。

在接收到 POST 请求时，服务器处理它，然后将控制权转移给 GET 请求。随后，将根据 GET 请求的响应显示确认页面。理想情况下，即使最后一个 GET 请求被尝试了不止一次，也不应该有任何副作用。

### 2.2.闪存属性的生命周期

为了使用 [PRG](https://web.archive.org/web/20220524021952/https://en.wikipedia.org/wiki/Post/Redirect/Get) 模式完成表单提交，我们需要在重定向后将信息从最初的 POST 请求传输到最终的 GET 请求。

不幸的是，我们既不能使用`[RequestAttributes](https://web.archive.org/web/20220524021952/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/context/request/RequestAttributes.html)`也不能使用`[SessionAttributes](https://web.archive.org/web/20220524021952/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/SessionAttributes.html).` ,因为前者无法在跨不同控制器的重定向中存活，而后者将持续整个会话，甚至在表单提交结束之后。

但是，我们不需要担心，因为 Spring 的 web 框架提供了可以解决这个问题的 flash 属性。

让我们看看`[RedirectAttributes](https://web.archive.org/web/20220524021952/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/mvc/support/RedirectAttributes.html)`接口中的方法，它们可以帮助我们在项目中使用 flash 属性:

```
RedirectAttributes addFlashAttribute(String attributeName, @Nullable Object attributeValue);

RedirectAttributes addFlashAttribute(Object attributeValue);

Map<String, ?> getFlashAttributes();
```

**闪现属性短暂**。因此，就在重定向之前，它们被临时存储在某个底层存储中。重定向后，它们仍可用于后续请求，然后就消失了。

### 2.3.`FlashMap`数据结构

Spring 提供了一个名为 [`FlashMap`](https://web.archive.org/web/20220524021952/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/FlashMap.html) 的抽象数据结构，用于将 flash 属性存储为键值对。

让我们来看看`FlashMap`类的定义:

```
public final class FlashMap extends HashMap<String, Object> implements Comparable<FlashMap> {

    @Nullable
    private String targetRequestPath;

    private final MultiValueMap<String, String> targetRequestParams 
      = new LinkedMultiValueMap<>(4);

    private long expirationTime = -1;
}
```

我们可以注意到，`FlashMap`类继承了`[HashMap](/web/20220524021952/https://www.baeldung.com/java-hashmap)`类的行为。因此， **`FlashMap`实例可以存储属性**的键值映射。此外，我们可以绑定一个`FlashMap`实例，仅由特定的重定向 URL 使用。

此外，每个请求有两个`FlashMap`实例，即输入`FlashMap`和输出`FlashMap,` ，它们在 PRG 模式中起着重要作用:

*   输出`FlashMap`在 POST 请求中用于临时保存闪存属性，并在重定向后将其发送给下一个 GET 请求
*   在最终的 GET 请求中使用输入`FlashMap`来访问重定向前一个 POST 请求发送的只读闪存属性

### 2.4.`FlashMapManager`和`RequestContextUtils`

顾名思义，我们可以使用`[FlashMapManager](https://web.archive.org/web/20220524021952/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/FlashMapManager.html)`来管理`FlashMap`实例。

首先，让我们来看看这个策略界面的定义:

```
public interface FlashMapManager {

    @Nullable
    FlashMap retrieveAndUpdate(HttpServletRequest request, HttpServletResponse response);

    void saveOutputFlashMap(FlashMap flashMap, HttpServletRequest request, HttpServletResponse response);
}
```

简单地说，我们可以说,`FlashMapManager`允许我们在一些底层存储中读取、更新和保存`FlashMap`实例。

接下来，让我们熟悉一下 [`RequestContextUtils`](https://web.archive.org/web/20220524021952/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/support/RequestContextUtils.html) 抽象实用程序类中可用的几个**静态方法。**

为了将我们的焦点保持在本教程的范围内，我们将把我们的覆盖范围限制在与 flash 属性相关的方法上:

```
public static Map<String, ?> getInputFlashMap(HttpServletRequest request);

public static FlashMap getOutputFlashMap(HttpServletRequest request);

public static FlashMapManager getFlashMapManager(HttpServletRequest request);

public static void saveOutputFlashMap(String location, 
  HttpServletRequest request, HttpServletResponse response);
```

我们可以使用这些方法来检索输入/输出`FlashMap`实例，获取请求的`FlashMapManager`，并保存一个`FlashMap`实例。

## 3.表单提交用例

到目前为止，我们已经对闪存属性的不同概念有了基本的了解。所以，让我们更进一步，在一个诗歌比赛 web 应用程序中使用它们。

我们的诗歌比赛应用程序有一个简单的用例，通过提交表单接受来自不同诗人的诗歌参赛作品。此外，参赛作品将包含与一首诗相关的必要信息，如标题、正文和作者姓名。

### 3.1.百里香叶构型

我们将使用 **[百里香](/web/20220524021952/https://www.baeldung.com/thymeleaf-in-spring-mvc)，这是一个 Java 模板引擎，用于通过简单的 HTML 模板创建动态网页**。

首先，我们需要将`[spring-boot-starter-thymeleaf](https://web.archive.org/web/20220524021952/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-thymeleaf)`依赖项添加到项目的`pom.xml`中:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <version>2.2.1.RELEASE</version>
</dependency>
```

接下来，我们可以在位于`src/main/resources`目录下的`pplication.properties`文件中定义一些百里香叶特有的属性:

```
spring.thymeleaf.cache=false
spring.thymeleaf.enabled=true 
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

定义了这些属性后，我们现在可以在`/src/main/resources/templates`目录下创建所有视图。反过来，Spring 将把`.html`后缀附加到我们的控制器中命名的所有视图上。

### 3.2.领域模型

接下来，让我们**在`Poem`类中定义我们的域模型**:

```
public class Poem {
    private String title;
    private String author;
    private String body;
}
```

此外，我们可以在我们的`Poem`类中添加`isValidPoem()`静态方法来帮助我们验证字段不允许空字符串:

```
public static boolean isValidPoem(Poem poem) {
    return poem != null && Strings.isNotBlank(poem.getAuthor()) 
      && Strings.isNotBlank(poem.getBody())
      && Strings.isNotBlank(poem.getTitle());
}
```

### 3.3.创建表单

现在，我们准备创建我们的提交表单。为此，**我们需要一个端点`/poem/submit`，它将服务一个 GET 请求，向用户显示表单**:

```
@GetMapping("/poem/submit")
public String submitGet(Model model) {
    model.addAttribute("poem", new Poem());
    return "submit";
}
```

这里，我们使用一个模型作为容器来保存用户提供的特定于诗歌的数据。此外，`submitGet `方法返回由`submit`视图服务的视图。

此外，我们希望将 POST 表单与模型属性`poem`绑定在一起:

```
<form action="#" method="post" th:action="@{/poem/submit}" th:object="${poem}">
    <!-- form fields for poem title, body, and author -->
</form> 
```

### 3.4.发布/重定向/获取提交流程

现在，让我们为表单启用 POST 操作。为此，我们将在`PoemSubmission`控制器中创建 **`/poem/submit`端点来服务 POST 请求**:

```
@PostMapping("/poem/submit")
public RedirectView submitPost(
    HttpServletRequest request, 
    @ModelAttribute Poem poem, 
    RedirectAttributes redirectAttributes) {
    if (Poem.isValidPoem(poem)) {
        redirectAttributes.addFlashAttribute("poem", poem);
        return new RedirectView("/poem/success", true);
    } else {
        return new RedirectView("/poem/submit", true);
    }
}
```

我们可以注意到，**如果提交成功，那么控制权转移到`/poem/success`** 端点。此外，在启动重定向之前，我们添加了诗歌数据作为 flash 属性。

现在，我们需要向用户显示一个确认页面，所以让我们实现服务于 GET 请求的`/poem/success`端点的功能:

```
@GetMapping("/poem/success")
public String getSuccess(HttpServletRequest request) {
    Map<String, ?> inputFlashMap = RequestContextUtils.getInputFlashMap(request);
    if (inputFlashMap != null) {
        Poem poem = (Poem) inputFlashMap.get("poem");
        return "success";
    } else {
        return "redirect:/poem/submit";
    }
}
```

这里需要注意的是，在我们决定重定向到成功页面之前，我们**需要验证`FlashMap` 。**

最后，让我们使用成功页面中的 flash 属性`poem `来显示用户提交的诗歌的标题:

```
<h1 th:if="${poem}">
    <p th:text="${'You have successfully submitted poem titled - '+ poem?.title}"/>
    Click <a th:href="@{/poem/submit}"> here</a> to submit more.
</h1>
```

## 4.结论

在本教程中，我们学习了一些关于 Post/Redirect/Get 模式和 flash 属性的概念。此外，我们还在一个 Spring Boot web 应用程序中的简单表单提交中看到了 flash 属性的作用。

和往常一样，该教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220524021952/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-3)