# 分组 Javax 验证约束

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javax-validation-groups>

## 1.介绍

在我们的 [Java Bean 验证基础](/web/20220628130910/https://www.baeldung.com/javax-validation)教程中，我们看到了各种内置`javax.validation`约束的用法。在本教程中，我们将看到**如何对`javax.validation`约束**进行分组。

## 2.用例

在许多场景中，我们需要**对 bean 的某个字段集应用约束，然后我们想要对同一 bean 的另一个字段集应用约束。**

例如，假设我们有一个两步注册表单。第一步，我们要求用户提供基本信息，如名字、姓氏、电子邮件 id、电话号码和验证码。当用户提交这些数据时，我们只想验证这些信息。

在下一步中，我们要求用户提供一些其他信息，如地址，我们也希望验证这些信息—注意，captcha 在这两个步骤中都存在。

## 3.分组验证约束

所有的`javax`验证约束都有一个名为`groups` `.` **的属性，当我们向一个元素添加约束时，我们可以声明约束所属的组的名称。**这通过在约束的`groups`属性中指定组接口的类名来实现。

理解一件事的最好方法就是把手弄脏。让我们看看如何将`javax`约束组合成组。

### 3.1.声明约束组

第一步是创建一些接口。这些接口将成为约束组名称。在我们的用例中，我们将验证约束分为两组。

让我们看看第一个约束组，`BasicInfo`:

```java
public interface BasicInfo {
}
```

下一个约束组是`AdvanceInfo`:

```java
public interface AdvanceInfo {
}
```

### 3.2.使用约束组

现在我们已经声明了约束组，是时候在我们的`RegistrationForm` Java bean 中使用它们了:

```java
public class RegistrationForm {
    @NotBlank(groups = BasicInfo.class)
    private String firstName;
    @NotBlank(groups = BasicInfo.class)
    private String lastName;
    @Email(groups = BasicInfo.class)
    private String email;
    @NotBlank(groups = BasicInfo.class)
    private String phone;

    @NotBlank(groups = {BasicInfo.class, AdvanceInfo.class})
    private String captcha;

    @NotBlank(groups = AdvanceInfo.class)
    private String street;

    @NotBlank(groups = AdvanceInfo.class)
    private String houseNumber;

    @NotBlank(groups = AdvanceInfo.class)
    private String zipCode;

    @NotBlank(groups = AdvanceInfo.class)
    private String city;

    @NotBlank(groups = AdvanceInfo.class)
    private String contry;
}
```

使用约束 **`groups`** 属性，我们根据我们的用例将 bean 的字段分成两组。**默认情况下，所有约束都包含在默认约束组中。**

### 3.3.测试具有一个组的约束

既然我们已经声明了约束组并在我们的 bean 类中使用了它们，那么是时候看看这些约束组的作用了。

首先，当基本信息不完整时，我们将使用我们的`BasicInfo`约束组进行验证。对于我们在字段的`@NotBlank`约束的`groups`属性中使用了`BasicInfo.class` 的任何空白字段，我们应该会得到一个约束冲突:

```java
public class RegistrationFormUnitTest {
    private static Validator validator;

    @BeforeClass
    public static void setupValidatorInstance() {
        validator = Validation.buildDefaultValidatorFactory().getValidator();
    }

    @Test
    public void whenBasicInfoIsNotComplete_thenShouldGiveConstraintViolationsOnlyForBasicInfo() {
        RegistrationForm form = buildRegistrationFormWithBasicInfo();
        form.setFirstName("");

        Set<ConstraintViolation<RegistrationForm>> violations = validator.validate(form, BasicInfo.class);

        assertThat(violations.size()).isEqualTo(1);
        violations.forEach(action -> {
            assertThat(action.getMessage()).isEqualTo("must not be blank");
            assertThat(action.getPropertyPath().toString()).isEqualTo("firstName");
        });
    }

    private RegistrationForm buildRegistrationFormWithBasicInfo() {
        RegistrationForm form = new RegistrationForm();
        form.setFirstName("devender");
        form.setLastName("kumar");
        form.setEmail("[[email protected]](/web/20220628130910/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        form.setPhone("12345");
        form.setCaptcha("Y2HAhU5T");
        return form;
    }

    //... additional tests
}
```

在下一个场景中，我们将检查高级信息何时不完整，使用我们的`AdvanceInfo`约束组进行验证:

```java
@Test
public void whenAdvanceInfoIsNotComplete_thenShouldGiveConstraintViolationsOnlyForAdvanceInfo() {
    RegistrationForm form = buildRegistrationFormWithAdvanceInfo();
    form.setZipCode("");

    Set<ConstraintViolation<RegistrationForm>> violations = validator.validate(form, AdvanceInfo.class);

    assertThat(violations.size()).isEqualTo(1);
    violations.forEach(action -> {
        assertThat(action.getMessage()).isEqualTo("must not be blank");
        assertThat(action.getPropertyPath().toString()).isEqualTo("zipCode");
    });
}

private RegistrationForm buildRegistrationFormWithAdvanceInfo() {
    RegistrationForm form = new RegistrationForm();
    return populateAdvanceInfo(form);
}

private RegistrationForm populateAdvanceInfo(RegistrationForm form) {
    form.setCity("Berlin");
    form.setContry("DE");
    form.setStreet("alexa str.");
    form.setZipCode("19923");
    form.setHouseNumber("2a");
    form.setCaptcha("Y2HAhU5T");
    return form;
}
```

### 3.4.测试具有多个组的约束

我们可以为一个约束指定多个组。在我们的用例中，我们在基本和高级信息中都使用了`captcha`。我们先用`BasicInfo`来测试一下`captcha`:

```java
@Test
public void whenCaptchaIsBlank_thenShouldGiveConstraintViolationsForBasicInfo() {
    RegistrationForm form = buildRegistrationFormWithBasicInfo();
    form.setCaptcha("");

    Set<ConstraintViolation<RegistrationForm>> violations = validator.validate(form, BasicInfo.class);

    assertThat(violations.size()).isEqualTo(1);
    violations.forEach(action -> {
        assertThat(action.getMessage()).isEqualTo("must not be blank");
        assertThat(action.getPropertyPath().toString()).isEqualTo("captcha");
    });
}
```

现在让我们用`AdvanceInfo`来测试`captcha`:

```java
@Test
public void whenCaptchaIsBlank_thenShouldGiveConstraintViolationsForAdvanceInfo() {
    RegistrationForm form = buildRegistrationFormWithAdvanceInfo();
    form.setCaptcha("");

    Set<ConstraintViolation<RegistrationForm>> violations = validator.validate(form, AdvanceInfo.class);

    assertThat(violations.size()).isEqualTo(1);
    violations.forEach(action -> {
        assertThat(action.getMessage()).isEqualTo("must not be blank");
        assertThat(action.getPropertyPath().toString()).isEqualTo("captcha");
    });
}
```

## 4.用`GroupSequence`指定约束组验证顺序

默认情况下，约束组不以任何特定顺序进行计算。但是我们可能有一些用例，其中一些组应该在其他组之前被验证。为了实现这一点，我们可以使用`GroupSequence. `**指定组验证的顺序**

有两种方法可以使用`GroupSequence`注释:

*   在被验证的实体上
*   在一个`Interface`

### 4.1.在被验证的实体上使用`GroupSequence`

这是一种简单的约束排序方式。让我们用``GroupSequence`` 标注实体，并指定约束的顺序:

```java
@GroupSequence({BasicInfo.class, AdvanceInfo.class})
public class RegistrationForm {
    @NotBlank(groups = BasicInfo.class)
    private String firstName;
    @NotBlank(groups = AdvanceInfo.class)
    private String street;
}
```

### 4.2.在界面上使用`GroupSequence`

我们还可以使用`interface`来指定约束验证的顺序。这种方法的优点是相同的序列可以用于其他实体。让我们看看如何将`GroupSequence`与我们上面定义的接口一起使用:

```java
@GroupSequence({BasicInfo.class, AdvanceInfo.class})
public interface CompleteInfo {
}
```

### 4.3.测试`GroupSequence`

现在让我们先测试`GroupSequence.` ，我们将测试如果`BasicInfo`不完整，那么`AdvanceInfo` 组约束将不会被求值:

```java
@Test
public void whenBasicInfoIsNotComplete_thenShouldGiveConstraintViolationsForBasicInfoOnly() {
    RegistrationForm form = buildRegistrationFormWithBasicInfo();
    form.setFirstName("");

    Set<ConstraintViolation<RegistrationForm>> violations = validator.validate(form, CompleteInfo.class);

    assertThat(violations.size()).isEqualTo(1);
    violations.forEach(action -> {
        assertThat(action.getMessage()).isEqualTo("must not be blank");
        assertThat(action.getPropertyPath().toString()).isEqualTo("firstName");
    });
}
```

接下来，测试当`BasicInfo`完成时，应该评估`AdvanceInfo`约束:

```java
@Test
public void whenBasicAndAdvanceInfoIsComplete_thenShouldNotGiveConstraintViolationsWithCompleteInfoValidationGroup() {
    RegistrationForm form = buildRegistrationFormWithBasicAndAdvanceInfo();

    Set<ConstraintViolation<RegistrationForm>> violations = validator.validate(form, CompleteInfo.class);

    assertThat(violations.size()).isEqualTo(0);
}
```

## 5.结论

在这个快速教程中，我们看到了如何对`javax.validation`约束进行分组。

像往常一样，GitHub 上的所有代码片段[都是可用的。](https://web.archive.org/web/20220628130910/https://github.com/eugenp/tutorials/tree/master/javaxval)