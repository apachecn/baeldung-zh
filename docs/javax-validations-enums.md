# 枚举类型的验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javax-validations-enums>

## 1。简介

在教程 [Java Bean 验证基础](/web/20221102035442/https://www.baeldung.com/javax-validation)中，我们看到了如何使用 [JSR 380](https://web.archive.org/web/20221102035442/https://beanvalidation.org/2.0/) 对各种类型应用`javax` 验证。在教程 [Spring MVC 定制验证](/web/20221102035442/https://www.baeldung.com/spring-mvc-custom-validator)中，我们看到了如何创建定制验证。

在接下来的教程中，**我们将关注使用定制注释为枚举构建验证。**

## 2。验证枚举

**不幸的是，大多数标准注释不能应用于枚举**。

例如，当对一个枚举应用`@Pattern`注释时，我们会收到一个类似 Hibernate Validator 的错误:

```java
javax.validation.UnexpectedTypeException: HV000030: No validator could be found for constraint 
 'javax.validation.constraints.Pattern' validating type 'com.baeldung.javaxval.enums.demo.CustomerType'. 
 Check configuration for 'customerTypeMatchesPattern' 
```

实际上，唯一可以应用于 enum 的标准注释是`@NotNull`和`@Null.`

## 3。验证枚举的模式

**让我们从定义一个注释来验证枚举的模式开始:**

```java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = EnumNamePatternValidator.class)
public @interface EnumNamePattern {
    String regexp();
    String message() default "must match \"{regexp}\"";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

现在，我们可以简单地使用正则表达式将这个新注释添加到我们的`CustomerType`枚举中:

```java
@EnumNamePattern(regexp = "NEW|DEFAULT")
private CustomerType customerType;
```

**正如我们所看到的，注释实际上并不包含验证逻辑。因此，我们需要提供一个`ConstraintValidator:`**

```java
public class EnumNamePatternValidator implements ConstraintValidator<EnumNamePattern, Enum<?>> {
    private Pattern pattern;

    @Override
    public void initialize(EnumNamePattern annotation) {
        try {
            pattern = Pattern.compile(annotation.regexp());
        } catch (PatternSyntaxException e) {
            throw new IllegalArgumentException("Given regex is invalid", e);
        }
    }

    @Override
    public boolean isValid(Enum<?> value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;
        }

        Matcher m = pattern.matcher(value.name());
        return m.matches();
    }
}
```

在这个例子中，实现非常类似于标准的`@Pattern`验证器。**然而，这一次，我们匹配枚举的名称。**

## 4。验证枚举的子集

将枚举与正则表达式匹配不是类型安全的。相反，与枚举的实际值进行比较更有意义。

然而，由于注释的限制，这样的注释不能成为通用的。这是因为批注的参数只能是特定枚举的具体值，而不是枚举父类的实例。

让我们看看如何为我们的`CustomerType`枚举创建一个特定的子集验证注释:

```java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = CustomerTypeSubSetValidator.class)
public @interface CustomerTypeSubset {
    CustomerType[] anyOf();
    String message() default "must be any of {anyOf}";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

然后，该注释可以应用于类型`CustomerType`的枚举:

```java
@CustomerTypeSubset(anyOf = {CustomerType.NEW, CustomerType.OLD})
private CustomerType customerType;
```

**接下来，我们需要定义`CustomerTypeSubSetValidator`来检查给定枚举值的列表是否包含当前的**:

```java
public class CustomerTypeSubSetValidator implements ConstraintValidator<CustomerTypeSubset, CustomerType> {
    private CustomerType[] subset;

    @Override
    public void initialize(CustomerTypeSubset constraint) {
        this.subset = constraint.anyOf();
    }

    @Override
    public boolean isValid(CustomerType value, ConstraintValidatorContext context) {
        return value == null || Arrays.asList(subset).contains(value);
    }
}
```

虽然注释必须特定于某个枚举，但是我们当然可以在不同的验证器之间共享代码。

## 5。验证字符串是否匹配枚举的值

我们也可以做相反的事情，而不是验证一个枚举来匹配一个`String`。为此，我们可以创建一个注释来检查`String`对于特定的 enum 是否有效。

```java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = ValueOfEnumValidator.class)
public @interface ValueOfEnum {
    Class<? extends Enum<?>> enumClass();
    String message() default "must be any of enum {enumClass}";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

这个注释可以添加到一个`String`字段，我们可以传递任何枚举类。

```java
@ValueOfEnum(enumClass = CustomerType.class)
private String customerTypeString;
```

**让我们定义`ValueOfEnumValidator`来检查`String`(或任何`CharSequence)`是否包含在枚举**中:

```java
public class ValueOfEnumValidator implements ConstraintValidator<ValueOfEnum, CharSequence> {
    private List<String> acceptedValues;

