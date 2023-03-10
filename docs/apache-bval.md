# Apache BVal 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-bval>

## 1。简介

在本文中，我们将看看 **`Apache BVal`库对`Java Bean Validation`规范(`JSR 349` )** 的实现。

## 2。Maven 依赖关系

为了使用`Apache BVal`，我们首先需要将以下依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.apache.bval</groupId>
    <artifactId>bval-jsr</artifactId>
    <version>1.1.2</version>
</dependency>
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>
```

自定义`BVal`约束可以在可选的`bval-extras`依赖关系中找到:

```java
<dependency>
    <groupId>org.apache.bval</groupId>
    <artifactId>bval-extras</artifactId>
    <version>1.1.2</version>
</dependency>
```

最新版本的 [bval-jsr](https://web.archive.org/web/20220524020626/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22bval-jsr%22) 、 [bval-extras](https://web.archive.org/web/20220524020626/https://search.maven.org/classic/#search%7Cga%7C1%7Capache%20bval%20extras) 和 [validation-api](https://web.archive.org/web/20220524020626/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22javax.validation%22) 可以从 Maven Central 下载。

## 3。应用约束

`Apache BVal`提供了在`javax.validation`包中定义的所有约束的实现。为了将约束应用于 bean 的属性，我们可以**将约束注释添加到属性声明**。

让我们创建一个有四个属性的`User`类，然后应用`@NotNull`、`@Size`和`@Min`注释:

```java
public class User {

    @NotNull
    private String email;

    private String password;

    @Size(min=1, max=20)
    private String name;

    @Min(18)
    private int age;

    // standard constructor, getters, setters
}
```

## 4。验证 bean

为了验证应用在`User`类上的约束，我们需要获得一个`ValidatorFactory`实例和一个或多个`Validator`实例。

### 4.1。`ValidatorFactory`获得一个

`Apache BVal`文档建议获取该类的一个实例，因为工厂创建是一个要求很高的过程:

```java
ValidatorFactory validatorFactory 
  = Validation.byProvider(ApacheValidationProvider.class)
  .configure().buildValidatorFactory();
```

### 4.2。`Validator`获得一个

接下来，我们需要从上面定义的`validatorFactory`中获取一个`Validator`实例:

```java
Validator validator = validatorFactory.getValidator();
```

这是一个线程安全的实现，所以我们可以安全地重用已经创建的实例。

`Validator`类提供了三种方法来确定 bean 的有效性:`validate()`、 `validateProperty()`和`validateValue()`。

这些方法中的每一个都返回一组`ConstraintViolation`对象，这些对象包含关于未遵守的约束的信息。

### 4.3。`validate()` API

`validate()`方法检查整个 bean 的有效性，这意味着它**验证应用于作为参数传递的对象**的属性的所有约束。

让我们创建一个`JUnit`测试，其中我们定义了一个`User`对象，并使用`validate()`方法来测试它的属性:

```java
@Test
public void givenUser_whenValidate_thenValidationViolations() {
    User user
      = new User("[[email protected]](/web/20220524020626/https://www.baeldung.com/cdn-cgi/l/email-protection)", "pass", "nameTooLong_______________", 15);

    Set<ConstraintViolation<User>> violations = validator.validate(user);
    assertTrue("no violations", violations.size() > 0);
}
```

### 4.4。`validateProperty()` API

`validateProperty()`方法可用于**验证 bean** 的单个属性。

让我们创建一个`JUnit`测试，在这个测试中，我们将定义一个`User`对象，它的`age`属性小于要求的最小值 18，并验证验证这个属性会导致一个违例:

```java
@Test
public void givenInvalidAge_whenValidateProperty_thenConstraintViolation() {
    User user = new User("[[email protected]](/web/20220524020626/https://www.baeldung.com/cdn-cgi/l/email-protection)", "pass", "Ana", 12);

    Set<ConstraintViolation<User>> propertyViolations
      = validator.validateProperty(user, "age");

    assertEquals("size is not 1", 1, propertyViolations.size());
}
```

### 4.5。`validateValue()` API

在将某个值设置到 bean 上之前，`validateValue()`方法可用于**检查该值对于 bean** 的属性是否是有效值。

让我们用一个`User`对象创建一个`JUnit`测试，然后验证值`20`对于属性`age`是一个有效值:

```java
@Test
public void givenValidAge_whenValidateValue_thenNoConstraintViolation() {
    User user = new User("[[email protected]](/web/20220524020626/https://www.baeldung.com/cdn-cgi/l/email-protection)", "pass", "Ana", 18);

    Set<ConstraintViolation<User>> valueViolations
      = validator.validateValue(User.class, "age", 20);

    assertEquals("size is not 0", 0, valueViolations.size());
}
```

### 4.6。`ValidatorFactory`关闭

**使用完`ValidatorFactory`后，一定要记得在结束时关闭**:

```java
if (validatorFactory != null) {
    validatorFactory.close();
}
```

## 5。非`JSR`约束

`Apache BVal`库还提供了一系列不属于`JSR`规范的**约束，并提供了额外的、更强大的验证功能。**

`bval-jsr`包包含两个附加约束:`@Email`用于验证有效的电子邮件地址，以及`@NotEmpty`用于确保值不为空。

其余的自定义`BVal`约束在可选包`bval-extras`中提供。

这个包包含用于验证各种数字格式的**约束，比如确保数字是有效的国际银行账号的`@IBAN`注释，验证有效的标准图书编号的`@Isbn`注释，以及验证国际商品编号的`@EAN13`注释。**

该库还提供了**注释，用于确保各类信用卡号** : `@AmericanExpress`、`@Diners`、`@Discover`、`@Mastercard`、`@Visa`的有效性。

您可以通过使用`@Domain`和`@InetAddress`注释来**确定一个值是否包含有效的域或互联网地址**。

最后，这个包包含了 `@Directory`和`@NotDirectory`注释，用于**验证一个`File`对象是否是一个目录**。

让我们在我们的`User`类上定义额外的属性，并对它们应用一些非`JSR`注释:

```java
public class User {

