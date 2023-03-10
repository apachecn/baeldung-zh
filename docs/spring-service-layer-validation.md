# 服务层中的 Spring 验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-service-layer-validation>

## 1。 **概述**

在本教程中，我们将讨论 Java 应用程序的服务层中的 Spring 验证。虽然 **Spring Boot 支持与定制验证器的无缝集成，但是执行验证的事实标准是[Hibernate Validator](https://web.archive.org/web/20221129002729/http://hibernate.org/validator/)。**

在这里，我们将学习如何将我们的验证逻辑从我们的控制器中转移到一个单独的服务层中。此外，我们将在 Spring 应用程序的服务层实现验证。

## 2。应用分层

根据不同的需求，Java 业务应用程序可以采用几种不同的形式和类型。例如，我们必须根据这些标准来确定我们的应用程序需要哪些层。除非有特定的需求，否则许多应用程序不会从服务或存储库层增加的复杂性和维护成本中受益。

我们可以通过使用多层来解决所有这些问题。这些层是:

[![Layered Architecture](img/e0bfabe3d2187a1299bfc27e237e5519.png)](/web/20221129002729/https://www.baeldung.com/wp-content/uploads/2021/06/Layered-Architecture.png)

消费者层或 web 层是 Web 应用程序的最顶层。它负责解释用户的输入，并提供适当的响应。其他层抛出的异常也必须由 web 层处理。由于 web 层是我们的应用程序的入口点，它负责身份验证，并作为防止未授权用户的第一道防线。

web 层下面是服务层。它充当了一个事务屏障，并容纳了应用程序和基础架构服务。此外，服务层的公共 API 由应用服务提供。它们通常作为交易边界，负责授权交易。基础设施服务提供了连接外部工具(包括文件系统、数据库和电子邮件服务器)的“管道代码”。这些方法通常由几个应用程序服务使用。

web 应用程序的最低层是持久层。换句话说，它负责与用户的数据存储进行交互。

## 3。服务层中的**验证**

服务层是应用程序中便于控制器和持久层之间通信的一层。此外，业务逻辑存储在服务层。它特别包括验证逻辑。模型状态用于控制器和服务层之间的通信。

将验证视为业务逻辑有利也有弊，Spring 的验证(和数据绑定)架构也不排除这两者。尤其是验证，不应该绑定到 web 层，应该易于本地化，并且应该允许使用任何可用的验证器。

此外，客户端输入数据并不总是通过 REST 控制器进程，并且**如果我们不在服务层也进行验证，不可接受的数据可能会通过，从而导致几个问题**。在本例中，**我们将使用标准的 Java JSR-303 验证方案**。

## 4。 **举例**

让我们考虑一个使用 Spring Boot 开发的简单用户帐户注册表单。

### 4.1。简单域类

首先，我们只有姓名、年龄、电话和密码属性:

```java
public class UserAccount {

    @NotNull(message = "Password must be between 4 to 15 characters")
    @Size(min = 4, max = 15)
    private String password;

    @NotBlank(message = "Name must not be blank")
    private String name;

    @Min(value = 18, message = "Age should not be less than 18")
    private int age;

    @NotBlank(message = "Phone must not be blank")
    private String phone;

    // standard constructors / setters / getters / toString
}
```

在上面的类中，我们使用了四个注释——`@NotNull`、`@Size`、`@NotBlank`和`@Min`——来确保输入属性既不是 null 也不是空白，并且符合大小要求。

### 4.2。在服务层实现验证

有许多可用的验证解决方案，Spring 或 Hibernate 处理实际的验证。**另一方面，手动验证是一种可行的替代方法**。当涉及到将验证集成到我们应用的正确部分时，这给了我们很大的灵活性。

接下来，让我们在服务类中实现我们的验证:

```java
@Service
public class UserAccountService {

    @Autowired
    private Validator validator;

    @Autowired
    private UserAccountDao dao;

    public String addUserAccount(UserAccount useraccount) {

        Set<ConstraintViolation<UserAccount>> violations = validator.validate(useraccount);

        if (!violations.isEmpty()) {
            StringBuilder sb = new StringBuilder();
            for (ConstraintViolation<UserAccount> constraintViolation : violations) {
                sb.append(constraintViolation.getMessage());
            }
            throw new ConstraintViolationException("Error occurred: " + sb.toString(), violations);
        }

        dao.addUserAccount(useraccount);       
        return "Account for " + useraccount.getName() + " Added!";
    }
}
```

**T0【是 Bean 验证 API 的一部分，负责验证 Java 对象**。此外，Spring 自动提供了一个`Validator`实例，我们可以将它注入到我们的`UserAccountService`中。`Validator`用于验证`validate(..)`函数中传递的对象。结果是`ConstraintViolation`的一个`Set`。

如果没有违反验证约束(对象有效)，则`Set`为空。否则，我们扔个`ConstraintViolationException`。

### 4.3。实施休息控制器

在此之后，让我们构建 Spring REST 控制器类来向客户端或最终用户显示服务，并评估应用程序的输入验证:

```java
@RestController
public class UserAccountController {

    @Autowired
    private UserAccountService service;

    @PostMapping("/addUserAccount")
    public Object addUserAccount(@RequestBody UserAccount userAccount) {
        return service.addUserAccount(userAccount);
    }
}
```

我们没有在上面的 REST 控制器表单中使用`@Valid`注释来防止任何验证。

### 4.4。测试剩余控制器

现在，让我们通过运行 Spring Boot 应用程序来测试这个方法。之后，使用 Postman 或任何其他 API 测试工具，我们将 JSON 输入发布到`localhost:8080/addUserAccount` URL:

```java
{
   "name":"Baeldung",
   "age":25,
   "phone":"1234567890",
   "password":"test",
   "useraddress":{
      "countryCode":"UK"
   }
}
```

After confirming that the test runs successfully, let's now check if the validation is working as per the expectation. The next logical step is to test the application with few invalid inputs. Hence, we'll update our input JSON with invalid values:

```java
{
   "name":"",
   "age":25,
   "phone":"1234567890",
   "password":"",
   "useraddress":{
      "countryCode":"UK"
   }
}
```

The console now shows the error message, Hence, **we can see how the usage of Validator is essential for validation**:

```java
Error occurred: Password must be between 4 to 15 characters, Name must not be blank
```

## 5.利弊

在服务/业务层，这通常是一种成功的验证方法。它不限于方法参数，可以应用于各种对象。例如，我们可以从数据库中加载一个对象，更改它，然后在继续之前验证它。

我们也可以使用这种方法进行单元测试，这样我们就可以模拟服务类。**为了便于在单元测试中进行真正的验证，我们可以手工生成必要的`Validator`实例**。

在我们的测试中，这两种情况都不需要引导 Spring 应用程序上下文。

## 6.结论

在这个快速教程中，我们探索了 Java 业务应用程序的不同层。我们学习了如何将我们的验证逻辑从我们的控制器中转移到一个独立的服务层中。此外，我们实现了一种在 Spring 应用程序的服务层执行验证的方法。

示例中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221129002729/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-validation)