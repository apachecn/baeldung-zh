# Java 复制构造函数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-copy-constructor>

## 1.介绍

Java 类中的复制构造函数是一个[构造函数](/web/20221231135948/https://www.baeldung.com/java-constructors)，它由**使用同一个 Java 类的另一个对象**创建一个对象。

当我们想要复制一个有几个字段的复杂对象时，或者当我们想要制作一个现有对象的[深度副本](/web/20221231135948/https://www.baeldung.com/java-deep-copy)时，这是很有帮助的。

## 2.如何创建复制构造函数

要创建复制构造函数，我们可以首先声明一个构造函数，它将一个相同类型的对象作为参数:

```java
public class Employee {
    private int id;
    private String name;

    public Employee(Employee employee) {
    }
}
```

然后，我们将输入对象的每个字段复制到新实例中:

```java
public class Employee {
    private int id;
    private String name;

    public Employee(Employee employee) {
        this.id = employee.id;
        this.name = employee.name;
    }
}
```

我们这里有一个`shallow copy`，这很好，因为我们所有的字段——在这个例子中是一个`int` 和一个`String` ——要么是[原始类型](/web/20221231135948/https://www.baeldung.com/java-primitives)要么是[不可变类型](/web/20221231135948/https://www.baeldung.com/java-immutable-object)。

如果 Java 类有可变字段，那么我们可以在它的复制构造函数中创建一个 [`deep copy`](/web/20221231135948/https://www.baeldung.com/java-deep-copy) 。使用深度副本，新创建的对象独立于原始对象，因为我们为每个可变对象创建了一个不同的副本:

```java
public class Employee {
    private int id;
    private String name;
    private Date startDate;

    public Employee(Employee employee) {
        this.id = employee.id;
        this.name = employee.name;
        this.startDate = new Date(employee.startDate.getTime());
    }
}
```

## 3.复制构造函数与克隆

在 Java 中，我们还可以使用 [`clone`](/web/20221231135948/https://www.baeldung.com/java-deep-copy) 方法从现有对象创建一个对象。然而，复制构造函数比`clone`方法有一些优势:

1.  复制构造函数更容易实现。我们不需要实现`Cloneable`接口和处理`CloneNotSupportedException`。
2.  `clone`方法返回一个通用的`Object`引用。因此，我们需要将其类型转换为适当的类型。
3.  我们不能在`clone` 方法中给`final`字段赋值。但是，我们可以在复制构造函数中这样做。

## 4.继承问题

Java 中的复制构造函数不能被子类继承。因此，如果我们试图从一个父类引用初始化一个子对象，**当我们用复制构造函数克隆它时，我们将面临一个造型问题**。

为了说明这个问题，让我们首先创建一个`Employee`的子类和它的复制构造函数:

```java
public class Manager extends Employee {
    private List<Employee> directReports;
    // ... other constructors

    public Manager(Manager manager) {
        super(manager.id, manager.name, manager.startDate);
        this.directReports = directReports.stream()
          .collect(Collectors.toList());
    }
} 
```

然后，我们声明一个`Employee`变量并用`Manager`构造函数实例化它:

```java
Employee source = new Manager(1, "Baeldung Manager", startDate, directReports);
```

由于引用类型是`Employee`，我们必须将其转换为`Manager`类型，这样我们就可以使用`Manager`类的复制构造函数:

```java
Employee clone = new Manager((Manager) source);
```

如果输入对象不是`Manager`类的实例，我们可能会在运行时得到`ClassCastException` 。

避免在复制构造函数中进行强制转换的一种方法是为这两个类创建一个新的可继承方法:

```java
public class Employee {
   public Employee copy() {
        return new Employee(this);
    }
}

public class Manager extends Employee {
    @Override
    public Employee copy() {
        return new Manager(this);
    }
}
```

在每个类方法中，我们用`this`对象的输入调用它的复制构造函数。这样，我们可以保证生成的对象等于调用者对象:

```java
Employee clone = source.copy();
```

## 5.结论

在本教程中，我们用一些代码示例展示了如何创建复制构造函数。此外，我们讨论了我们应该避免使用`clone`方法的几个原因。

当我们使用复制构造函数来克隆引用类型是父类的子类对象时，它有一个强制转换问题。我们为这个问题提供了一个解决方案。

和往常一样，该教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221231135948/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-constructors)