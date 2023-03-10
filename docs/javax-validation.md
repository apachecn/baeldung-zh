# Java Bean 验证基础

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javax-validation>

## 1。概述

在这个快速教程中，我们将介绍用标准框架——JSR 380，也称为 T0——验证 Java bean 的基础知识。

在大多数应用程序中，验证用户输入是一个非常常见的需求。Java Bean 验证框架已经成为处理这种逻辑的事实上的标准。

## 延伸阅读:

## [在 Spring Boot 验证](/web/20220706124058/https://www.baeldung.com/spring-boot-bean-validation)

Learn how to validate domain objects in Spring Boot using Hibernate Validator, the reference implementation of the Bean Validation framework.[Read more](/web/20220706124058/https://www.baeldung.com/spring-boot-bean-validation) →

## 【Bean 验证 2.0 的方法约束

An introduction to method constraints using Bean Validation 2.0.[Read more](/web/20220706124058/https://www.baeldung.com/javax-validation-method-constraints) →

## 2。JSR 380

JSR 380 是用于 bean 验证的 Java API 的规范，是 Jakarta EE 和 JavaSE 的一部分。这确保了 bean 的属性满足特定的标准，使用诸如`@NotNull`、`@Min`和`@Max`的注释。

该版本需要 Java 8 或更高版本，并利用 Java 8 中添加的新功能，如类型注释和对新类型的支持，如`Optional`和`LocalDate`。

有关规格的完整信息，请继续通读 [JSR 380](https://web.archive.org/web/20220706124058/https://jcp.org/en/jsr/detail?id=380) 。

## 3。依赖性

我们将使用一个 Maven 示例来展示所需的依赖关系。但是当然，这些罐子可以通过各种方式添加。

### 3.1。验证 API

根据 JSR 380 规范，*验证-api* 依赖关系包含标准验证 api:

```java
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
```

### 3.2。验证 API 参考实现

Hibernate Validator 是验证 API 的参考实现。

要使用它，我们需要添加以下依赖项:

```java
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.13.Final</version>
</dependency> 
```

快速注意: **` [hibernate-validator](https://web.archive.org/web/20220706124058/https://search.maven.org/artifact/org.hibernate.validator/hibernate-validator)`与 Hibernate 的持久性方面完全分开。**因此，通过将它添加为依赖项，我们没有将这些持久性方面添加到项目中。

### 3.3。表达式语言依赖关系

JSR 380 支持变量插值，允许在违规消息中使用表达式。

为了解析这些表达式，我们将添加来自 GlassFish 的 [`javax.el`](https://web.archive.org/web/20220706124058/https://search.maven.org/artifact/org.glassfish/javax.el) 依赖项，它包含表达式语言规范的实现:

```java
<dependency>
    <groupId>org.glassfish</groupId>
    <artifactId>javax.el</artifactId>
    <version>3.0.0</version>
</dependency>
```

## 4。使用验证注释

这里，我们将使用一个`User` bean，并向它添加一些简单的验证:

```java
import javax.validation.constraints.AssertTrue;
import javax.validation.constraints.Max;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import javax.validation.constraints.Email;

public class User {

    @NotNull(message = "Name cannot be null")
    private String name;

    @AssertTrue
    private boolean working;

    @Size(min = 10, max = 200, message 
      = "About Me must be between 10 and 200 characters")
    private String aboutMe;

    @Min(value = 18, message = "Age should not be less than 18")
    @Max(value = 150, message = "Age should not be greater than 150")
    private int age;

    @Email(message = "Email should be valid")
    private String email;

    // standard setters and getters 
} 
```

示例中使用的所有注释都是标准的 JSR 注释:

*   ***@NotNull*** 验证注释属性值不是 *null* 。
*   ***@AssertTrue*** 验证注释属性值为`true.`
*   ***@Size*** 验证注释属性值的大小在属性 *min* 和 *max* 之间；可以应用于`String`、`Collection`、`Map`和数组属性。
*   ***@Min*** 验证注释属性的值不小于*值*属性。
*   ***@Max*** 验证注释属性的值不大于*值*属性。
*   `**@Email**`验证带注释的属性是有效的电子邮件地址。

一些注释接受额外的属性，但是*消息*属性是所有注释共有的。这是当相应属性值验证失败时通常会呈现的消息。

以及一些可以在 JSR 中找到的附加注释:

*   `**@NotEmpty**`验证属性不为 null 或空；可应用于`String`、`Collection`、`Map`或`Array`值。
*   `**@NotBlank**`只能应用于文本值，并验证该属性不为空或空白。
*   `**@Positive**`和`**@PositiveOrZero**`适用于数值，并验证它们是严格正的，或正的，包括 0。
*   `**@Negative**`和`**@NegativeOrZero**`适用于数值，并验证它们是严格的负数，或包括 0 在内的负数。
*   `**@Past**` 和`**@PastOrPresent**`验证日期值是过去还是包括现在在内的过去；可以应用于包括 Java 8 中添加的日期类型。
*   `**@Future** and **@FutureOrPresent**`验证日期值是在未来，还是在包括现在在内的未来。

**验证注释也可以应用于集合的元素**:

```java
List<@NotBlank String> preferences;
```

在这种情况下，添加到首选项列表中的任何值都将被验证。

此外，规范**支持 Java 8 中新的`Optional`类型**:

```java
private LocalDate dateOfBirth;

public Optional<@Past LocalDate> getDateOfBirth() {
    return Optional.of(dateOfBirth);
}
```

这里，验证框架将自动打开 *LocalDate* 值并验证它。

## 5。程序验证

一些框架——比如 Spring——有简单的方法通过使用注释来触发验证过程。这主要是为了让我们不必与编程验证 API 交互。

现在让我们走手动路线，以编程方式进行设置:

```java
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
Validator validator = factory.getValidator(); 
```

为了验证 bean，我们首先需要一个使用`ValidatorFactory`构建的`Validator`对象。

### 5.1。定义 Bean

我们现在要设置这个无效的用户——使用一个空值`name`:

```java
User user = new User();
user.setWorking(true);
user.setAboutMe("Its all about me!");
user.setAge(50); 
```

### 5.2。验证 Bean

现在我们有了一个`Validator`，我们可以通过将 bean 传递给 *validate* 方法来验证它。

任何违反在`User`对象中定义的约束的行为都将作为*集合*返回:

```java
Set<ConstraintViolation<User>> violations = validator.validate(user); 
```

通过迭代违例，我们可以使用 *getMessage* 方法获得所有违例消息:

```java
for (ConstraintViolation<User> violation : violations) {
    log.error(violation.getMessage()); 
} 
```

在我们的示例(`ifNameIsNull_nameValidationFails`)中，集合将包含一个带有消息“Name 不能为 null”的`ConstraintViolation`。

## 6。结论

本文主要关注标准 Java 验证 API 的简单传递。我们展示了使用`javax.validation`注释和 API 进行 bean 验证的基础。

像往常一样，本文中概念的实现和所有代码片段都可以在 GitHub 上找到[。](https://web.archive.org/web/20220706124058/https://github.com/eugenp/tutorials/tree/master/javaxval)