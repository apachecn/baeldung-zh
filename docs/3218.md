# Java 中的私有构造函数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-private-constructors>

## 1.介绍

私有构造函数**允许我们限制类**的实例化。简而言之，它们防止在类本身之外的任何地方创建类实例。

公共和私有构造函数一起使用，允许控制我们希望如何实例化我们的类——这被称为构造函数委托。

## 2.典型用法

限制显式类实例化有几种模式和好处，我们将在本教程中介绍最常见的几种:

*   [单例模式](/web/20220703152758/https://www.baeldung.com/java-singleton)
*   委托构造函数
*   不可实例化类
*   [构建器模式](/web/20220703152758/https://www.baeldung.com/java-builder-pattern-freebuilder)

让我们看看**如何定义私有构造函数**:

```
public class PrivateConstructorClass {

    private PrivateConstructorClass() {
        // in the private constructor
    }
}
```

我们定义私有构造函数类似于公共构造函数；我们只是简单地将关键字`public`改为`private`。

## 3.在单例模式中使用私有构造函数

单例模式是我们最常遇到使用私有构造函数的地方之一。私有构造函数**允许我们将类实例化限制到单个对象实例**:

```
public final class SingletonClass {

    private static SingletonClass INSTANCE;
    private String info = "Initial info class";

    private SingletonClass() {
    }

    public static SingletonClass getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new SingletonClass();
        }

        return INSTANCE;
    }

    // getters and setters
}
```

我们可以通过调用`SingletonClass.getInstance()`来创建一个实例——这要么返回一个现有的实例，要么创建一个实例(如果这是第一次实例化的话)。我们只能通过使用`getInstance()`静态方法来实例化这个类。

## 4.使用私有构造函数委托构造函数

私有构造函数的另一个常见用例是提供一种构造函数委托的方法。构造函数委托**允许我们通过几个不同的构造函数传递参数，同时将初始化限制在特定的位置**。

在这个例子中， *ValueTypeClass* 允许使用值和类型进行初始化——但是我们只希望允许它用于类型的子集。通用构造函数必须是私有的，以确保只使用允许的类型:

```
public class ValueTypeClass {

    private final String value;
    private final String type;

    public ValueTypeClass(int x) {
        this(Integer.toString(x), "int");
    }

    public ValueTypeClass(boolean x) {
        this(Boolean.toString(x), "boolean");
    }

    private ValueTypeClass(String value, String type) {
        this.value = value;
        this.type = type;
    }

    // getters and setters
}
```

我们可以通过两个不同的公共构造函数初始化`ValueType` `Class`:一个接受`int`，另一个接受`boolean`。然后，这些构造函数中的每一个都调用一个公共的私有构造函数来完成对象初始化。

## 5.使用私有构造函数创建不可实例化的类

不可实例化的类是我们不能实例化的类。在这个例子中，我们将创建**一个简单包含静态方法**集合的类:

```
public class StringUtils {

    private StringUtils() {
        // this class cannot be instantiated
    }

    public static String toUpperCase(String s) {
        return s.toUpperCase();
    }

    public static String toLowerCase(String s) {
        return s.toLowerCase();
    }
}
```

`StringUtils`类包含几个静态实用方法，由于私有构造函数而不能被实例化。

实际上，没有必要允许对象实例化，因为静态方法不需要使用对象实例。

## 6.在生成器模式中使用私有构造函数

构建器模式允许我们一步一步地构造复杂的对象，而不是让几个构造器提供不同的方法来创建对象。私有构造函数限制初始化，允许构建器管理对象的创建。

在这个例子中，我们创建了一个保存雇员的`name`、`age`和`department`的`Employee`类:

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

正如我们所见，我们已经将`Employee`构造函数设为私有——因此，我们不能显式实例化该类。

我们现在将内部的`Builder`类添加到`Employee`类中:

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

构建器现在可以用`name`、`age`或`department`创建不同的雇员——对于我们必须提供多少字段没有限制:

```
Employee.Builder emplBuilder = new Employee.Builder();

Employee employee = emplBuilder
  .setName("baeldung")
  .setDepartment("Builder Pattern")
  .build();
```

我们创建了一个名为“T1”的“T0”和一个名为“T2”的部门。没有提供年龄，所以将使用默认的原语`int`值 0。

## 7.使用私有构造函数来防止子类化

私有构造函数的另一个可能用途是防止类的子类化。如果我们试图创建这样的子类，那就无法调用 `super` 构造函数 。然而，需要注意的是**我们通常会创建一个类 [`final`](/web/20220703152758/https://www.baeldung.com/java-final) 来防止子类化，而不是使用私有构造函数**。

## 8.结论

私有构造函数的主要用途是限制类的实例化。 **当我们想要限制一个类** 的外部创建时，私有构造函数特别有用。

单例、工厂和静态方法对象是限制对象实例化如何有助于实施某种模式的例子。

常量类和静态方法类也规定了一个类不应该是可实例化的。 重要的是要记住，我们还可以将私有构造函数与公共构造函数结合起来，以允许不同公共 构造函数 定义 内部的代码共享。

这些例子的代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220703152758/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-constructors)