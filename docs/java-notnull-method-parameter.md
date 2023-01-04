# 在方法参数上使用@NotNull

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-notnull-method-parameter>

## 1.概观

这是一个常见的问题。保护代码的一种方法是给方法参数添加注释，比如`@NotNull` 。

通过使用@ `NotNull`，我们表明如果我们想避免一个异常，我们决不能用`null`调用我们的方法。然而，就其本身而言，这还不够。我们来了解一下原因。

## 2.`@NotNull`方法参数上的注释

首先，让我们用一个简单地返回一个`String`的长度的方法创建一个类。

让我们也给我们的参数添加一个`@NotNull`注释:

```java
public class NotNullMethodParameter {
    public int validateNotNull(@NotNull String data) {
        return data.length();
    }
}
```

**当我们导入`NotNull, w` e 时应该注意到一个`@NotNull`注释**有几种实现。**所以，我们需要确保它来自正确的包装。**

我们将使用`javax.validation.constraints`包。

现在，让我们创建一个`NotNullMethodParameter`并用一个`null`参数调用我们的方法:

```java
NotNullMethodParameter notNullMethodParameter = new NotNullMethodParameter();
notNullMethodParameter.doesNotValidate(null);
```

尽管我们有了`NotNull`注释，我们还是得到了一个`NullPointerException`:

```java
java.lang.NullPointerException
```

我们的注释没有效果，因为没有验证器来执行它。

## 3.添加验证程序

所以，让我们添加 Hibernate Validator，`javax.validation`参考实现，来识别我们的`@NotNull`。

除了我们的验证器，我们还需要为表达式语言(EL)添加一个依赖项，它用于呈现消息:

```java
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.2.3.Final</version>
</dependency>

<dependency>
    <groupId>org.glassfish</groupId>
    <artifactId>javax.el</artifactId>
    <version>3.0.0</version>
</dependency>
```

当我们不包括 EL 依赖项时，我们得到一个`ValidationException`来提醒我们:

```java
javax.validation.ValidationException: HV000183: Unable to initialize 'javax.el.ExpressionFactory'. Check that you have the EL dependencies on the classpath, or use ParameterMessageInterpolator instead
```

有了依赖关系，我们就可以实施我们的`@NotNull`注释。

因此，让我们使用默认的`ValidatorFactory`创建一个验证器:

```java
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
Validator validator = factory.getValidator();
```

然后，让我们把我们的论点作为我们注释方法的第一行:

```java
validator.validate(myString);
```

现在，当我们用空参数调用我们的方法时，我们的`@NotNull`被强制执行:

```java
java.lang.IllegalArgumentException: HV000116: The object to be validated must not be null.
```

这很好，但是**必须在每个带注释的方法中添加对我们的验证器的调用，这导致了大量的样板文件**。

## 4.Spring Boot

幸运的是，我们可以在 Spring Boot 应用程序中使用更简单的方法。

### 4.1.Spring Boot 验证

首先，让我们添加 Maven 依赖项，以便用 Spring Boot 进行验证:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
    <version>2.7.1</version>
</dependency>
```

我们的依赖带来了我们需要的所有 Spring Boot 和认可。这意味着我们可以移除之前对 Hibernate 和 EL 的依赖来保持我们的`pom.xml`干净。

现在，让我们创建一个 Spring 管理的`Component`，**，确保我们添加了`@Validated`** **注释**。让我们用一个`validateNotNull`方法来创建它，该方法接受一个`String`参数并返回我们数据的长度，并用@ `NotNull`来注释我们的参数:

```java
@Component
@Validated
public class ValidatingComponent {
    public int validateNotNull(@NotNull String data) {
        return data.length();
    }
}
```

最后，让我们用自动连接的`ValidatingComponent`创建一个`SpringBootTest`。让我们也添加一个使用`null`作为方法参数的测试:

```java
@SpringBootTest
class ValidatingComponentTest {
    @Autowired ValidatingComponent component;

    @Test
    void givenNull_whenValidate_thenConstraintViolationException() {
        assertThrows(ConstraintViolationException.class, () -> component.validate(null));
    }
}
```

我们得到的`ConstraintViolationException`有我们的参数名和一个‘不得为空’的消息`:`

```java
javax.validation.ConstraintViolationException: validate.data: must not be null
```

我们可以在我们的[方法约束](/web/20220815135856/https://www.baeldung.com/javax-validation-method-constraints)文章中了解更多关于注释方法的信息。

### 4.2.告诫的话

尽管这适用于我们的`public`方法，但是让我们看看当我们添加另一个没有注释但是调用我们原来的注释方法的方法时会发生什么:

```java
public String callAnnotatedMethod(String data) {
    return validateNotNull(data);
}
```

我们的`NullPointerException`回来了。**当我们从同一个类中的另一个方法调用带注释的方法时，Spring 不会强制执行`NotNull`约束。**

### 4.3.雅加达和 Spring Boot 3.0

对于 Jakarta，**验证包名称最近从`javax.validation`更改为`jakarta.validation`** 。Spring Boot 3.0 基于雅加达，所以使用了更新的`jakarta.validation`软件包。从 7.0 开始的`hibernate-validator`版本也是如此。*及以后。这意味着当我们升级时，我们需要改变我们在验证注释中使用的包名。

## 5.结论

在本文中，我们学习了如何在标准 Java 应用程序中的方法参数上使用`@NotNull`注释。我们还学习了如何使用 Spring Boot 的`@Validated`注释来简化我们的 Spring Bean 方法参数验证，同时也注意到了它的局限性。最后，我们注意到，当我们将 Spring Boot 项目更新到 3.0 时，我们应该会将我们的`javax`包更改为`jakarta`。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220815135856/https://github.com/eugenp/tutorials/tree/master/javaxval)