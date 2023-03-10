# 用 Java 链接构造函数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-chain-constructors>

## 1.概观

在这个简短的教程中，我们将看到如何在 Java 中**链接构造函数。这是一种方便的设计模式，可以减少重复代码，提高可读性。**

首先，我们将解释什么是[构造函数](/web/20220525131659/https://www.baeldung.com/java-constructors)链接。然后，我们将看到如何在同一个类中链接它们，并使用父类中的构造函数。最后但同样重要的是，我们将分析这种方法的优点和缺点。

## 2.用示例链接构造函数定义

**构造函数链接是调用一系列构造函数**的过程。我们可以通过两种方式做到这一点:

*   通过使用`this()`关键字链接同一个类中的构造函数
*   通过使用`super()`关键字链接父类的构造函数

让我们看看展示这两种方法的例子。

### 2.1.在同一个类中链接构造函数

让我们定义一个简单的包含几个属性的`Person`类:

```java
public class Person {
    private final String firstName;
    private final String middleName;
    private final String lastName;
    private final int age;

    //getters, equals and hashcode
}
```

`firstName`、`lastName, `和`age `是我们希望在对象初始化期间始终设置的属性。然而，并不是每个人都有中间名。因此`middleName`属性是可选的。

考虑到这一点，我们将创建两个构造函数。第一个接受所有四个属性:

```java
public Person(String firstName, String middleName, String lastName, int age) {
    this.firstName = firstName;
    this.middleName = middleName;
    this.lastName = lastName;
    this.age = age;
}
```

第二个构造函数将接受三个必需的属性并省略可选字段:

```java
public Person(String firstName, String lastName, int age) {
    this(firstName, null, lastName, age);
}
```

**我们使用的是`this()`** **关键字。它必须总是构造函数**的第一行。这确保了我们链接到的构造函数将首先被调用。

请记住，构造函数在类中的顺序是不相关的。这意味着我们的第二个构造函数可以放在 `Person ` 类中的任何地方，它仍然可以正确工作。

### 2.2.从父类链接构造函数

让我们定义一个从上一节中创建的`Person`类继承而来的`Customer`类:

```java
public class Customer extends Person {
    private final String loyaltyCardId;

   //getters, equals and hashcode
}
```

它包含一个额外的属性。现在，让我们以类似于在`Person`类中的方式创建两个构造函数:

```java
public Customer(String firstName, String lastName, int age, String loyaltyCardId) {
    this(firstName, null, lastName, age, loyaltyCardId);
}

public Customer(String firstName, String middleName, String lastName, int age, String loyaltyCardId) {
    super(firstName, middleName, lastName, age);
    this.loyaltyCardId = loyaltyCardId;
}
```

第一个构造函数使用`this()`关键字链接到第二个构造函数，后者接受所有必需的和可选的属性。这里，我们第一次使用了`super()`关键字。

它的行为非常类似于`this()`关键字。唯一的区别是 **`super()`链接到父类中对应的构造函数，而`this()`链接到同一个类中的构造函数**。

请记住，与前面的关键字类似，`super()`必须始终是构造函数**的第一行。**这意味着首先调用父类的构造函数 i 。之后，该值被赋给`loyaltyCardId`属性。

## 3.链接构造函数的优点和缺点

**构造函数链接最大的优点是** **重复代码少**。换句话说，我们使用了不重复自己([干](/web/20220525131659/https://www.baeldung.com/java-clean-code#2-dry-amp-kiss))的原则。这是因为我们在单个构造函数中完成了对象的初始化，通常是接受所有属性的构造函数。其他构造函数只是用来传递接收到的数据和为缺失的属性添加默认值。

链接构造函数使代码更具可读性。我们不必在所有的构造函数中重复属性赋值。相反，我们在一个地方做这个。

另一方面，**当使用构造函数链**时，我们公开了更多构造对象的方法。在某些项目中，这可能是一个很大的缺点。在那些情况下，我们应该寻找[工厂](/web/20220525131659/https://www.baeldung.com/creational-design-patterns#factory-method)或[建造者](/web/20220525131659/https://www.baeldung.com/creational-design-patterns#builder)模式来隐藏多个建造者。

## 4.结论

在本文中，我们讨论了 Java 中的构造函数链接。首先，我们解释了什么叫做构造函数链接。然后，我们展示了如何使用同一个类中的构造函数以及使用父类的构造函数来实现这一点。最后，我们讨论了链接构造函数的优点和缺点。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220525131659/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-4)