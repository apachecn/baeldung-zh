# 带有 Bean 验证的约束组合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bean-validation-constraint-composition>

## 1.概观

在本教程中，我们将讨论 [Bean 验证](/web/20220811173529/https://www.baeldung.com/javax-validation)的约束组合。

**将多个约束组合在一个单独的自定义注释下可以减少代码重复，提高可读性**。我们将看到如何创建组合约束，以及如何根据我们的需要定制它们。

对于代码示例，我们将拥有与 [Java Bean 验证基础](/web/20220811173529/https://www.baeldung.com/javax-validation)中相同的依赖关系。

## 2.理解问题

首先，让我们熟悉一下数据模型。对于本文中的大多数示例，我们将使用`Account` 类:

```java
public class Account {

    @NotNull
    @Pattern(regexp = ".*\\d.*", message = "must contain at least one numeric character")
    @Length(min = 6, max = 32, message = "must have between 6 and 32 characters")
    private String username;

    @NotNull
    @Pattern(regexp = ".*\\d.*", message = "must contain at least one numeric character")
    @Length(min = 6, max = 32, message = "must have between 6 and 32 characters")
    private String nickname;

    @NotNull
    @Pattern(regexp = ".*\\d.*", message = "must contain at least one numeric character")
    @Length(min = 6, max = 32, message = "must have between 6 and 32 characters")
    private String password;

    // getters and setters
}
```

我们可以注意到,`@NotNull, @Pattern,` 和`@Length`约束组对三个字段中的每一个都重复。

**此外，如果这些字段中的一个出现在不同层的多个类中，约束应该匹配——导致更多的代码重复**。

例如，我们可以想象在一个 DTO 对象中有一个`username`字段和一个`@Entity`模型。

## 3.创建组合约束

我们可以通过将三个约束组合在一个带有合适名称的自定义注释下来避免代码重复:

```java
@NotNull
@Pattern(regexp = ".*\\d.*", message = "must contain at least one numeric character")
@Length(min = 6, max = 32, message = "must have between 6 and 32 characters")
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {})
public @interface ValidAlphanumeric {

    String message() default "field should have a valid length and contain numeric character(s).";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

因此，我们现在可以使用`@ValidAlphanumeric` 来验证`Account`字段:

```java
public class Account {

    @ValidAlphanumeric
    private String username;

    @ValidAlphanumeric
    private String password;

    @ValidAlphanumeric
    private String nickname;

    // getters and setters
}
```

因此，我们可以测试`@ValidAlphanumeric`注释，并预期违反的约束一样多。

例如，如果我们将`username`设置为`“john”,` ,我们应该会遇到两个违例，因为它太短并且不包含数字字符:

```java
@Test
public void whenUsernameIsInvalid_validationShouldReturnTwoViolations() {
    Account account = new Account();
    account.setPassword("valid_password123");
    account.setNickname("valid_nickname123");
    account.setUsername("john");

    Set<ConstraintViolation<Account>> violations = validator.validate(account);

    assertThat(violations).hasSize(2);
}
```

## 4.使用`@ReportAsSingleViolation`

另一方面，**我们可能希望验证返回整个组**的单个`ConstraintViolation`。

为了实现这一点，我们必须用`@ReportAsSingleViolation`来注释我们的组合约束:

```java
@NotNull
@Pattern(regexp = ".*\\d.*", message = "must contain at least one numeric character")
@Length(min = 6, max = 32, message = "must have between 6 and 32 characters")
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {})
@ReportAsSingleViolation
public @interface ValidAlphanumericWithSingleViolation {

    String message() default "field should have a valid length and contain numeric character(s).";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

之后，我们可以使用`password`字段测试我们的新注释，并期待一个单独的违例:

```java
@Test
public void whenPasswordIsInvalid_validationShouldReturnSingleViolation() {
    Account account = new Account();
    account.setUsername("valid_username123");
    account.setNickname("valid_nickname123");
    account.setPassword("john");

    Set<ConstraintViolation<Account>> violations = validator.validate(account);

    assertThat(violations).hasSize(1);
} 
```

## 5.布尔约束合成

到目前为止，只有当所有的组合约束都有效时，验证才通过。这是因为 **`ConstraintComposition` 值默认为** `**CompositionType.AND**. `

然而，如果我们想检查是否至少有一个有效的约束，我们可以改变这种行为。

为此，我们需要将`ConstraintComposition` 切换到`CompositionType.` `OR`:

```java
@Pattern(regexp = ".*\\d.*", message = "must contain at least one numeric character")
@Length(min = 6, max = 32, message = "must have between 6 and 32 characters")
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {})
@ConstraintComposition(CompositionType.OR)
public @interface ValidLengthOrNumericCharacter {

    String message() default "field should have a valid length or contain numeric character(s).";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

例如，给定一个太短但至少有一个数字字符的值，应该没有冲突。

让我们使用模型中的`nickname`字段来测试这个新注释:

```java
@Test
public void whenNicknameIsTooShortButContainsNumericCharacter_validationShouldPass() {
    Account account = new Account();
    account.setUsername("valid_username123");
    account.setPassword("valid_password123");
    account.setNickname("doe1");

    Set<ConstraintViolation<Account>> violations = validator.validate(account);

    assertThat(violations).isEmpty();
}
```

类似地，如果我们想确保约束失效，我们可以**使用`CompositionType.` `ALL_FALSE `。**

## 6.使用组合约束进行方法验证

而且，我们可以使用组合约束作为[方法约束](/web/20220811173529/https://www.baeldung.com/javax-validation-method-constraints)。

为了验证方法的返回值，我们只需将`@SupportedValidationTarget(ValidationTarget.ANNOTATED_ELEMENT)` 添加到组合约束中:

```java
@NotNull
@Pattern(regexp = ".*\\d.*", message = "must contain at least one numeric character")
@Length(min = 6, max = 32, message = "must have between 6 and 32 characters")
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {})
@SupportedValidationTarget(ValidationTarget.ANNOTATED_ELEMENT)
public @interface AlphanumericReturnValue {

    String message() default "method return value should have a valid length and contain numeric character(s).";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

为了举例说明这一点，我们将使用`getAnInvalidAlphanumericValue` 方法，该方法用我们的自定义约束进行了注释:

```java
@Component
@Validated
public class AccountService {

    @AlphanumericReturnValue
    public String getAnInvalidAlphanumericValue() {
        return "john"; 
    }
}
```

现在，让我们调用这个方法，并期待抛出一个`ConstraintViolationException`:

```java
@Test
public void whenMethodReturnValuesIsInvalid_validationShouldFail() {
    assertThatThrownBy(() -> accountService.getAnInvalidAlphanumericValue())				 
      .isInstanceOf(ConstraintViolationException.class)
      .hasMessageContaining("must contain at least one numeric character")
      .hasMessageContaining("must have between 6 and 32 characters");
}
```

## 7.结论

在本文中，我们看到了如何使用组合约束来避免代码重复。

之后，我们学习了定制组合约束，使用布尔逻辑进行验证，返回单个约束违反，并应用于方法返回值。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220811173529/https://github.com/eugenp/tutorials/tree/master/javaxval)