    @Override
    public void initialize(ValueOfEnum annotation) {
        acceptedValues = Stream.of(annotation.enumClass().getEnumConstants())
                .map(Enum::name)
                .collect(Collectors.toList());
    }

    @Override
    public boolean isValid(CharSequence value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;
        }

        return acceptedValues.contains(value.toString());
    }
}
```

当处理 JSON 对象时，这种验证尤其有用。因为在将 JSON 对象中的错误值映射到枚举时，会出现以下异常:

```java
Cannot deserialize value of type CustomerType from String value 'UNDEFINED': value not one
 of declared Enum instance names: [...]
```

当然，我们可以处理这个异常。然而，这并不允许我们一次报告所有违规行为。

我们可以将值映射到一个`String`，而不是映射到一个 enum。然后，我们将使用我们的验证器来检查它是否匹配任何枚举值。

## 6。将所有这些整合在一起

我们现在可以使用任何新的验证来验证 beans。最重要的是，我们所有的验证都接受`null`值。因此，我们也可以将它与注释`@NotNull`结合起来:

```java
public class Customer {
    @ValueOfEnum(enumClass = CustomerType.class)
    private String customerTypeString;

    @NotNull
    @CustomerTypeSubset(anyOf = {CustomerType.NEW, CustomerType.OLD})
    private CustomerType customerTypeOfSubset;

    @EnumNamePattern(regexp = "NEW|DEFAULT")
    private CustomerType customerTypeMatchesPattern;

    // constructor, getters etc.
}
```

在下一节中，我们将看到如何测试我们的新注释。

## 7。测试我们的 Javax 枚举验证

为了测试我们的验证器，我们将设置一个验证器，它支持我们新定义的注释。我们将为所有的测试准备好豆子。

首先，我们希望确保有效的`Customer`实例不会导致任何违规:

```java
@Test 
public void whenAllAcceptable_thenShouldNotGiveConstraintViolations() { 
    Customer customer = new Customer(); 
    customer.setCustomerTypeOfSubset(CustomerType.NEW); 
    Set violations = validator.validate(customer); 
    assertThat(violations).isEmpty(); 
}
```

其次，我们希望我们的新注释支持并接受`null`值。我们只希望有一次违规。这应该由`@NotNull `注解在`customerTypeOfSubset` 上报告:

```java
@Test
public void whenAllNull_thenOnlyNotNullShouldGiveConstraintViolations() {
    Customer customer = new Customer();
    Set<ConstraintViolation> violations = validator.validate(customer);
    assertThat(violations.size()).isEqualTo(1);

    assertThat(violations)
      .anyMatch(havingPropertyPath("customerTypeOfSubset")
      .and(havingMessage("must not be null")));
}
```

最后，当输入无效时，我们验证我们的验证器来报告违规:

```java
@Test
public void whenAllInvalid_thenViolationsShouldBeReported() {
    Customer customer = new Customer();
    customer.setCustomerTypeString("invalid");
    customer.setCustomerTypeOfSubset(CustomerType.DEFAULT);
    customer.setCustomerTypeMatchesPattern(CustomerType.OLD);

    Set<ConstraintViolation> violations = validator.validate(customer);
    assertThat(violations.size()).isEqualTo(3);

    assertThat(violations)
      .anyMatch(havingPropertyPath("customerTypeString")
      .and(havingMessage("must be any of enum class com.baeldung.javaxval.enums.demo.CustomerType")));
    assertThat(violations)
      .anyMatch(havingPropertyPath("customerTypeOfSubset")
      .and(havingMessage("must be any of [NEW, OLD]")));
    assertThat(violations)
      .anyMatch(havingPropertyPath("customerTypeMatchesPattern")
      .and(havingMessage("must match \"NEW|DEFAULT\"")));
}
```

## 8。结论

在本教程中，我们介绍了使用定制注释和验证器来验证枚举的三个选项。

首先，我们学习了如何使用正则表达式来验证枚举的名称。

其次，我们讨论了特定枚举的值子集的验证。我们还解释了为什么我们不能构建一个通用的注释来做这件事。

最后，我们还看了如何为字符串构建一个验证器。以便检查`String`是否符合给定枚举的特定值。

和往常一样，Github 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221102035442/https://github.com/eugenp/tutorials/tree/master/javaxval)