    @NotNull
    @Email
    private String email;

    @NotEmpty
    private String password;

    @Size(min=1, max=20)
    private String name;

    @Min(18)
    private int age;

    @Visa
    private String cardNumber = "";

    @IBAN
    private String iban = "";

    @InetAddress
    private String website = "";

    @Directory
    private File mainDirectory = new File(".");

    // standard constructor, getters, setters
}
```

可以用与`JSR`约束相似的方式测试约束:

```java
@Test
public void whenValidateNonJSR_thenCorrect() {
    User user = new User("[[email protected]](/web/20220524020626/https://www.baeldung.com/cdn-cgi/l/email-protection)", "pass", "Ana", 20);
    user.setCardNumber("1234");
    user.setIban("1234");
    user.setWebsite("10.0.2.50");
    user.setMainDirectory(new File("."));

    Set<ConstraintViolation<User>> violations 
      = validator.validateProperty(user,"iban");

    assertEquals("size is not 1", 1, violations.size());

    violations = validator.validateProperty(user,"website");

    assertEquals("size is not 0", 0, violations.size());

    violations = validator.validateProperty(user, "mainDirectory");

    assertEquals("size is not 0", 0, violations.size());
}
```

虽然这些额外的注释对于潜在的验证需求来说很方便，但是使用不属于`JSR`规范的注释的一个缺点是，如果以后需要的话，您不能轻易地切换到不同的`JSR`实现。

## 6。自定义约束

为了定义我们自己的约束，我们首先需要创建一个遵循标准语法的注释。

让我们创建一个`Password`注释，它将定义用户密码必须满足的条件:

```java
@Constraint(validatedBy = { PasswordValidator.class })
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface Password {
    String message() default "Invalid password";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    int length() default 6;

    int nonAlpha() default 1;
}
```

**`password`值的实际验证是在实现`ConstraintValidator`接口**的类中完成的——在我们的例子中，是`PasswordValidator`类。这个类覆盖了`isValid()`方法，并验证`password`的长度是否小于`length`属性，以及它包含的非字母数字字符是否少于`nonAlpha`属性中指定的数量:

```java
public class PasswordValidator 
  implements ConstraintValidator<Password, String> {

    private int length;
    private int nonAlpha;

    @Override
    public void initialize(Password password) {
        this.length = password.length();
        this.nonAlpha = password.nonAlpha();
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext ctx) {
        if (value.length() < length) {
            return false;
        }
        int nonAlphaNr = 0;
        for (int i = 0; i < value.length(); i++) {
            if (!Character.isLetterOrDigit(value.charAt(i))) {
                nonAlphaNr++;
            }
        }
        if (nonAlphaNr < nonAlpha) {
            return false;
        }
        return true;
    }
}
```

让我们将自定义约束应用于`User`类的`password`属性:

```java
@Password(length = 8)
private String password;
```

我们可以创建一个`JUnit`测试来验证无效的`password`值会导致违反约束:

```java
@Test
public void givenValidPassword_whenValidatePassword_thenNoConstraintViolation() {
    User user = new User("[[email protected]](/web/20220524020626/https://www.baeldung.com/cdn-cgi/l/email-protection)", "password", "Ana", 20);
    Set<ConstraintViolation<User>> violations 
      = validator.validateProperty(user, "password");

    assertEquals(
      "message incorrect",
      "Invalid password", 
      violations.iterator().next().getMessage());
}
```

现在让我们创建一个`JUnit`测试，在这个测试中我们验证一个有效的`password`值:

```java
@Test
public void givenValidPassword_whenValidatePassword_thenNoConstraintViolation() {
    User user = new User("[[email protected]](/web/20220524020626/https://www.baeldung.com/cdn-cgi/l/email-protection)", "password#", "Ana", 20);

    Set<ConstraintViolation<User>> violations 
      = validator.validateProperty(user, "password");
    assertEquals("size is not 0", 0, violations.size());
}
```

## 7 .**。结论**

在本文中，我们举例说明了`Apache BVal` bean 验证实现的使用。

本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524020626/https://github.com/eugenp/tutorials/tree/master/apache-libraries)