# Javax BigDecimal 验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javax-bigdecimal-validation>

## 1.介绍

在教程 [Java Bean 验证基础](/web/20221129011650/https://www.baeldung.com/javax-validation)中，我们看到了如何将基本的`javax`验证应用于各种类型，在本教程中，我们将重点关注将`javax`验证与`BigDecimal`一起使用。

## 2.验证`BigDecimal`实例

不幸的是，**与`BigDecimal`，我们不能使用经典的`@Min`或`@Max` javax 注释。**

幸运的是，我们有一套专门的注释来使用它们:

*   `@DecimalMin`

*   `@Digits`

*   `@DecimalMax`

`BigDecimal`是不是[因为精度高](/web/20221129011650/https://www.baeldung.com/java-bigdecimal-biginteger)而成为金融计算的首选。

让我们看看我们的`Invoice`类，它有一个类型为`BigDecimal`的字段:

```
public class Invoice {

    @DecimalMin(value = "0.0", inclusive = false)
    @Digits(integer=3, fraction=2)
    private BigDecimal price;
    private String description;

    public Invoice(BigDecimal price, String description) {
        this.price = price;
        this.description = description;
    }
}
```

### 2.1.`@DecimalMin`

**带注释的元素必须是一个大于或等于指定最小值的数字。** `@DecimalMin`有一个属性`inclusive`，表示指定的最小值是包含还是排除。

### 2.2.`@DecimalMax`

`@DecimalMax`是`@DecimalMin`的对应物。带注释的元素必须是一个小于或等于指定最大值的数字。`@DecimalMax`有一个`inclusive`属性，指定指定的最大值是包含还是排除。

另外，`@Min`和`@Max`只接受`long`值。在`@DecimalMin`和`@DecimalMax`中，我们可以指定`string`格式的值，可以是任意数值类型。

### 2.3.`@Digits`

在许多情况下，我们需要验证一个`decimal`数字的`integral`部分和`fraction`部分的位数。

**`@Digit`注释有两个属性，`integer`和`fraction`，用于指定数字`.`的`integral`部分和`fraction` 部分允许的位数**

根据[官方文件](https://web.archive.org/web/20221129011650/https://docs.oracle.com/javaee/7/api/javax/validation/constraints/Digits.html)，`integer`允许我们指定该数字接受的`integral`位数的最大值**。**

类似地，`fraction`属性允许我们指定这个数字接受的`fractional`位数的最大值**。**

### 2.4.测试案例

让我们看看这些注释的作用。

首先，我们将添加一个测试，根据我们的验证创建一个价格无效的发票，并检查验证是否会失败:

```
public class InvoiceUnitTest {

    private static Validator validator;

    @BeforeClass
    public static void setupValidatorInstance() {
        validator = Validation.buildDefaultValidatorFactory().getValidator();
    }

    @Test
    public void whenMoreThanThreeIntegerDigits_thenShouldGiveConstraintViolations() {
        Invoice invoice = new Invoice(new BigDecimal("1021.21"), "Book purchased");
        Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
        assertThat(violations).hasSize(1);
        assertThat(violations)
            .extracting("message")
            .containsOnly("numeric value out of bounds (<3 digits>.<2 digits> expected)");
    }
}
```

现在让我们用正确的价格检查验证:

```
@Test
public void whenLessThanThreeIntegerDigits_thenShouldNotGiveConstraintViolations() {
    Invoice invoice = new Invoice(new BigDecimal("10.21"), "Book purchased");
    Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
    assertThat(violations).isEmpty();
}
```

同样，让我们看看小数部分的验证是如何工作的:

```
@Test
public void whenTwoFractionDigits_thenShouldNotGiveConstraintViolations() {
    Invoice invoice = new Invoice(new BigDecimal("99.99"), "Book purchased");
    Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
    assertThat(violations).isEmpty();
}

@Test
public void whenMoreThanTwoFractionDigits_thenShouldGiveConstraintViolations() {
    Invoice invoice = new Invoice(new BigDecimal("99.999"), "Book purchased");
    Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
    assertThat(violations).hasSize(1);
    assertThat(violations)
        .extracting("message")
        .containsOnly("numeric value out of bounds (<3 digits>.<2 digits> expected)");
}
```

等于 0.00 的价格应该违反我们的约束:

```
@Test
public void whenPriceIsZero_thenShouldGiveConstraintViolations() {
    Invoice invoice = new Invoice(new BigDecimal("0.00"), "Book purchased");
    Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
    assertThat(violations).hasSize(1);
    assertThat(violations)
        .extracting("message")
        .containsOnly("must be greater than 0.0");
}
```

最后，让我们看看价格大于零的情况:

```
@Test
public void whenPriceIsGreaterThanZero_thenShouldNotGiveConstraintViolations() {
    Invoice invoice = new Invoice(new BigDecimal("100.50"), "Book purchased");
    Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
    assertThat(violations).isEmpty();
}
```

## 3.结论

在本文中，我们看到了如何对 BigDecimal 使用`javax`验证。

所有代码片段都可以在 GitHub 上找到。