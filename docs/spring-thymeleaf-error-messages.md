# 在春天用百里香叶显示错误信息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-thymeleaf-error-messages>

## 1.概观

在本教程中，我们将看到**如何在[百里香叶](/web/20220728105348/https://www.baeldung.com/spring-boot-crud-thymeleaf)模板**中显示来自基于 Spring 的后端应用程序的错误消息。

出于演示目的，我们将创建一个简单的 Spring Boot 用户注册应用程序，并验证各个输入字段。此外，我们将看到一个如何处理全局级错误的例子。

首先，我们将快速设置后端应用程序，然后进入 UI 部分。

## 2.示例 Spring Boot 应用程序

为了创建一个简单的用户注册 Spring Boot 应用程序，**我们需要一个控制器、一个存储库和一个实体**。

然而，甚至在此之前，我们应该添加 Maven 依赖项。

### 2.1.Maven 依赖性

让我们添加我们需要的所有 Spring Boot 启动器 MVC 位的[Web](https://web.archive.org/web/20220728105348/https://search.maven.org/search?q=a:spring-boot-starter-web%20AND%20g:org.springframework.boot), hibernate 实体验证的[验证](https://web.archive.org/web/20220728105348/https://search.maven.org/search?q=a:spring-boot-starter-validation%20AND%20g:org.springframework.boot), UI 的[百里香](https://web.archive.org/web/20220728105348/https://search.maven.org/search?q=a:spring-boot-starter-thymeleaf%20AND%20g:org.springframework.boot),存储库的 [JPA](https://web.archive.org/web/20220728105348/https://search.maven.org/search?q=a:spring-boot-starter-data-jpa%20AND%20g:org.springframework.boot) 。此外，我们需要一个 [H2](https://web.archive.org/web/20220728105348/https://search.maven.org/search?q=a:h2%20AND%20g:com.h2database) 依赖项来拥有一个内存数据库:

```java
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-web</artifactId> 
    <version>2.4.3</version> 
</dependency> 
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-validation</artifactId> 
    <version>2.4.3</version> 
</dependency> 
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-thymeleaf</artifactId> 
    <version>2.4.3</version> 
</dependency> 
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-data-jpa</artifactId> 
    <version>2.4.3</version> 
</dependency> 
<dependency> 
    <groupId>com.h2database</groupId> 
    <artifactId>h2</artifactId> 
    <scope>runtime</scope> 
    <version>1.4.200</version> 
</dependency>
```

### 2.2.实体

这是我们的`User`实体:

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @NotEmpty(message = "User's name cannot be empty.")
    @Size(min = 5, max = 250)
    private String fullName;

    @NotEmpty(message = "User's email cannot be empty.")
    private String email;

    @NotNull(message = "User's age cannot be null.")
    @Min(value = 18)
    private Integer age;

    private String country;

    private String phoneNumber;

    // getters and setters
} 
```

正如我们所看到的，我们已经为用户输入添加了一些[验证约束](/web/20220728105348/https://www.baeldung.com/spring-boot-bean-validation)。例如，字段不应为 null 或空，而应具有特定的大小或值。

值得注意的是，我们没有在`country`或`phoneNumber`字段上添加任何约束。这是因为我们将使用它们作为生成全局错误或与特定字段无关的错误的示例。

### 2.3.仓库

我们将使用一个简单的 [JPA 存储库](/web/20220728105348/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)作为我们的基本用例:

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {}
```

### 2.4.控制器

最后，为了在后端将所有东西连接在一起，让我们组装一个`UserController`:

```java
@Controller
public class UserController {

    @Autowired
    private UserRepository repository;
    @GetMapping("/add")
    public String showAddUserForm(User user) {
        return "errors/addUser";
    }

    @PostMapping("/add")
    public String addUser(@Valid User user, BindingResult result, Model model) {
        if (result.hasErrors()) {
            return "errors/addUser";
        }
        repository.save(user);
        model.addAttribute("users", repository.findAll());
        return "errors/home";
    }
}
```

这里我们在路径`/add`定义一个 [`GetMapping`](/web/20220728105348/https://www.baeldung.com/spring-new-requestmapping-shortcuts) 来显示注册表单。我们的`PostMapping`在相同的路径上处理表单提交时的验证，如果一切顺利，随后保存到存储库。

## 3.带有错误消息的百里香叶模板

既然已经介绍了基础知识，我们就来看问题的关键，即创建 UI 模板和显示错误消息(如果有的话)。

**让我们根据可以显示的错误类型来逐步构建模板**。

### 3.1.显示字段错误

Thymeleaf 提供了一个内置的`field.hasErrors`方法，该方法根据给定字段是否存在错误返回一个布尔值。结合 [`th:if`](/web/20220728105348/https://www.baeldung.com/spring-mvc-thymeleaf-conditional-css-classes#using-thif) 我们可以选择显示错误是否存在:

```java
<p th:if="${#fields.hasErrors('age')}">Invalid Age</p>
```

接下来，如果我们希望**添加任何样式，我们可以有条件地使用[`th:class`](/web/20220728105348/https://www.baeldung.com/spring-mvc-thymeleaf-conditional-css-classes#using-thclass)**:

```java
<p  th:if="${#fields.hasErrors('age')}" th:class="${#fields.hasErrors('age')}? error">
  Invalid Age</p>
```

我们简单的嵌入式 CSS 类`error`将元素变成红色:

```java
<style>
    .error {
        color: red;
    }
</style>
```

另一个百里香属性`th:errors`给了我们在指定的选择器上显示所有错误的能力，比如说`email:`

```java
<div>
    <label for="email">Email</label> <input type="text" th:field="*{email}" />
    <p th:if="${#fields.hasErrors('email')}" th:errorclass="error" th:errors="*{email}" />
</div>
```

在上面的代码片段中，我们还可以看到使用 CSS 样式的变化。这里**我们使用`th:errorclass`，这消除了我们使用任何条件属性来应用 CSS** 的需要。

或者，我们可以选择使用 [`th:each`](/web/20220728105348/https://www.baeldung.com/thymeleaf-iteration) 迭代给定字段上的所有验证消息:

```java
<div>
    <label for="fullName">Name</label> <input type="text" th:field="*{fullName}" 
      id="fullName" placeholder="Full Name">
    <ul>
        <li th:each="err : ${#fields.errors('fullName')}" th:text="${err}" class="error" />
    </ul>
</div>
```

值得注意的是，我们在这里使用了另一个百里香方法`fields.errors()`来收集我们的后端应用程序为`fullName`字段返回的所有验证消息。

现在，为了测试这一点，让我们启动我们的引导应用程序，点击端点 [`http://localhost:8080/add`](https://web.archive.org/web/20220728105348/http://localhost:8080/add) 。

当我们不提供任何输入时，我们的页面看起来是这样的:

[![](img/b81098e258e507f3f1e54dcca029cd82.png)](/web/20220728105348/https://www.baeldung.com/wp-content/uploads/2021/04/Thymeleaf_fieldErrs.png)

### 3.2.一次显示所有错误

接下来，让我们看看如何在一个地方显示所有错误信息，而不是一个接一个地显示每个错误信息。

为此，**我们将使用百里香的`fields.hasAnyErrors()`方法**:

```java
<div th:if="${#fields.hasAnyErrors()}">
    <ul>
        <li th:each="err : ${#fields.allErrors()}" th:text="${err}" />
    </ul>
</div>
```

正如我们所看到的，我们在这里使用了另一个变体`fields.allErrors()`来迭代 HTML 表单上所有字段的所有错误。

我们本来可以用`#fields.hasErrors(‘*')`代替`fields.hasAnyErrors()`。类似地，`#fields.errors(‘*')`是上面使用的`#fields.allErrors()`的替代。

效果是这样的:

[![](img/a2ff0529fdbf9285154f2123ead99a70.png)](/web/20220728105348/https://www.baeldung.com/wp-content/uploads/2021/04/Thymeleaf_allErrors.png)

### 3.3.在表单外显示错误

下一个。让我们考虑一个场景，我们希望在 HTML 表单外显示验证消息。

在这种情况下，**不使用选择或`(*{….})`，我们只需要使用格式为`(${….})`** 的全限定变量名:

```java
<h4>Errors on a single field:</h4>
<div th:if="${#fields.hasErrors('${user.email}')}"
 th:errors="*{user.email}"></div>
<ul>
    <li th:each="err : ${#fields.errors('user.*')}" th:text="${err}" />
</ul>
```

这将在`email`字段显示所有错误信息。

现在，**让我们看看如何一次显示所有消息**:

```java
<h4>All errors:</h4>
<ul>
<li th:each="err : ${#fields.errors('user.*')}" th:text="${err}" />
</ul>
```

这是我们在页面上看到的:

[![](img/76d83eae3eed495df1ea9cad2ba3f683.png)](/web/20220728105348/https://www.baeldung.com/wp-content/uploads/2021/04/Thymeleaf_outsideForm.png)

### 3.4.显示全局错误

在现实生活中，可能会出现与特定字段无关的错误。我们可能有一个用例，其中**我们需要考虑多个输入，以便验证一个业务条件**。这些被称为全局错误。

让我们考虑一个简单的例子来说明这一点。对于我们的`country`和`phoneNumber`字段，我们可以添加一个检查，对于给定的国家，电话号码应该以特定的前缀开头。

我们需要在后端做一些更改来添加这种验证。

首先，我们将添加一个 [`Service`](/web/20220728105348/https://www.baeldung.com/spring-component-repository-service) 来执行这个验证:

```java
@Service
public class UserValidationService {
    public String validateUser(User user) {
        String message = "";
        if (user.getCountry() != null && user.getPhoneNumber() != null) {
            if (user.getCountry().equalsIgnoreCase("India") 
              && !user.getPhoneNumber().startsWith("91")) {
                message = "Phone number is invalid for " + user.getCountry();
            }
        }
        return message;
    }
}
```

正如我们所看到的，我们添加了一个小案例。对于国家/地区`India`，电话号码应以前缀`91`开头。

其次，我们需要调整控制器的`PostMapping`:

```java
@PostMapping("/add")
public String addUser(@Valid User user, BindingResult result, Model model) {
    String err = validationService.validateUser(user);
    if (!err.isEmpty()) {
        ObjectError error = new ObjectError("globalError", err);
        result.addError(error);
    }
    if (result.hasErrors()) {
        return "errors/addUser";
    }
    repository.save(user);
    model.addAttribute("users", repository.findAll());
    return "errors/home";
}
```

最后，在百里香模板中，**我们将添加常量`global`来显示这种类型的错误**:

```java
<div th:if="${#fields.hasErrors('global')}">
    <h3>Global errors:</h3>
    <p th:each="err : ${#fields.errors('global')}" th:text="${err}" class="error" />
</div>
```

或者，代替常量，我们可以使用方法`#fields.hasGlobalErrors()`和`#fields.globalErrors()`来达到同样的目的。

这是我们在输入无效输入时看到的内容:

[![](img/0576368183eb520ecf0a03af0ac6d209.png)](/web/20220728105348/https://www.baeldung.com/wp-content/uploads/2021/04/Thymeleaf_global.png)

## 4.结论

在本教程中，我们构建了一个简单的 Spring Boot 应用程序来演示如何在百里香叶中显示各种类型的错误。

我们研究了一个接一个地显示字段错误，然后一次性显示所有错误，HTML 表单外的错误，以及全局错误。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220728105348/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-3)