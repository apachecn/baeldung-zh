# Java 隐式超级构造函数未定义错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-implicit-super-constructor-is-undefined-error>

## 1.概观

在本教程中，我们将仔细看看 Java **“隐式超级构造函数未定义”**错误。首先，我们将创建一个如何生成它的示例。接下来，我们将解释异常的主要原因，稍后我们将看到如何修复它。

## 2.实际例子

现在来看一个产生编译错误“隐式超构造函数 X()未定义”的例子。必须显式调用另一个构造函数”。

这里，`X`表示父类，它被任何看到这个错误的子类扩展。

首先，让我们创建一个父类`Person`:

```java
public class Person {

    String name;
    Integer age;

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

   // setters and getters
}
```

现在，让我们创建一个子类`Employee` ，它的父类是 `Person`:

```java
public class Employee extends Person {

    Double salary;

    public Employee(String name, Integer age, Double salary) {
        this.salary = salary;
    }

    // setters and getters
}
```

现在，在我们的 IDE 中，我们将看到错误:

```java
Implicit super constructor Person() is undefined. Must explicitly invoke another constructor
```

在某些情况下，如果子类没有构造函数，我们会得到类似的错误。

例如，让我们考虑没有构造函数的`Employee`:

```java
public class Employee extends Person {

    Double salary;

    // setters and getters
}
```

我们将在 IDE 中看到错误:

```java
Implicit super constructor Person() is undefined for default constructor. Must define an explicit constructor
```

## 3.原因

在 Java 继承中，[构造函数链接](/web/20220927001856/https://www.baeldung.com/java-chain-constructors)指的是使用`super`方法调用一系列构造函数来链接父类的构造函数。子类构造函数必须显式或隐式地调用超类构造函数。无论哪种方式，都必须定义一个超级构造函数。

没有父类的类将`Object`类作为其父类。Java 中的`Object`类有一个不带参数的构造函数。

当一个类没有构造函数时，**编译器会添加一个默认的不带参数的构造函数，并且在第一条语句中，编译器会插入一个对`super`** 的调用——该调用会调用`Object`类的构造函数。

让我们假设我们的`Person`类不包含任何构造函数，也没有父类。一旦我们编译，我们可以看到编译器已经添加了默认的构造函数:

```java
public Person() {
    super();
}
```

相反，**如果在`Person`类中已经有一个构造函数，编译器不会添加这个默认的无参数构造函数。**

现在，如果我们创建扩展了`Person,`的子类`Employee`，我们会在`Employee`类中得到一个错误:

```java
Implicit super constructor Person() is undefined for default constructor. Must define an explicit constructor
```

由于编译器会插入一个对`Employee`构造函数的`super`调用，它不会在父类`Person.`中找到一个没有参数的构造函数

## 4.解决办法

为了解决这个错误，我们需要明确地向编译器提供信息。

我们需要做的第一件事是从`Employee`构造函数中显式调用`super`构造函数:

```java
public Employee(String name, Integer age, Double salary) {
    super(name, age);
    this.salary = salary;
}
```

现在，假设我们需要创建一个只有`salary`字段的`Employee`对象。让我们编写构造函数:

```java
public Employee(Double salary) {
    super();
    this.salary = salary;
}
```

尽管向`Employee`构造函数**添加了`super`调用，我们仍然会收到一个错误，因为`Person`类仍然缺少一个匹配的构造函数**。我们可以通过在`Person`类中显式创建一个无参数构造函数来解决这个问题:

```java
public Person(String name, Integer age) {
    this.name = name;
    this.age = age;
}

public Person() {
}
```

最后，感谢这些改变，我们不会得到编译错误。

## 5.结论

我们已经解释了 Java 的“隐式超级构造函数未定义”错误。然后，我们讨论了如何产生错误和异常的原因。最后，我们讨论了解决错误的方法。

和往常一样，本文的示例代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220927001856/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-constructors/)