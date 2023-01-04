# Spring MVC 中的会话属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-session-attributes>

## 1。概述

在开发 web 应用程序时，我们经常需要在几个视图中引用相同的属性。例如，我们可能有需要在多个页面上显示的购物车内容。

存储这些属性的一个好位置是在用户会话中。

在本教程中，我们将关注一个简单的例子，并研究使用会话属性的两种不同策略:

*   **使用作用域代理**
*   **使用@ `SessionAttributes`标注**

## 2。Maven 设置

我们将使用 Spring Boot 启动器来引导我们的项目，并引入所有必要的依赖项。

我们的设置需要一个父声明、web 启动器和百里香启动器。

我们还将包括 spring test starter，以便在我们的单元测试中提供一些额外的实用工具:

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.2</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
     </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

这些依赖项的最新版本可以在 Maven Central 上找到[。](https://web.archive.org/web/20220922154514/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20(a%3A%22spring-boot-starter-web%22%20OR%20a%3A%22spring-boot-starter-thymeleaf%22%20OR%20a%3A%22spring-boot-starter-test%22))

## 3。示例用例

我们的示例将实现一个简单的“TODO”应用程序。我们将有一个创建`TodoItem`实例的表单和一个显示所有`TodoItem`的列表视图。

如果我们使用表单创建一个`TodoItem`，表单的后续访问将用最近添加的`TodoItem`的值进行预填充。**我们将使用** t **his 特性来演示如何“记住”存储在会话范围内的表单值**。

我们的两个模型类被实现为简单的 POJOs:

```
public class TodoItem {

    private String description;
    private LocalDateTime createDate;

    // getters and setters
}
```

```
public class TodoList extends ArrayDeque<TodoItem>{

}
```

我们的`TodoList`类扩展了`ArrayDeque`,让我们可以通过`peekLast`方法方便地访问最近添加的项目。

我们需要两个控制器类:我们要研究的每种策略一个。它们会有细微的差别，但核心功能在两者中都有体现。每个人将有 3 个`@RequestMapping`:

*   `**@GetMapping(“/form”)**`–该方法将负责初始化表单并呈现表单视图。如果`TodoList`不为空，该方法将用最近添加的`TodoItem`预填充表单。
*   **`@PostMapping(“/form”)`**–该方法负责将提交的`TodoItem`添加到`TodoList`中，并重定向到列表 URL。
*   **`@GetMapping(“/todos.html”) –`** 这个方法只是将`TodoList`添加到`Model`中进行显示，并呈现列表视图。

## 4。使用作用域代理

### 4.1。设置

在这个设置中，我们的`TodoList`被配置为由代理支持的会话范围的`@Bean`。事实上，`@Bean`是一个代理，这意味着我们能够将它注入到我们的单例作用域`@Controller`中。

由于上下文初始化时没有会话，Spring 将创建一个代理`TodoList`作为依赖注入。当请求需要时，`TodoList`的目标实例将被实例化。

关于 Spring 中 bean 作用域的更深入的讨论，请参考我们关于主题的[文章。](/web/20220922154514/https://www.baeldung.com/spring-bean-scopes)

首先，我们在一个`@Configuration`类中定义 bean:

```
@Bean
@Scope(
  value = WebApplicationContext.SCOPE_SESSION, 
  proxyMode = ScopedProxyMode.TARGET_CLASS)
public TodoList todos() {
    return new TodoList();
}
```

接下来，我们将 bean 声明为对`@Controller`的依赖，并像对任何其他依赖一样注入它:

```
@Controller
@RequestMapping("/scopedproxy")
public class TodoControllerWithScopedProxy {

    private TodoList todos;

    // constructor and request mappings
} 
```

最后，在请求中使用 bean 只需要调用它的方法:

```
@GetMapping("/form")
public String showForm(Model model) {
    if (!todos.isEmpty()) {
        model.addAttribute("todo", todos.peekLast());
    } else {
        model.addAttribute("todo", new TodoItem());
    }
    return "scopedproxyform";
}
```

### 4.2。单元测试

为了使用作用域代理**测试我们的实现，我们首先配置一个`SimpleThreadScope`** `.` 这将确保我们的单元测试准确地模拟我们正在测试的代码的运行时条件。

首先，我们定义一个`TestConfig`和一个`CustomScopeConfigurer`:

```
@Configuration
public class TestConfig {

    @Bean
    public CustomScopeConfigurer customScopeConfigurer() {
        CustomScopeConfigurer configurer = new CustomScopeConfigurer();
        configurer.addScope("session", new SimpleThreadScope());
        return configurer;
    }
}
```

现在我们可以开始测试表单的初始请求是否包含未初始化的`TodoItem:`

```
@RunWith(SpringRunner.class) 
@SpringBootTest
@AutoConfigureMockMvc
@Import(TestConfig.class) 
public class TodoControllerWithScopedProxyIntegrationTest {

    // ...

    @Test
    public void whenFirstRequest_thenContainsUnintializedTodo() throws Exception {
        MvcResult result = mockMvc.perform(get("/scopedproxy/form"))
          .andExpect(status().isOk())
          .andExpect(model().attributeExists("todo"))
          .andReturn();

        TodoItem item = (TodoItem) result.getModelAndView().getModel().get("todo");

        assertTrue(StringUtils.isEmpty(item.getDescription()));
    }
} 
```

我们还可以确认我们的提交发出了一个重定向，并且后续的表单请求已经预先填充了新添加的`TodoItem`:

```
@Test
public void whenSubmit_thenSubsequentFormRequestContainsMostRecentTodo() throws Exception {
    mockMvc.perform(post("/scopedproxy/form")
      .param("description", "newtodo"))
      .andExpect(status().is3xxRedirection())
      .andReturn();

    MvcResult result = mockMvc.perform(get("/scopedproxy/form"))
      .andExpect(status().isOk())
      .andExpect(model().attributeExists("todo"))
      .andReturn();
    TodoItem item = (TodoItem) result.getModelAndView().getModel().get("todo");

    assertEquals("newtodo", item.getDescription());
}
```

### 4.3。讨论

使用作用域代理策略的一个关键特性是**它对请求映射方法签名没有影响。**与`@SessionAttributes`策略相比，这保持了很高的可读性。

回想一下默认情况下控制器有`singleton`作用域是很有帮助的。

这就是为什么我们必须使用代理，而不是简单地注入一个无代理的会话范围的 bean。我们不能将一个较小范围的 bean 注入到一个较大范围的 bean 中。

在这种情况下，尝试这样做将会触发一个异常，并显示一条包含`Scope ‘session' is not active for the current thread`的消息。

如果我们愿意用会话范围来定义我们的控制器，我们可以避免指定一个`proxyMode`。这可能有缺点，特别是如果创建控制器的成本很高，因为必须为每个用户会话创建一个控制器实例。

注意`TodoList` 可用于其他组件的注射。这可能是一个优点，也可能是一个缺点，具体取决于使用案例。如果让 bean 对整个应用程序可用是有问题的，那么可以使用`@SessionAttributes`将实例的范围限定到控制器，我们将在下一个例子中看到。

## 5。使用`@SessionAttributes`标注

### 5.1。设置

在这个设置中，我们没有将`TodoList`定义为弹簧管理的`@Bean`。相反，我们**将它声明为一个`@ModelAttribute`，并指定`@SessionAttributes`注释将它限定在控制器**的会话范围内。

第一次访问我们的控制器时，Spring 将实例化一个实例，并将其放在`Model`中。由于我们也在`@SessionAttributes`中声明了 bean，Spring 将存储实例。

关于春季`@ModelAttribute`更深入的讨论，请参考我们关于的[文章。](/web/20220922154514/https://www.baeldung.com/spring-mvc-and-the-modelattribute-annotation)

首先，我们通过在控制器上提供一个方法来声明我们的 bean，并用`@ModelAttribute`注释这个方法:

```
@ModelAttribute("todos")
public TodoList todos() {
    return new TodoList();
} 
```

接下来，我们通过使用`@SessionAttributes`通知控制器将我们的`TodoList` 视为会话范围的:

```
@Controller
@RequestMapping("/sessionattributes")
@SessionAttributes("todos")
public class TodoControllerWithSessionAttributes {
    // ... other methods
}
```

最后，为了在请求中使用 bean，我们在`@RequestMapping`的方法签名中提供了对它的引用:

```
@GetMapping("/form")
public String showForm(
  Model model,
  @ModelAttribute("todos") TodoList todos) {

    if (!todos.isEmpty()) {
        model.addAttribute("todo", todos.peekLast());
    } else {
        model.addAttribute("todo", new TodoItem());
    }
    return "sessionattributesform";
} 
```

在`@PostMapping`方法中，我们在返回我们的`RedirectView`之前注入`RedirectAttributes`并调用`addFlashAttribute`。与我们的第一个示例相比，这是实现中的一个重要区别:

```
@PostMapping("/form")
public RedirectView create(
  @ModelAttribute TodoItem todo, 
  @ModelAttribute("todos") TodoList todos, 
  RedirectAttributes attributes) {
    todo.setCreateDate(LocalDateTime.now());
    todos.add(todo);
    attributes.addFlashAttribute("todos", todos);
    return new RedirectView("/sessionattributes/todos.html");
}
```

Spring 为重定向场景使用了一个专门的`Model`的`RedirectAttributes` 实现来支持 URL 参数的编码。在重定向过程中，存储在`Model`上的任何属性通常只有包含在 URL 中时才可用于框架。

**通过使用`addFlashAttribute` ，我们告诉框架，我们希望我们的`TodoList` 在重定向**后仍然存在，而不需要在 URL 中对其进行编码。

### 5.2。单元测试

窗体视图控制器方法的单元测试与我们在第一个例子中看到的测试是相同的。然而，`@PostMapping`的测试略有不同，因为我们需要访问闪存属性来验证行为:

```
@Test
public void whenTodoExists_thenSubsequentFormRequestContainsesMostRecentTodo() throws Exception {
    FlashMap flashMap = mockMvc.perform(post("/sessionattributes/form")
      .param("description", "newtodo"))
      .andExpect(status().is3xxRedirection())
      .andReturn().getFlashMap();

    MvcResult result = mockMvc.perform(get("/sessionattributes/form")
      .sessionAttrs(flashMap))
      .andExpect(status().isOk())
      .andExpect(model().attributeExists("todo"))
      .andReturn();
    TodoItem item = (TodoItem) result.getModelAndView().getModel().get("todo");

    assertEquals("newtodo", item.getDescription());
}
```

### 5.3。讨论

在会话中存储属性的`@ModelAttribute`和`@SessionAttributes`策略是一个简单的解决方案，即**不需要额外的上下文配置或 Spring 管理的`@Bean`的**。

与我们的第一个例子不同，有必要在`@RequestMapping` 方法`.`中注入`TodoList`

此外，我们必须将闪存属性用于重定向场景。

## 6。结论

在本文中，我们研究了使用作用域代理和`@SessionAttributes`作为在 Spring MVC 中处理会话属性的两种策略。请注意，在这个简单的示例中，存储在会话中的任何属性将只在会话的生命周期中存在。

如果我们需要在服务器重启或会话超时之间保存属性，我们可以考虑使用 Spring Session 来透明地保存信息。查看我们关于春季会议的文章了解更多信息。

和往常一样，本文中使用的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220922154514/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-forms-thymeleaf)