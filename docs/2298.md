# POJO、JavaBeans、DTO 和 VO 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pojo-javabeans-dto-vo>

## 1.概观

在本教程中，我们将了解什么是数据传输对象(DTO)、值对象(VO)、普通 Java 对象(POJO)和 JavaBeans。我们将看看它们之间的区别，并了解使用哪种类型以及何时使用。

## 2.普通旧 Java 对象

**[POJO](/web/20220815033752/https://www.baeldung.com/java-pojo-class) ，也称为 Plain Old Java Object，是一个普通的 Java 对象，不引用任何特定的框架。**这是一个用来指代简单、轻量级 Java 对象的术语。

POJO 不为属性和方法使用任何命名约定。

让我们定义一个基本的`EmployeePOJO`对象，它有三个属性:

```
public class EmployeePOJO {

    private String firstName;
    private String lastName;
    private LocalDate startDate;

    public EmployeePOJO(String firstName, String lastName, LocalDate startDate) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.startDate = startDate;
    }

    public String name() {
        return this.firstName + " " + this.lastName;
    }

    public LocalDate getStart() {
        return this.startDate;
    }
}
```

正如我们所看到的，上面的 Java 对象定义了表示雇员的结构，并且不依赖于任何框架。

## 3.JavaBeans

### 3.1.什么是 JavaBean？

JavaBean 很大程度上类似于 POJO，对于如何实现它有一些严格的规则。

规则规定它应该是可序列化的，有一个空构造函数，并允许使用遵循`getX()` 和`setX()` 约定的方法访问变量。

### 3.2.作为 JavaBean 的 POJO

因为 JavaBean 本质上是一个 POJO，所以让我们通过实现必要的 Bean 规则将`EmployeePOJO`转换成 JavaBean:

```
public class EmployeeBean implements Serializable {

    private static final long serialVersionUID = -3760445487636086034L;
    private String firstName;
    private String lastName;
    private LocalDate startDate;

    public EmployeeBean() {
    }

    public EmployeeBean(String firstName, String lastName, LocalDate startDate) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.startDate = startDate;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    // additional getters and setters
}
```

这里，为了将 POJO 转换成 JavaBean，我们实现了`Serializable`接口，将属性标记为`private`，并使用 getter/setter 方法来访问属性。

## 4.数据传输对象

### 4.1.DTO 模式

**DTO，也称为[数据传输对象](/web/20220815033752/https://www.baeldung.com/java-dto-pattern)，封装值以在进程或网络之间传送数据。**

这有助于减少调用方法的数量。通过在单个调用中包含多个参数或值，我们减少了远程操作中的网络开销。

这种模式的另一个优点是序列化逻辑的封装。它让程序以特定的格式存储和传输数据。

DTO 没有任何明显的行为。通过将领域模型与表示层分离，它基本上有助于使代码松散耦合。

### 4.2.如何使用 DTO？

dto 具有扁平的结构，没有任何业务逻辑。它们使用与 POJOs 相同的格式。DTO 只包含与序列化或解析相关的存储、访问器和方法。

dto 基本上映射到一个域模型，从而将数据发送到一个方法或服务器。

让我们创建`EmployeeDTO`，它将创建一个雇员所需的所有细节组合在一起。我们将在一个请求中将这些数据发送到服务器，以优化与 API 的交互:

```
public class EmployeeDTO {

    private String firstName;
    private String lastName;
    private LocalDate startDate;

    // standard getters and setters
}
```

上面的 DTO 与不同的服务交互，并处理数据流。这种 DTO 模式可以在任何服务中使用，没有任何框架限制。

## 5.维多利亚皇家勋章

**VO 也称为值对象，是一种特殊类型的对象，可以保存`java.lang.Integer`、`java.lang.Long`等值。**

VO 应该总是覆盖`equals()`和`hashCode()`方法。VO 通常封装小对象，如数字、日期、字符串等。它们遵循值语义，也就是说，它们直接改变对象的值并传递副本而不是引用。

让值对象不可变是一个很好的实践。只有通过创建新对象，而不是通过更新旧对象本身的值，才会发生值的更改。这有助于理解隐式契约，即两个创建为相等的值对象应该保持相等。

让我们定义`EmployeeVO`并覆盖`equals()`和`hashCode()`方法:

```
public class EmployeeVO {

    private String firstName;
    private String lastName;
    private LocalDate startDate;

    public EmployeeVO(String firstName, String lastName, LocalDate startDate) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.startDate = startDate;
    }
    // Getters

    @Override
    public boolean equals(Object obj) {

        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;

        EmployeeVO emp = (EmployeeVO) obj;

        return Objects.equals(firstName, emp.firstName)
          && Objects.equals(lastName, emp.lastName)
          && Objects.equals(startDate, emp.startDate);
    }

    @Override
    public int hashCode() {
        return Objects.hash(firstName, lastName, startDate);
    }
}
```

## 6.结论

在本文中，我们看到了 POJO、JavaBeans、DTO 和值对象的定义。我们还看到了一些框架和库如何利用 JavaBean 命名约定，以及如何将 POJO 转换成 JavaBean。我们还了解了 DTO 模式和值对象，以及它们在不同场景中的用法。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220815033752/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-4)