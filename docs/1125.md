# 用 FreeBuilder 自动生成构建器模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-builder-pattern-freebuilder>

## 1.概观

在本教程中，我们将使用 [FreeBuilder 库](https://web.archive.org/web/20220628155456/https://freebuilder.inferred.org/)在 Java 中生成构建器类。

## 2.生成器设计模式

Builder 是面向对象语言中使用最广泛的[创建设计模式](/web/20220628155456/https://www.baeldung.com/creational-design-patterns)之一。它**抽象了复杂领域对象的实例化，并为创建实例提供了一个流畅的 API** 。因此，它有助于维护一个简洁的领域层。

尽管构建器很有用，但实现起来通常很复杂，尤其是在 Java 中。甚至更简单的值对象也需要大量样板代码。

## 3.Java 中的生成器实现

在我们继续 FreeBuilder 之前，让我们为我们的`Employee `类实现一个样板生成器:

```
public class Employee {

    private final String name;
    private final int age;
    private final String department;

    private Employee(String name, int age, String department) {
        this.name = name;
        this.age = age;
        this.department = department;
    }
}
```

和一个内部的`Builder `类:

```
public static class Builder {

    private String name;
    private int age;
    private String department;

    public Builder setName(String name) {
        this.name = name;
        return this;
    }

    public Builder setAge(int age) {
        this.age = age;
        return this;
    }

    public Builder setDepartment(String department) {
        this.department = department;
        return this;
    }

    public Employee build() {
        return new Employee(name, age, department);
    }
}
```

因此，我们现在可以使用构建器来实例化`Employee `对象:

```
Employee.Builder emplBuilder = new Employee.Builder();

Employee employee = emplBuilder
  .setName("baeldung")
  .setAge(12)
  .setDepartment("Builder Pattern")
  .build();
```

如上所示，**实现一个构建器类需要大量的样板代码。**

在后面的章节中，我们将看到 FreeBuilder 如何立即简化这个实现。

## 4.Maven 依赖性

为了添加 FreeBuilder 库，我们将在我们的`pom.xml`中添加 [FreeBuilder Maven 依赖项](https://web.archive.org/web/20220628155456/https://search.maven.org/search?q=g:org.inferred%20AND%20a:freebuilder&core=gav):

```
<dependency>
    <groupId>org.inferred</groupId>
    <artifactId>freebuilder</artifactId>
    <version>2.4.1</version>
</dependency>
```

## 5.`FreeBuilder`注释

### 5.1.生成构建器

FreeBuilder 是一个开源库，帮助开发人员在实现构建器类时避免样板代码。它利用 Java 中的注释处理来生成构建器模式的具体实现。

我们将**用`@`** `**FreeBuilder** `注释前面章节中的`Employee `类，看看它如何自动生成构建器类:

```
@FreeBuilder
public interface Employee {

    String name();
    int age();
    String department();

    class Builder extends Employee_Builder {
    }
}
```

需要指出的是， **`Employee `现在是一个** `**interface** `而不是一个 POJO 类。此外，它包含了一个`Employee `对象的所有属性作为方法。

在我们继续使用这个构建器之前，我们必须配置我们的 ide 以避免任何编译问题。由于`FreeBuilder `在编译时自动生成`Employee_Builder `类，IDE **通常会报错第 8 行**的`ClassNotFoundException`。

为了避免这样的问题，**我们需要在 [IntelliJ](https://web.archive.org/web/20220628155456/https://www.jetbrains.com/help/idea/configuring-annotation-processing.html) 或 [Eclipse](https://web.archive.org/web/20220628155456/https://help.eclipse.org/kepler/index.jsp?topic=%2Forg.eclipse.jdt.doc.isv%2Fguide%2Fjdt_apt_getting_started.htm)** 中启用注释处理。同时，我们将使用 FreeBuilder 的注释处理器`org.inferred.freebuilder.processor.Processor.` 另外，用于生成这些源文件的目录应该标记为 [Generated Sources Root](https://web.archive.org/web/20220628155456/https://www.jetbrains.com/help/idea/content-roots.html) 。

或者，**我们也可以执行`mvn install`** 来构建项目并生成所需的构建器类。

最后，我们已经编译了我们的项目，现在可以使用`Employee.Builder `类:

```
Employee.Builder builder = new Employee.Builder();

Employee employee = builder.name("baeldung")
  .age(10)
  .department("Builder Pattern")
  .build();
```

总而言之，这和我们之前看到的构建器类有两个主要区别。首先，**我们必须为`Employee `类的所有属性设置值。否则，它抛出一个** `**IllegalStateException**.`

我们将在后面的章节中看到 FreeBuilder 如何处理可选属性。

其次，`Employee.Builder `的方法名不遵循 JavaBean 命名约定。我们将在下一节看到这一点。

### 5.2.JavaBean 命名约定

为了强制 FreeBuilder 遵循 JavaBean 命名约定，我们必须**在`Employee `中重命名我们的方法，并在方法前加上前缀** `**get**:`

```
@FreeBuilder
public interface Employee {

    String getName();
    int getAge();
    String getDepartment();

    class Builder extends Employee_Builder {
    }
}
```

这将生成遵循 JavaBean 命名约定的 getters 和 setters:

```
Employee employee = builder
  .setName("baeldung")
  .setAge(10)
  .setDepartment("Builder Pattern")
  .build();
```

### 5.3.映射方法

加上 getters 和 setters，FreeBuilder 还在 Builder 类中添加了 mapper 方法。这些映射器方法**接受一个[一元运算符](/web/20220628155456/https://www.baeldung.com/java-8-functional-interfaces)作为输入，**从而允许开发人员计算复杂的字段值。

假设我们的`Employee` 类也有一个薪水字段:

```
@FreeBuilder
public interface Employee {
    Optional<Double> getSalaryInUSD();
}
```

现在假设我们需要转换作为输入提供的工资货币:

```
long salaryInEuros = INPUT_SALARY_EUROS;
Employee.Builder builder = new Employee.Builder();

Employee employee = builder
  .setName("baeldung")
  .setAge(10)
  .mapSalaryInUSD(sal -> salaryInEuros * EUROS_TO_USD_RATIO)
  .build();
```

FreeBuilder 为所有字段提供了这样的映射器方法。

## 6.默认值和约束检查

### 6.1.设置默认值

到目前为止，我们讨论的`Employee.Builder`实现期望客户端传递所有字段的值。事实上，在缺少字段的情况下，它会通过一个`IllegalStateException `使初始化过程失败。

为了避免这样的失败，**我们可以为字段设置默认值或者使它们可选**。

我们可以在`Employee.Builder `构造函数中设置默认值:

```
@FreeBuilder
public interface Employee {

    // getter methods

    class Builder extends Employee_Builder {

        public Builder() {
            setDepartment("Builder Pattern");
        }
    }
}
```

所以我们只需在构造函数中设置默认的`department `。该值将应用于所有`Employee `对象。

### 6.2.约束检查

通常，我们对字段值有一定的约束。例如，一封有效的电子邮件必须包含一个“@”，或者一个`Employee `的年龄必须在一个范围内。

这种约束要求我们对输入值进行验证。并且 **FreeBuilder 允许我们仅仅通过覆盖`setter `方法**来添加这些验证:

```
@FreeBuilder
public interface Employee {

    // getter methods

    class Builder extends Employee_Builder {

        @Override
        public Builder setEmail(String email) {
            if (checkValidEmail(email))
                return super.setEmail(email);
            else
                throw new IllegalArgumentException("Invalid email");

        }

        private boolean checkValidEmail(String email) {
            return email.contains("@");
        }
    }
}
```

## 7.可选值

### 7.1.使用`Optional`字段

一些对象包含可选字段，其值可以为空。 **FreeBuilder 允许我们使用 [Java `Optional`](/web/20220628155456/https://www.baeldung.com/java-optional) 类型**来定义这样的字段:

```
@FreeBuilder
public interface Employee {

    String getName();
    int getAge();

    // other getters

    Optional<Boolean> getPermanent();

    Optional<String> getDateOfJoining();

    class Builder extends Employee_Builder {
    }
}
```

现在我们可以跳过为`Optional `字段提供任何值:

```
Employee employee = builder.setName("baeldung")
  .setAge(10)
  .setPermanent(true)
  .build();
```

值得注意的是，我们只是传递了`permanent` 字段的值，而不是`Optional. `，因为我们没有为`dateOfJoining `字段设置**的值，所以它将是`Optional.empty() `，这是`Optional `字段的默认值。**

### 7.2.使用`@Nullable` 字段

尽管在 Java 中建议使用`Optional` 来处理`null` s，FreeBuilder 允许 **us 使用`[@Nullable](/web/20220628155456/https://www.baeldung.com/java-avoid-null-check)` 来向后兼容**:

```
@FreeBuilder
public interface Employee {

    String getName();
    int getAge();

    // other getter methods

    Optional<Boolean> getPermanent();
    Optional<String> getDateOfJoining();

    @Nullable String getCurrentProject();

    class Builder extends Employee_Builder {
    }
}
```

在某些情况下使用`[Optional](/web/20220628155456/https://www.baeldung.com/java-optional-return)` [是不明智的](/web/20220628155456/https://www.baeldung.com/java-optional-return)，这也是为什么`@Nullable `更适合构建器类的另一个原因。

## 8.收藏品和地图

FreeBuilder 特别支持收藏和地图:

```
@FreeBuilder
public interface Employee {

    String getName();
    int getAge();

    // other getter methods

    List<Long> getAccessTokens();
    Map<String, Long> getAssetsSerialIdMapping();

    class Builder extends Employee_Builder {
    }
}
```

FreeBuilder 添加了**方便的方法，将输入元素添加到 Builder 类**的集合中:

```
Employee employee = builder.setName("baeldung")
  .setAge(10)
  .addAccessTokens(1221819L)
  .addAccessTokens(1223441L, 134567L)
  .build();
```

builder 类中还有一个`getAccessTokens() `方法，**返回一个不可修改的列表**。同样，对于`Map:`

```
Employee employee = builder.setName("baeldung")
  .setAge(10)
  .addAccessTokens(1221819L)
  .addAccessTokens(1223441L, 134567L)
  .putAssetsSerialIdMapping("Laptop", 12345L)
  .build();
```

`Map `的`getter `方法也**返回一个不可修改的映射**给客户端代码。

## 9.嵌套生成器

对于现实世界的应用程序，我们可能不得不为我们的域实体嵌套很多值对象。由于嵌套对象本身需要构建器实现，FreeBuilder 允许嵌套的可构建类型。

例如，假设我们在`Employee `类中有一个嵌套的复杂类型`Address `:

```
@FreeBuilder
public interface Address {

    String getCity();

    class Builder extends Address_Builder {
    }
}
```

现在，FreeBuilder 生成了将`Address.Builder`作为输入的`setter `方法和`Address `类型:

```
Address.Builder addressBuilder = new Address.Builder();
addressBuilder.setCity(CITY_NAME);

Employee employee = builder.setName("baeldung")
  .setAddress(addressBuilder)
  .build();
```

值得注意的是，FreeBuilder 还增加了一个方法来**自定义** `**Employee**:`中已有的`Address `对象

```
Employee employee = builder.setName("baeldung")
  .setAddress(addressBuilder)
  .mutateAddress(a -> a.setPinCode(112200))
  .build();
```

除了`FreeBuilder `类型，FreeBuilder 还允许嵌套其他构建器，如 [protos](/web/20220628155456/https://www.baeldung.com/google-protocol-buffer) 。

## 10.构建部分对象

正如我们之前讨论过的，FreeBuilder 会对任何违反约束的情况抛出一个`IllegalStateException `——例如，强制字段缺少值。

尽管这是生产环境所期望的**，但是它使得独立于一般约束的**单元测试变得复杂**。**

为了放松这些限制，FreeBuilder 允许我们构建部分对象:

```
Employee employee = builder.setName("baeldung")
  .setAge(10)
  .setEmail("[[email protected]](/web/20220628155456/https://www.baeldung.com/cdn-cgi/l/email-protection)")
  .buildPartial();

assertNotNull(employee.getEmail());
```

因此，即使我们没有为`Employee`设置所有的强制字段，我们仍然可以验证`email `字段是否有有效值。

## 11.自定义`toString() `方法

对于值对象，**我们经常需要添加一个自定义的`toString() `实现。** FreeBuilder 通过`abstract `类允许这样做:

```
@FreeBuilder
public abstract class Employee {

    abstract String getName();

    abstract int getAge();

    @Override
    public String toString() {
        return getName() + " (" + getAge() + " years old)";
    }

    public static class Builder extends Employee_Builder{
    }
}
```

我们将`Employee `声明为抽象类而不是接口，并提供了自定义的`toString() `实现。

## 12.与其他构建器库的比较

我们在本文中讨论的构建器实现与那些 [Lombok](/web/20220628155456/https://www.baeldung.com/lombok-builder) 、 [Immutables](/web/20220628155456/https://www.baeldung.com/immutables) 或任何其他[注释处理器](/web/20220628155456/https://www.baeldung.com/java-annotation-processing-builder)非常相似。然而，**有几个我们已经讨论过的显著特征**:

## 13.结论

在本文中，我们使用 FreeBuilder 库在 Java 中生成了一个构建器类。我们在注释**的帮助下实现了构建器类的各种定制，从而减少了实现**所需的样板代码。

我们还看到了 FreeBuilder 与其他一些库的不同之处，并在本文中简要讨论了其中的一些特性。

所有的代码示例都可以在 [GitHub](https://web.archive.org/web/20220628155456/https://github.com/eugenp/tutorials/tree/master/patterns/design-patterns-creational) 上找到。