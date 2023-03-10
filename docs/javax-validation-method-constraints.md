# Bean 验证 2.0 的方法约束

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javax-validation-method-constraints>

## 1。概述

在本文中，我们将讨论如何使用 Bean Validation 2.0 (JSR-380)来定义和验证方法约束。

在上一篇文章的[中，我们讨论了带有内置注释的 JSR-380，以及如何实现属性验证。](/web/20221004131734/https://www.baeldung.com/javax-validation)

这里，我们将重点关注不同类型的方法约束，例如:

*   单参数约束
*   交叉参数
*   退货限制

此外，我们将了解如何使用 Spring Validator 手动和自动验证约束。

对于下面的例子，我们需要与 [Java Bean 验证基础](/web/20221004131734/https://www.baeldung.com/javax-validation)中完全相同的依赖关系。

## 2。方法约束声明

首先，**我们将首先讨论如何声明方法参数的约束和方法**的返回值。

如前所述，我们可以使用来自`javax.validation.constraints`的注释，但是我们也可以指定自定义约束(例如自定义约束或交叉参数约束)。

### 2.1。单参数约束

定义单个参数的约束很简单。**我们只需根据需要为每个参数添加注释**:

```java
public void createReservation(@NotNull @Future LocalDate begin,
  @Min(1) int duration, @NotNull Customer customer) {

    // ...
}
```

同样，我们可以对构造函数使用相同的方法:

```java
public class Customer {

    public Customer(@Size(min = 5, max = 200) @NotNull String firstName, 
      @Size(min = 5, max = 200) @NotNull String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    // properties, getters, and setters
}
```

### 2.2。使用交叉参数约束

在某些情况下，我们可能需要一次验证多个值，例如，两个数字量一个比另一个大。

对于这些场景，我们可以定义自定义的交叉参数约束，这可能依赖于两个或更多的参数。

**交叉参数约束可以认为是等价于类级约束**的方法验证。我们可以使用这两者来实现基于几个属性的验证。

让我们考虑一个简单的例子:上一节中的`createReservation()`方法的一个变体接受两个类型为`LocalDate:`的参数，一个开始日期和一个结束日期。

因此，我们想确定`begin`是在未来，而`end`是在`begin`之后。与前面的例子不同，我们不能使用单参数约束来定义它。

相反，我们需要一个交叉参数约束。

与单参数约束相反，**交叉参数约束是在方法或构造函数**上声明的:

```java
@ConsistentDateParameters
public void createReservation(LocalDate begin, 
  LocalDate end, Customer customer) {

    // ...
}
```

### 2.3。创建交叉参数约束

为了实现`@ConsistentDateParameters`约束，我们需要两步。

首先，我们需要**定义约束注释**:

```java
@Constraint(validatedBy = ConsistentDateParameterValidator.class)
@Target({ METHOD, CONSTRUCTOR })
@Retention(RUNTIME)
@Documented
public @interface ConsistentDateParameters {

    String message() default
      "End date must be after begin date and both must be in the future";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

这里，这三个属性对于约束注释是必需的:

*   `message –` 返回创建错误消息的默认键，这使我们能够使用消息插值
*   `groups`–允许我们为约束条件指定验证组
*   `payload`–可由 Bean 验证 API 的客户端使用，以将自定义有效负载对象分配给约束

有关如何定义自定义约束的详细信息，请查看官方文档。

之后，我们可以定义验证器类:

```java
@SupportedValidationTarget(ValidationTarget.PARAMETERS)
public class ConsistentDateParameterValidator 
  implements ConstraintValidator<ConsistentDateParameters, Object[]> {

    @Override
    public boolean isValid(
      Object[] value, 
      ConstraintValidatorContext context) {

        if (value[0] == null || value[1] == null) {
            return true;
        }

        if (!(value[0] instanceof LocalDate) 
          || !(value[1] instanceof LocalDate)) {
            throw new IllegalArgumentException(
              "Illegal method signature, expected two parameters of type LocalDate.");
        }

        return ((LocalDate) value[0]).isAfter(LocalDate.now()) 
          && ((LocalDate) value[0]).isBefore((LocalDate) value[1]);
    }
}
```

正如我们所见，`isValid()` 方法包含了实际的验证逻辑。首先，我们确保得到两个类型为`LocalDate.`的参数。之后，我们检查这两个参数是否都在未来，以及`end`是否在`begin`之后。

另外，需要注意的是，`ConsistentDateParameterValidator`类上的`@SupportedValidationTarget(ValidationTarget**.**PARAMETERS)`注释是必需的。这样做的原因是因为`@ConsistentDateParameter`是在方法级设置的，但是约束应该应用于方法参数(而不是方法的返回值，我们将在下一节讨论)。

注意:Bean 验证规范建议将`null`-值视为有效。如果`null`不是一个有效值，应该使用`@NotNull`-注释。

### 2.4。返回值约束

有时我们需要验证一个方法返回的对象。为此，我们可以使用返回值约束。

以下示例使用内置约束:

```java
public class ReservationManagement {

    @NotNull
    @Size(min = 1)
    public List<@NotNull Customer> getAllCustomers() {
        return null;
    }
}
```

对于`getAllCustomers()`，以下约束适用:

*   首先，返回的列表不能是`null`，并且必须至少有一个条目
*   此外，列表不得包含`null`条目

### 2.5。返回值自定义约束

在某些情况下，我们可能还需要验证复杂的对象:

```java
public class ReservationManagement {

    @ValidReservation
    public Reservation getReservationsById(int id) {
        return null;
    }
}
```

在这个例子中，一个返回的`Reservation`对象必须满足由`@ValidReservation`定义的约束，我们接下来将定义它。

同样，**我们首先必须定义约束注释**:

```java
@Constraint(validatedBy = ValidReservationValidator.class)
@Target({ METHOD, CONSTRUCTOR })
@Retention(RUNTIME)
@Documented
public @interface ValidReservation {
    String message() default "End date must be after begin date "
      + "and both must be in the future, room number must be bigger than 0";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

之后，我们定义验证器类:

```java
public class ValidReservationValidator
  implements ConstraintValidator<ValidReservation, Reservation> {

    @Override
    public boolean isValid(
      Reservation reservation, ConstraintValidatorContext context) {

        if (reservation == null) {
            return true;
        }

        if (!(reservation instanceof Reservation)) {
            throw new IllegalArgumentException("Illegal method signature, "
            + "expected parameter of type Reservation.");
        }

        if (reservation.getBegin() == null
          || reservation.getEnd() == null
          || reservation.getCustomer() == null) {
            return false;
        }

        return (reservation.getBegin().isAfter(LocalDate.now())
          && reservation.getBegin().isBefore(reservation.getEnd())
          && reservation.getRoom() > 0);
    }
}
```

### 2.6。构造函数中的返回值

正如我们之前在`ValidReservation`接口中将`METHOD`和`CONSTRUCTOR`定义为`target`一样，**我们也可以注释`Reservation`的构造函数来验证构造的实例**:

```java
public class Reservation {

    @ValidReservation
    public Reservation(
      LocalDate begin, 
      LocalDate end, 
      Customer customer, 
      int room) {
        this.begin = begin;
        this.end = end;
        this.customer = customer;
        this.room = room;
    }

    // properties, getters, and setters
}
```

### 2.7。级联验证

最后，Bean Validation API 不仅允许我们验证单个对象，还允许我们使用所谓的级联验证来验证对象图。

**因此，如果我们想要验证复杂的对象**，我们可以使用`@Valid`进行级联验证。这适用于方法参数和返回值。

让我们假设我们有一个带有一些属性约束的`Customer`类:

```java
public class Customer {

    @Size(min = 5, max = 200)
    private String firstName;

    @Size(min = 5, max = 200)
    private String lastName;

    // constructor, getters and setters
}
```

一个`Reservation`类可能有一个`Customer`属性，以及更多带有约束的属性:

```java
public class Reservation {

    @Valid
    private Customer customer;

    @Positive
    private int room;

    // further properties, constructor, getters and setters
}
```

如果我们现在引用`Reservation`作为方法参数，**我们可以强制递归验证所有属性**:

```java
public void createNewCustomer(@Valid Reservation reservation) {
    // ...
}
```

正如我们所见，我们在两个地方使用了`@Valid`:

*   关于`reservation`-参数:当`createNewCustomer()`被调用时，触发`Reservation`-对象的验证
*   因为我们在这里有一个嵌套的对象图，我们还必须在`customer`-属性上添加一个`@Valid`:因此，它触发这个嵌套属性的验证

这也适用于返回类型为 *Reservation* 的对象的方法:

```java
@Valid
public Reservation getReservationById(int id) {
    return null;
}
```

## 3。验证方法约束

在前一节中声明了约束之后，我们现在可以开始实际验证这些约束了。为此，我们有多种方法。

### 3.1。使用弹簧自动验证

Spring Validation 提供了与 Hibernate Validator 的集成。

注意:Spring 验证是基于 AOP 的，并且使用 Spring AOP 作为默认实现。因此，验证只对方法有效，对构造函数无效。

如果我们现在希望 Spring 自动验证我们的约束，我们必须做两件事:

**首先，我们要用`@Validated`** 对需要验证的 beans 进行注释:

```java
@Validated
public class ReservationManagement {

    public void createReservation(@NotNull @Future LocalDate begin, 
      @Min(1) int duration, @NotNull Customer customer){

        // ...
    }

    @NotNull
    @Size(min = 1)
    public List<@NotNull Customer> getAllCustomers(){
        return null;
    }
}
```

**其次，我们要提供一个`MethodValidationPostProcessor` bean:**

```java
@Configuration
@ComponentScan({ "org.baeldung.javaxval.methodvalidation.model" })
public class MethodValidationConfig {

    @Bean
    public MethodValidationPostProcessor methodValidationPostProcessor() {
        return new MethodValidationPostProcessor();
    }
}
```

如果违反了约束，容器现在将抛出一个`javax.validation.ConstraintViolationException`。

如果我们使用 Spring Boot，只要`hibernate-validator`在类路径中，容器就会为我们注册一个`MethodValidationPostProcessor` bean。

### 3.2。使用 CDI 自动验证(JSR-365)

从版本 1.1 开始，Bean 验证与 CDI(Jakarta EE 的上下文和依赖注入)一起工作。

如果我们的应用程序运行在 Jakarta EE 容器中，容器将在调用时自动验证方法约束。

### 3.3。程序验证

对于独立 Java 应用程序中的**手动方法验证，我们可以使用`javax.validation.executable.ExecutableValidator`接口。**

我们可以使用以下代码检索实例:

```java
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
ExecutableValidator executableValidator = factory.getValidator().forExecutables();
```

ExecutableValidator 提供了四种方法:

*   `validateParameters()`和`validateReturnValue()`用于方法验证
*   用于构造器验证的`validateConstructorParameters()`和`validateConstructorReturnValue()`

验证我们的第一个方法`createReservation()`的参数如下所示:

```java
ReservationManagement object = new ReservationManagement();
Method method = ReservationManagement.class
  .getMethod("createReservation", LocalDate.class, int.class, Customer.class);
Object[] parameterValues = { LocalDate.now(), 0, null };
Set<ConstraintViolation<ReservationManagement>> violations 
  = executableValidator.validateParameters(object, method, parameterValues);
```

注意:官方文档不鼓励直接从应用程序代码中调用这个接口，而是通过方法拦截技术来使用它，比如 AOP 或代理。

如果你对如何使用`ExecutableValidator`界面感兴趣，可以看看[官方文档](https://web.archive.org/web/20221004131734/http://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#section-validating-executable-constraints)。

## 4。结论

在本教程中，我们快速浏览了如何在 Hibernate Validator 中使用方法约束，还讨论了 JSR-380 的一些新特性。

首先，我们讨论了如何声明不同类型的约束:

*   单参数约束
*   交叉参数
*   返回值约束

我们还了解了如何使用 Spring Validator 手动和自动验证约束。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221004131734/https://github.com/eugenp/tutorials/tree/master/javaxval)