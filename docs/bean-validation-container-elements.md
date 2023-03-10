# 使用 Bean Validation 2.0 验证容器元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/bean-validation-container-elements>

## 1。概述

2.0 版本的`Java Bean Validation`规范增加了几个新特性，其中之一是验证容器元素的可能性。

这个新功能利用了 Java 8 中引入的类型注释。因此，它需要 Java 版本 8 或更高版本才能工作。

**验证注释可以添加到容器中，例如集合、`Optional`对象以及其他内置和自定义容器。**

关于`Java Bean Validation`的介绍以及如何设置我们需要的`Maven`依赖项，请查看我们的[以前的文章](/web/20220627090448/https://www.baeldung.com/javax-validation)。

在接下来的几节中，我们将重点验证每种类型容器的元素。

## 2。集合元素

我们可以向类型为`java.util.Iterable`、`java.util.List`和`java.util.Map`的集合的元素添加验证注释。

让我们看一个验证列表元素的例子:

```java
public class Customer {    
     List<@NotBlank(message="Address must not be blank") String> addresses;

    // standard getters, setters 
}
```

在上面的例子中，我们已经为一个`Customer`类定义了一个`addresses`属性，它包含了不能为空的元素`Strings`。

注意，`@NotBlank`验证应用于`String`元素，而不是整个集合。如果集合为空，则不应用验证。

让我们验证一下，如果我们试图将一个空的`String`添加到`addresses`列表中，验证框架将返回一个`ConstraintViolation`:

```java
@Test
public void whenEmptyAddress_thenValidationFails() {
    Customer customer = new Customer();
    customer.setName("John");

    customer.setAddresses(Collections.singletonList(" "));
    Set<ConstraintViolation<Customer>> violations = 
      validator.validate(customer);

    assertEquals(1, violations.size());
    assertEquals("Address must not be blank", 
      violations.iterator().next().getMessage());
}
```

接下来，让我们看看如何验证类型为`Map`的集合的元素:

```java
public class CustomerMap {

    private Map<@Email String, @NotNull Customer> customers;

    // standard getters, setters
}
```

注意**我们可以为`Map`元素**的键和值添加验证注释。

让我们验证添加一个带有无效电子邮件的条目是否会导致验证错误:

```java
@Test
public void whenInvalidEmail_thenValidationFails() {
    CustomerMap map = new CustomerMap();
    map.setCustomers(Collections.singletonMap("john", new Customer()));
    Set<ConstraintViolation<CustomerMap>> violations
      = validator.validate(map);

    assertEquals(1, violations.size());
    assertEquals(
      "Must be a valid email", 
      violations.iterator().next().getMessage());
}
```

## 3。`Optional`价值观

验证约束也可以应用于`Optional`值:

```java
private Integer age;

public Optional<@Min(18) Integer> getAge() {
    return Optional.ofNullable(age);
}
```

让我们创建一个年龄太低的`Customer`,并验证这是否会导致验证错误:

```java
@Test
public void whenAgeTooLow_thenValidationFails() {
    Customer customer = new Customer();
    customer.setName("John");
    customer.setAge(15);
    Set<ConstraintViolation<Customer>> violations
      = validator.validate(customer);

    assertEquals(1, violations.size());
}
```

另一方面，如果`age`为空，则`Optional`值不被验证:

```java
@Test
public void whenAgeNull_thenValidationSucceeds() {
    Customer customer = new Customer();
    customer.setName("John");
    Set<ConstraintViolation<Customer>> violations
      = validator.validate(customer);

    assertEquals(0, violations.size());
}
```

## 4。非通用容器元素

除了为类型参数添加注释之外，我们还可以对非泛型容器应用验证，只要带有`@UnwrapByDefault`注释的类型有一个值提取器。

值提取器是从容器中提取值进行验证的类。

**参考实现包含`OptionalInt`、`OptionalLong`和`OptionalDouble`、**的取值器:

```java
@Min(1)
private OptionalInt numberOfOrders;
```

在这种情况下， `@Min`注释应用于包装的`Integer`值，而不是容器。

## 5。自定义容器元素

除了内置的值提取器，我们还可以定义自己的值提取器，并用容器类型注册它们。

这样，我们可以向自定义容器的元素添加验证注释。

让我们添加一个包含`companyName`属性的新的`Profile`类:

```java
public class Profile {
    private String companyName;

    // standard getters, setters 
}
```

接下来，我们想在`Customer`类中添加一个带有`@NotBlank`注释的`Profile`属性——它验证了`companyName`不是一个空的`String`:

```java
@NotBlank
private Profile profile;
```

要做到这一点，我们需要一个值提取器来确定应用于`companyName`属性的验证，而不是直接应用于 profile 对象。

让我们添加一个实现了`ValueExtractor`接口并覆盖了`extractValue()`方法的`ProfileValueExtractor`类:

```java
@UnwrapByDefault
public class ProfileValueExtractor 
  implements ValueExtractor<@ExtractedValue(type = String.class) Profile> {

    @Override
    public void extractValues(Profile originalValue, 
      ValueExtractor.ValueReceiver receiver) {
        receiver.value(null, originalValue.getCompanyName());
    }
}
```

这个类还需要指定使用`@ExtractedValue`注释提取的值的类型。

此外，我们还添加了**`@UnwrapByDefault`注释，指定验证应该应用于未包装的值，而不是容器**。

最后，我们需要通过向`META-INF/services`目录添加一个名为`javax.validation.valueextraction.ValueExtractor`的文件来注册该类，该文件包含我们的`ProfileValueExtractor`类的全名:

```java
org.baeldung.javaxval.container.validation.valueextractors.ProfileValueExtractor
```

现在，当我们用一个空的`companyName`来验证一个带有*配置文件*属性的`Customer`对象时，我们会看到一个验证错误:

```java
@Test
public void whenProfileCompanyNameBlank_thenValidationFails() {
    Customer customer = new Customer();
    customer.setName("John");
    Profile profile = new Profile();
    profile.setCompanyName(" ");
    customer.setProfile(profile);
    Set<ConstraintViolation<Customer>> violations
     = validator.validate(customer);

    assertEquals(1, violations.size());
}
```

注意，如果您使用的是`hibernate-validator-annotation-processor`，那么在自定义容器类中添加一个验证注释，当它被标记为`@UnwrapByDefault`时，将会导致 6.0.2 版本中的编译错误。

这是一个[已知问题](https://web.archive.org/web/20220627090448/http://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-annotationprocessor-known-issues)，可能会在未来版本中得到解决。

## 6。结论

在本文中，我们展示了如何使用`Java Bean Validation 2.0.`来验证几种类型的容器元素

你可以在 GitHub 上找到示例[的完整源代码。](https://web.archive.org/web/20220627090448/https://github.com/eugenp/tutorials/tree/master/javaxval)