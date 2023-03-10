# Bean 验证中@NotNull、@NotEmpty 和@NotBlank 约束的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bean-validation-not-null-empty-blank>

## 1。概述

[Bean 验证](https://web.archive.org/web/20221103025345/https://beanvalidation.org/2.0/) **是一个标准的验证规范，它允许我们通过使用一组以注释形式声明的约束来轻松验证域对象**。

虽然总体上 bean 验证实现的使用相当简单，比如 [Hibernate Validator](https://web.archive.org/web/20221103025345/http://hibernate.org/validator/) ，但是关于这些约束是如何实现的，有必要探究一些微妙但相关的差异。

在本教程中，**我们将探讨`@NotNull`、`@NotEmpty,`和`@NotBlank`约束**之间的区别。

## 2。美芬依赖

为了快速建立工作环境并测试`@NotNull`、`@NotEmpty`和`@NotBlank`约束的行为，首先我们需要添加所需的 Maven 依赖项。

在这种情况下，我们将使用[Hibernate Validator](https://web.archive.org/web/20221103025345/http://hibernate.org/validator/)(bean 验证参考实现)来验证我们的域对象。

这里是我们的`pom.xml`文件的相关部分:

```java
<dependencies> 
    <dependency> 
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>6.0.13.Final</version>
    </dependency> 
    <dependency> 
        <groupId>org.glassfish</groupId>
        <artifactId>javax.el</artifactId>
        <version>3.0.0</version>
     </dependency>
</dependencies> 
```

我们将在单元测试中使用 [JUnit](https://web.archive.org/web/20221103025345/https://junit.org/junit5/) 和 [AssertJ](https://web.archive.org/web/20221103025345/https://joel-costigliola.github.io/assertj/) ，所以请务必在 Maven Central 上查看最新版本的 [hibernate-validator](https://web.archive.org/web/20221103025345/https://search.maven.org/search?q=g:org.hibernate%20AND%20a:hibernate-validator&core=gav) 、 [GlassFish 的 EL 实现](https://web.archive.org/web/20221103025345/https://mvnrepository.com/artifact/org.glassfish/javax.el)、 [junit](https://web.archive.org/web/20221103025345/https://search.maven.org/search?q=g:junit%20AND%20a:junit&core=gav) 和 [assertj-core](https://web.archive.org/web/20221103025345/https://search.maven.org/search?q=g:org.assertj%20AND%20a:assertj-core&core=gav) 。

## 3。`@NotNull`约束

接下来，让我们实现一个简单的`UserNotNull`域类，并用`@NotNull`注释约束的`name`字段:

```java
public class UserNotNull {

    @NotNull(message = "Name may not be null")
    private String name;

    // standard constructors / getters / toString   
}
```

现在我们需要检查`@NotNull` 实际上是如何在`. `下工作的

为此，让我们为该类创建一个简单的单元测试，并验证它的几个实例:

```java
@BeforeClass
public static void setupValidatorInstance() {
    validator = Validation.buildDefaultValidatorFactory().getValidator();
}

@Test
public void whenNotNullName_thenNoConstraintViolations() {
    UserNotNull user = new UserNotNull("John");
    Set<ConstraintViolation<UserNotNull>> violations = validator.validate(user);

    assertThat(violations.size()).isEqualTo(0);
}

@Test
public void whenNullName_thenOneConstraintViolation() {
    UserNotNull user = new UserNotNull(null);
    Set<ConstraintViolation<UserNotNull>> violations = validator.validate(user);

    assertThat(violations.size()).isEqualTo(1);
}

@Test
public void whenEmptyName_thenNoConstraintViolations() {
    UserNotNull user = new UserNotNull("");
    Set<ConstraintViolation<UserNotNull>> violations = validator.validate(user);

    assertThat(violations.size()).isEqualTo(0);
} 
```

正如所料，`@NotNull`约束不允许受约束的字段为空值。但是，字段可以为空。

为了更好地理解这一点，让我们看一下 [`NotNullValidator`类](https://web.archive.org/web/20221103025345/http://docs.jboss.org/ejb3/app-server/HibernateAnnotations/api/org/hibernate/validator/NotNullValidator.html#isValid(java.lang.Object)) ' `isValid()`方法，这是`@NotNull`约束使用的方法。该方法的实现非常简单:

```java
public boolean isValid(Object object) {
    return object != null;  
}
```

如上图，用`@NotNull`约束的**字段(如`CharSequence`、`Collection`、`Map,`或`Array)`必须不为空。然而，空值是完全合法的**。

## 4。`@NotEmpty`约束

现在让我们实现一个示例`UserNotEmpty`类并使用`@NotEmpty`约束:

```java
public class UserNotEmpty {

    @NotEmpty(message = "Name may not be empty")
    private String name;

    // standard constructors / getters / toString
}
```

有了这个类，让我们通过给`name`字段分配不同的值来测试它:

```java
@Test
public void whenNotEmptyName_thenNoConstraintViolations() {
    UserNotEmpty user = new UserNotEmpty("John");
    Set<ConstraintViolation<UserNotEmpty>> violations = validator.validate(user);

    assertThat(violations.size()).isEqualTo(0);
}

@Test
public void whenEmptyName_thenOneConstraintViolation() {
    UserNotEmpty user = new UserNotEmpty("");
    Set<ConstraintViolation<UserNotEmpty>> violations = validator.validate(user);

    assertThat(violations.size()).isEqualTo(1);
}

@Test
public void whenNullName_thenOneConstraintViolation() {
    UserNotEmpty user = new UserNotEmpty(null);
    Set<ConstraintViolation<UserNotEmpty>> violations = validator.validate(user);

    assertThat(violations.size()).isEqualTo(1);
}
```

`@NotEmpty` 注释利用了`@NotNull`类的`isValid()`实现，并检查所提供对象的大小/长度(当然，这根据被验证对象的类型而变化)是否大于零。

简言之，**这意味着用`@NotEmpty` 约束的字段(如`CharSequence`、`Collection`、`Map,`或`Array)`必须不为空，并且其大小/长度必须大于零**。

此外，如果将`@NotEmpty`注释与`@Size.`一起使用，我们可以更加严格

这样做时，我们还会强制对象的最小和最大大小值在指定的最小/最大范围内:

```java
@NotEmpty(message = "Name may not be empty")
@Size(min = 2, max = 32, message = "Name must be between 2 and 32 characters long") 
private String name; 
```

## 5。`@NotBlank`约束

类似地，我们可以用`@NotBlank`注释约束一个类字段:

```java
public class UserNotBlank {

    @NotBlank(message = "Name may not be blank")
    private String name;

    // standard constructors / getters / toString

}
```

同样，我们可以实现一个单元测试来理解`@NotBlank`约束是如何工作的:

```java
@Test
public void whenNotBlankName_thenNoConstraintViolations() {
    UserNotBlank user = new UserNotBlank("John");
    Set<ConstraintViolation<UserNotBlank>> violations = validator.validate(user);

    assertThat(violations.size()).isEqualTo(0);
}

@Test
public void whenBlankName_thenOneConstraintViolation() {
    UserNotBlank user = new UserNotBlank(" ");
    Set<ConstraintViolation<UserNotBlank>> violations = validator.validate(user);

    assertThat(violations.size()).isEqualTo(1);
}

@Test
public void whenEmptyName_thenOneConstraintViolation() {
    UserNotBlank user = new UserNotBlank("");
    Set<ConstraintViolation<UserNotBlank>> violations = validator.validate(user);

    assertThat(violations.size()).isEqualTo(1);
}

@Test
public void whenNullName_thenOneConstraintViolation() {
    UserNotBlank user = new UserNotBlank(null);
    Set<ConstraintViolation<UserNotBlank>> violations = validator.validate(user);

    assertThat(violations.size()).isEqualTo(1);
} 
```

`@NotBlank`注释使用了 [`NotBlankValidator`](https://web.archive.org/web/20221103025345/https://docs.jboss.org/hibernate/validator/6.0/api/org/hibernate/validator/internal/constraintvalidators/hv/NotBlankValidator.html) 类，该类检查字符序列的修整长度是否为空:

```java
public boolean isValid(
  CharSequence charSequence, 
  ConstraintValidatorContext constraintValidatorContext)
    if (charSequence == null ) {
        return true; 
    } 
    return charSequence.toString().trim().length() > 0;
} 
```

有趣的是，对于空值，该方法返回 true。所以我们可能会认为`@NotBlank`确实允许空值，但实际上不允许。

**在`@NotBlank`类'`isValid()`之后调用了`@NotNull`类'`isValid()`方法，因此禁止空值。**

简单来说，**用`@NotBlank` 约束的一个`String`字段必须不为空，修剪后的长度必须大于零**。

## 6。并排比较

到目前为止，我们已经深入了解了`@NotNull`、`@NotEmpty`和`@NotBlank`约束如何在类字段上单独操作。

让我们执行一个快速的并排比较，这样我们可以鸟瞰约束的功能，并轻松地发现它们的差异:

*   `@NotNull:`被约束的`CharSequence`、`Collection`、`Map,`或`Array`只要不为空就是有效的，但可以为空。
*   `@NotEmpty:`一个被约束的`CharSequence`、`Collection`、`Map,`或`Array`只要不为空，并且其大小/长度大于零，就是有效的。
*   `@NotBlank:`一个被约束的`String`只要不为空，并且修剪长度大于零，就是有效的。

## 7 .**。结论**

在本文中，我们查看了 Bean 验证中实现的`@NotNull`、`@NotEmpty`和`@NotBlank`约束，并强调了它们的相似之处和不同之处。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221103025345/https://github.com/eugenp/tutorials/tree/master/javaxval)