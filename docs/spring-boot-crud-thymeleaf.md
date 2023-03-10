# Spring Boot 与百里香叶 CRUD 应用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-crud-thymeleaf>

## 1。概述

在 [JPA](https://web.archive.org/web/20220712173816/https://en.wikipedia.org/wiki/Java_Persistence_API) 实体上提供 CRUD 功能的 [DAO](/web/20220712173816/https://www.baeldung.com/java-dao-pattern) 层的实现可能是一个重复的、耗时的任务，在大多数情况下我们都想避免。

幸运的是， [Spring Boot](/web/20220712173816/https://www.baeldung.com/spring-boot) 使得通过一层标准的基于 JPA 的 CRUD 库创建 CRUD 应用程序变得很容易。

在本教程中，我们将学习如何用 Spring Boot 和[百里香](/web/20220712173816/https://www.baeldung.com/thymeleaf-in-spring-mvc)开发一个 CRUD web 应用程序。

## 延伸阅读:

## [百里香叶的春季请求参数](/web/20220712173816/https://www.baeldung.com/spring-thymeleaf-request-parameters)

Learn how to use request parameters with Spring and Thymeleaf.[Read more](/web/20220712173816/https://www.baeldung.com/spring-thymeleaf-request-parameters) →

## [更改 Spring Boot 的百里香模板目录](/web/20220712173816/https://www.baeldung.com/spring-thymeleaf-template-directory)

Learn about Thymeleaf template locations.[Read more](/web/20220712173816/https://www.baeldung.com/spring-thymeleaf-template-directory) →

## 2。美芬依赖

在这种情况下，我们将依靠[spring-boot-starter-parent](https://web.archive.org/web/20220712173816/https://search.maven.org/search?q=a:spring-boot-starter-parent&g:org.springframework.boot=&core=gav)进行简单的依赖管理、版本控制和插件配置。

因此，我们不需要在我们的`pom.xml`文件中指定项目依赖项的版本，除非覆盖 Java 版本:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
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
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
    </dependency>
</dependencies> 
```

## 3。域层

有了所有的项目依赖项，现在让我们实现一个简单的领域层。

为了简单起见，这一层将包括一个负责建模`User`实体的类:

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    @NotBlank(message = "Name is mandatory")
    private String name;

    @NotBlank(message = "Email is mandatory")
    private String email;

    // standard constructors / setters / getters / toString
}
```

让我们记住，我们已经用`@Entity`注释对类进行了注释。**因此，JPA 实现，也就是 Hibernate，在** **这种情况下，将能够在域实体上执行 CRUD 操作。**关于 Hibernate 的入门指南，请访问我们的教程 [Hibernate 5 with Spring](/web/20220712173816/https://www.baeldung.com/hibernate-5-spring) 。

此外，我们用`@NotBlank`约束约束了`name`和`email`字段。这意味着在持久化或更新数据库中的实体之前，我们可以使用 Hibernate Validator 来验证受约束的字段。

关于这方面的基础知识，请查看[我们关于 Bean 验证的相关教程](/web/20220712173816/https://www.baeldung.com/javax-validation)。

## 4。储存库层

此时，我们的示例 web 应用程序什么都不做。但这种情况即将改变。

Spring Data JPA **允许我们毫不费力地实现基于 JPA 的存储库(DAO 模式实现的一个有趣的名字)。**

[Spring Data JPA](/web/20220712173816/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 是 Spring Boot`spring-boot-starter-data-jpa`的一个关键组件，它使得通过放置在 JPA 实现之上的强大抽象层来添加 CRUD 功能变得容易。这个抽象层允许我们访问持久层，而不必从头开始提供我们自己的 DAO 实现。

为了给我们的应用程序提供关于`User`对象的基本 CRUD 功能，我们只需要扩展`CrudRepository`接口:

```java
@Repository
public interface UserRepository extends CrudRepository<User, Long> {}
```

就是这样！通过扩展`CrudRepository`接口，Spring Data JPA 将为我们提供存储库的 CRUD 方法的实现。

## 5。控制器层

**多亏了`spring-boot-starter-data-jpa`放置在底层 JPA 实现之上的抽象层，我们可以通过一个基本的 web 层轻松地向我们的 web 应用程序添加一些 CRUD 功能。**

在我们的例子中，一个控制器类就足以处理 GET 和 POST HTTP 请求，然后将它们映射到对我们的`UserRepository`实现的调用。

控制器类依赖于 Spring MVC 的一些关键特性。关于 Spring MVC 的详细指南，请查看我们的 [Spring MVC 教程](/web/20220712173816/https://www.baeldung.com/spring-mvc-tutorial)。

先说控制器的`showSignUpForm()`和`addUser()`方法。

前者将显示用户注册表单，而后者将在验证受约束的字段后在数据库中持久保存一个新实体。

如果实体没有通过验证，将重新显示注册表单。

否则，一旦保存了实体，持久化实体的列表将在相应的视图中更新:

```java
@Controller
public class UserController {

    @GetMapping("/signup")
    public String showSignUpForm(User user) {
        return "add-user";
    }

    @PostMapping("/adduser")
    public String addUser(@Valid User user, BindingResult result, Model model) {
        if (result.hasErrors()) {
            return "add-user";
        }

        userRepository.save(user);
        return "redirect:/index";
    }

    // additional CRUD methods
}
```

我们还需要一个针对`/index` URL 的映射:

```java
@GetMapping("/index")
public String showUserList(Model model) {
    model.addAttribute("users", userRepository.findAll());
    return "index";
}
```

在`UserController`中，我们还会有`showUpdateForm()`方法，它负责从数据库中获取与提供的`id`相匹配的`User`实体。

如果实体存在，它将作为模型属性传递给更新表单视图。

因此，可以用`name`和`email`字段的值填充表单:

```java
@GetMapping("/edit/{id}")
public String showUpdateForm(@PathVariable("id") long id, Model model) {
    User user = userRepository.findById(id)
      .orElseThrow(() -> new IllegalArgumentException("Invalid user Id:" + id));

    model.addAttribute("user", user);
    return "update-user";
} 
```

最后，我们在`UserController`类中有`updateUser()`和`deleteUser()`方法。

第一个将在数据库中持久保存更新的实体，而最后一个将删除给定的实体。

在任一情况下，持久化实体的列表将相应地更新:

```java
@PostMapping("/update/{id}")
public String updateUser(@PathVariable("id") long id, @Valid User user, 
  BindingResult result, Model model) {
    if (result.hasErrors()) {
        user.setId(id);
        return "update-user";
    }

    userRepository.save(user);
    return "redirect:/index";
}

@GetMapping("/delete/{id}")
public String deleteUser(@PathVariable("id") long id, Model model) {
    User user = userRepository.findById(id)
      .orElseThrow(() -> new IllegalArgumentException("Invalid user Id:" + id));
    userRepository.delete(user);
    return "redirect:/index";
}
```

## 6。视图层

至此，我们已经实现了一个功能控制器类，它对`User`实体执行 CRUD 操作。**尽管如此，这个模式中仍然缺少一个组件:视图层。**

在`src/main/resources/templates`文件夹下，我们需要创建显示注册表单和更新表单以及呈现持久化的`User`实体列表所需的 HTML 模板。

正如简介中所述，我们将使用 Thymeleaf 作为解析模板文件的底层模板引擎。

下面是`add-user.html`文件的相关部分:

```java
<form action="#" th:action="@{/adduser}" th:object="${user}" method="post">
    <label for="name">Name</label>
    <input type="text" th:field="*{name}" id="name" placeholder="Name">
    <span th:if="${#fields.hasErrors('name')}" th:errors="*{name}"></span>
    <label for="email">Email</label>
    <input type="text" th:field="*{email}" id="email" placeholder="Email">
    <span th:if="${#fields.hasErrors('email')}" th:errors="*{email}"></span>
    <input type="submit" value="Add User">   
</form>
```

**注意我们如何使用`@{/adduser}` URL 表达式来指定表单的`action`属性和`${}`变量表达式，以便在模板中嵌入动态内容，比如`name`和`email`字段的值以及验证后错误。**

类似于`add-user.html`，这里是`update-user.html`模板的样子:

```java
<form action="#" 
  th:action="@{/update/{id}(id=${user.id})}" 
  th:object="${user}" 
  method="post">
    <label for="name">Name</label>
    <input type="text" th:field="*{name}" id="name" placeholder="Name">
    <span th:if="${#fields.hasErrors('name')}" th:errors="*{name}"></span>
    <label for="email">Email</label>
    <input type="text" th:field="*{email}" id="email" placeholder="Email">
    <span th:if="${#fields.hasErrors('email')}" th:errors="*{email}"></span>
    <input type="submit" value="Update User">   
</form>
```

最后，我们有一个`index.html`文件，它显示了持久化实体的列表，以及用于编辑和删除现有实体的链接:

```java
<div th:switch="${users}">
    <h2 th:case="null">No users yet!</h2>
        <div th:case="*">
            <h2>Users</h2>
            <table>
                <thead>
                    <tr>
                        <th>Name</th>
                        <th>Email</th>
                        <th>Edit</th>
                        <th>Delete</th>
                    </tr>
                </thead>
                <tbody>
                <tr th:each="user : ${users}">
                    <td th:text="${user.name}"></td>
                    <td th:text="${user.email}"></td>
                    <td><a th:href="@{/edit/{id}(id=${user.id})}">Edit</a></td>
                    <td><a th:href="@{/delete/{id}(id=${user.id})}">Delete</a></td>
                </tr>
            </tbody>
        </table>
    </div>      
    <p><a href="/signup">Add a new user</a></p>
</div> 
```

为了简单起见，模板看起来相当骨架，只提供所需的功能，没有添加不必要的修饰。

为了给模板一个改进的、引人注目的外观，而不用在 HTML/CSS 上花费太多时间，我们可以很容易地使用一个免费的 [Twitter Bootstrap](https://web.archive.org/web/20220712173816/https://getbootstrap.com/) UI 工具包，比如 [Shards](https://web.archive.org/web/20220712173816/https://designrevision.com/downloads/shards/) 。

## 7 .**。运行应用程序**

最后，让我们定义应用程序的入口点。

像大多数 Spring Boot 应用程序一样，我们可以用一个简单的老方法来实现:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

现在让我们在 IDE 中点击“Run ”,然后打开浏览器并指向`http://localhost:8080`。

如果构建已经成功编译，我们应该会看到一个基本的 CRUD 用户仪表盘，带有添加新实体、编辑和删除现有实体的链接。

## 8。结论

在本文中，我们学习了如何用 Spring Boot 和百里香叶构建一个基本的 CRUD web 应用程序。

和往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220712173816/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-crud)