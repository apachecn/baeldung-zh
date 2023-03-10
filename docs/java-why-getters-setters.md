# Java 中 Getters 和 Setters 的意义

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-why-getters-setters>

## 1.介绍

Getters 和 Setters 在检索和更新封装类之外的变量值时起着重要的作用。setter 更新变量的值，而 getter 读取变量的值。

在本教程中，我们将讨论不使用[getter/setter](/web/20221128100505/https://www.baeldung.com/intro-to-project-lombok)的问题，它们的重要性，以及在 Java 中实现它们时要避免的常见错误。

## 2.Java 中没有 Getters 和 Setters 的生活

考虑这样一种情况，当我们想要基于某种条件改变一个对象的状态。如果没有 setter 方法，我们怎么能做到呢？

*   将变量标记为公共、受保护或默认
*   使用点(.)运算符

我们来看看这样做的后果。

## 3.不用 Getters 和 Setters 访问变量

首先，为了在没有 getter/setter 的情况下访问类外部的变量，我们必须将它们标记为 public、protected 或 default。**因此，我们正在失去对数据的控制，并损害了基本的 [OOP](/web/20221128100505/https://www.baeldung.com/cs/oop-modeling-real-world) 原则—[封装](/web/20221128100505/https://www.baeldung.com/java-oop)。**

第二，因为任何人都可以直接从类外部改变非私有字段，所以我们不能实现不变性。

第三，我们不能为变量的变化提供任何条件逻辑。让我们假设我们有一个带有字段`retirementAge`的类`Employee`:

```java
public class Employee {
    public String name;
    public int retirementAge;

// Constructor, but no getter/setter
}
```

注意，这里我们已经将字段设置为 public，以允许从类`Employee`外部访问。现在，我们需要更改员工的`retirementAge`:

```java
public class RetirementAgeModifier {

    private Employee employee = new Employee("John", 58);

    private void modifyRetirementAge(){
        employee.retirementAge=18;
    }
}
```

在这里，`Employee`类的任何客户端都可以很容易地用`retirementAge`字段做他们想做的事情。**没有办法验证这一变化。**

第四，我们如何从类外部实现对字段的只读或只写访问？

有 getters 和 setters 来救你。

## 4.Java 中 Getters 和 Setters 的意义

在众多好处中，让我们来看看使用 getters 和 setters 的一些最重要的好处:

*   **它帮助我们实现封装，封装用于隐藏类中结构化数据对象的状态，防止对它们的未授权直接访问**
*   通过将字段声明为私有并仅使用 getters 来实现不变性
*   Getters 和 setters 还支持其他功能，比如验证、错误处理，这些功能在将来会更容易添加。因此，我们可以添加条件逻辑，并根据需要提供行为
*   我们可以为字段提供不同的访问级别；例如，get(读访问)可以是公共的，而 set(写访问)可以是受保护的
*   对正确设置属性值的控制
*   使用 getters 和 setters，我们实现了 OOP 的另一个关键原则，即抽象，它隐藏了实现细节，这样就没有人能在其他类或模块中直接使用这些字段

## 5.避免错误

下面是实现 getters 和 setters 时要避免的最常见的错误。

### 5.1.对公共变量使用 Getters 和 Setters

公共变量可以在类外使用点(.)运算符。对公共变量使用 getters 和 setters 是没有意义的:

```java
public class Employee {
    public String name;
    public int retirementAge;

    public void setName(String name) {
        this.name = name;
    }
    public String getName() {
        return this.name;
    } 
    // getter/setter for retirementAge
}
```

在这种情况下，可以用 getters 和 setters 完成的任何事情也可以通过简单地将字段设置为 public 来完成。

根据经验，我们需要**总是根据实现封装的需要使用最受限制的访问修饰符。**

### 5.2.在 Setter 方法中直接分配对象引用

当我们在 setter 方法中直接分配对象引用时，这两个引用都指向内存中的一个对象。因此，使用任何引用变量进行的更改实际上都是在同一个对象上进行的:

```java
public void setEmployee(Employee employee) {
    this.employee = employee;
}
```

然而，我们可以使用[深度复制](/web/20221128100505/https://www.baeldung.com/java-deep-copy)将所有元素从一个对象复制到另一个对象。因此，`this`对象的状态变得独立于现有的(传递的)employee 对象:

```java
public void setEmployee(Employee employee) {
    this.employee.setName(employee.getName());
    this.employee.setRetirementAge(employee.getRetirementAge());
}
```

### 5.3.直接从 Getter 方法返回对象引用

类似地，如果 getter 方法直接返回对象的引用，任何人都可以从外部代码使用该引用来更改对象的状态:

```java
public Employee getEmployee() {
    return this.employee;
}
```

让我们使用这个 `getEmployee()`方法并改变`retirementAge:`

```java
private void modifyAge() {
    Employee employeeTwo = getEmployee();
    employeeTwo.setRetirementAge(65);
}
```

这导致原始对象不可恢复的丢失。

因此，我们应该返回对象的副本，而不是从 getter 方法返回引用。其中一种方法如下:

```java
public Employee getEmployee() {
    return new Employee(this.employee.getName(), this.employee.getRetirementAge());
}
```

然而，我们还应该记住，在 getter 或 setter 中创建对象的副本可能并不总是最佳实践。例如，在循环中调用上述 getter 方法可能会导致开销很大的操作。

另一方面，如果我们希望我们的集合保持不可修改，从 getter 返回集合的副本是有意义的。然后，我们必须确定哪种方法最适合特定的情况。

### 5.4.添加不必要的 Getters 和 Setters

通过 getters 和 setters，我们可以控制成员变量的访问和赋值。但是，在许多地方，事实证明这是不必要的。此外，它使代码变得冗长:

```java
private String name;

public String getName() {
    return name;
}

public void setName(String name) {
    this.name = name;
}
```

简单地为一个类中的私有字段定义公共的 getter 和 setter，就相当于将该字段变为公共的而没有 getter 和 setter。因此，明智的做法是仔细考虑是否为所有字段定义访问器方法。

## 6.结论

在本教程中，我们讨论了在 Java 中使用 getters 和 setters 的优缺点。我们还讨论了在实现 getters 和 setters 时要避免的一些常见错误，以及如何正确使用它们