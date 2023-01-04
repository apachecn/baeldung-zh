# 参数消息内插器指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-parametermessageinterpolator>

## 1.概观

Java JSR 380 的一个特性是允许在用参数插入验证消息时使用表达式。

当我们使用 Hibernate Validator 时，有一个需求是我们需要添加一个 Java JSR 341 的统一实现作为我们项目的依赖项。 JSR 341 也被称为表达式语言 API。

然而，如果我们不需要根据我们的用例来支持解析表达式，那么添加额外的库可能会很麻烦。

在这个简短的教程中，我们将看看如何在 Hibernate Validator 中配置 [`ParameterMessageInterpolator`](https://web.archive.org/web/20220625224013/https://docs.jboss.org/hibernate/stable/validator/api/org/hibernate/validator/messageinterpolation/ParameterMessageInterpolator.html) 。

## 2.消息插值器

除了验证 Java bean 的基础知识之外，bean 验证 API 的`MessageInterpolator`是一种抽象，它为我们提供了一种执行简单插值的方法，而没有解析表达式的麻烦。

此外， **Hibernate Validator 提供了一个不基于表达式的`ParameterMessageInterpolator,` ，因此，我们不需要任何额外的库来配置它。**

## 3.设置自定义消息插值器

为了消除表达式语言依赖性，我们可以使用定制的消息内插器，并在没有表达式支持的情况下配置 Hibernate Validator。

让我们展示一些设置定制消息内插器的简便方法。在我们的例子中，我们将使用内置的`ParameterMessageInterpolator`。

### 3.1.配置`ValidatorFactory`

设置自定义消息内插器的一种方法是在引导时配置`ValidatorFactory`。

因此，我们可以用`ParameterMessageInterpolator`构建一个`ValidatorFactory`实例:

```java
ValidatorFactory validatorFactory = Validation.byDefaultProvider()
  .configure()
  .messageInterpolator(new ParameterMessageInterpolator())
  .buildValidatorFactory(); 
```

### 3.2.配置`Validator`

类似地，我们可以在初始化`Validator`实例时设置`ParameterMessageInterpolator`:

```java
Validator validator = validatorFactory.usingContext()
  .messageInterpolator(new ParameterMessageInterpolator())
  .getValidator(); 
```

## 4.执行验证

为了了解`ParameterMessageInterpolator`是如何工作的，我们需要一个样本 Java bean，上面有一些 JSR 380 注释。

### 4.1.样本 Java Bean

让我们定义我们的样本 Java bean `Person`:

```java
public class Person {

    @Size(min = 10, max = 100, message = "Name should be between {min} and {max} characters")
    private String name;

    @Min(value = 18, message = "Age should not be less than {value}")
    private int age;

    @Email(message = "Email address should be in a correct format: ${validatedValue}")
    private String email;

    // standard getters and setters
} 
```

### 4.2.测试消息参数

当然，为了执行我们的验证，我们应该使用一个从`ValidatorFactory,` 访问的`Validator`实例，这是我们在`.`之前已经配置好的

因此，我们需要访问我们的`Validator`:

```java
Validator validator = validatorFactory.getValidator(); 
```

之后，我们可以为`name`字段编写测试方法:

```java
@Test
public void givenNameLengthLessThanMin_whenValidate_thenValidationFails() {
    Person person = new Person();
    person.setName("John Doe");
    person.setAge(18);

    Set<ConstraintViolation<Person>> violations = validator.validate(person);

    assertEquals(1, violations.size());

    ConstraintViolation<Person> violation = violations.iterator().next();

    assertEquals("Name should be between 10 and 100 characters", violation.getMessage());
}
```

验证消息正确地插入了变量`{min}`和`{max}`:

```java
Name should be between 10 and 100 characters 
```

接下来，让我们为`age`字段编写一个类似的测试:

```java
@Test
public void givenAgeIsLessThanMin_whenValidate_thenValidationFails() {
    Person person = new Person();
    person.setName("John Stephaner Doe");
    person.setAge(16);

    Set<ConstraintViolation<Person>> violations = validator.validate(person);

    assertEquals(1, violations.size());

    ConstraintViolation<Person> violation = violations.iterator().next();

    assertEquals("Age should not be less than 18", violation.getMessage());
}
```

类似地，如我们所料，验证消息被正确地插入了变量`{value}`:

```java
Age should not be less than 18 
```

### 4.3.测试表达式

为了查看`ParameterMessageInterpolator`如何处理表达式，让我们为`email`字段编写另一个测试，其中包含一个简单的`${validatedValue}`表达式:

```java
@Test
public void givenEmailIsMalformed_whenValidate_thenValidationFails() {
    Person person = new Person();
    person.setName("John Stephaner Doe");
    person.setAge(18);
    person.setEmail("johndoe.dev");

    Set<ConstraintViolation<Person>> violations = validator.validate(person);

    assertEquals(1, violations.size());

    ConstraintViolation<Person> violation = violations.iterator().next();

    assertEquals("Email address should be in a correct format: ${validatedValue}", violation.getMessage());
}
```

这一次，表达式`${validatedValue}`没有插值。

**`ParameterMessageInterpolator`只支持参数的插值，不解析使用`$`符号的表达式。**相反，它只是简单地返回未插值的图像。

## 5.结论

在本文中，我们了解了`ParameterMessageInterpolator`的用途以及如何在 Hibernate Validator 中配置它。

和往常一样，本教程涉及的所有例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220625224013/https://github.com/eugenp/tutorials/tree/master/javaxval)