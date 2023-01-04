# Java 构造函数指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-constructors>

## 1.介绍

构造者是 [`object-oriented design`](/web/20220722202440/https://www.baeldung.com/java-polymorphism) 的守门人。

在本教程中，我们将看到它们如何作为一个单独的位置来初始化被创建对象的内部状态。

让我们继续前进，创建一个表示银行账户的简单对象。

## 2.开设银行账户

假设我们需要创建一个表示银行账户的类。它会包含一个名字，创建日期和余额。

同样，让我们覆盖`toString`方法，将详细信息打印到控制台:

```java
class BankAccount {
    String name;
    LocalDateTime opened;
    double balance;

    @Override
    public String toString() {
        return String.format("%s, %s, %f", 
          this.name, this.opened.toString(), this.balance);
    }
} 
```

现在，这个类包含了存储银行账户信息所需的所有必要字段，但是它还不包含构造函数。

**这意味着如果我们创建一个新对象，字段值不会被初始化:**

```java
BankAccount account = new BankAccount();
account.toString(); 
```

运行上面的`toString `方法会导致一个异常，因为对象`name`和`opened`仍然是`null`:

```java
java.lang.NullPointerException
    at com.baeldung.constructors.BankAccount.toString(BankAccount.java:12)
    at com.baeldung.constructors.ConstructorUnitTest
      .givenNoExplicitContructor_whenUsed_thenFails(ConstructorUnitTest.java:23) 
```

## 3.无参数构造函数

让我们用一个构造函数来解决这个问题:

```java
class BankAccount {
    public BankAccount() {
        this.name = "";
        this.opened = LocalDateTime.now();
        this.balance = 0.0d;
    }
} 
```

注意我们刚刚写的构造函数的一些事情。首先，它是一个方法，但是没有返回类型。这是因为构造函数隐式地返回它创建的对象的类型。现在调用` new BankAccount()`会调用上面的构造函数。

其次，这不需要争论。这种特殊的构造函数叫做 n `o-argument constructor`。

为什么我们第一次不需要它呢？这是因为当我们**没有显式编写任何构造函数时，编译器会添加一个默认的、无参数的构造函数**。

这就是为什么我们能够第一次构造对象，即使我们没有显式地编写构造函数。默认的无参数构造函数将简单地将所有成员设置为它们的默认值。

对于对象来说，这就是导致我们之前看到的异常的`null,`。

## 4.参数化构造函数

现在，构造函数的一个真正好处是，当向对象注入状态时，它们帮助我们维护`encapsulation`。

所以，要对这个银行账户做一些真正有用的事情，我们需要能够真正地将一些初始值注入到对象中。

为此，**让我们写一个`parameterized constructor`，也就是一个接受一些参数的构造函数**:

```java
class BankAccount {
    public BankAccount() { ... }
    public BankAccount(String name, LocalDateTime opened, double balance) {
        this.name = name;
        this.opened = opened;
        this.balance = balance;
    }
} 
```

现在我们可以用我们的`BankAccount`类做一些有用的事情:

```java
 LocalDateTime opened = LocalDateTime.of(2018, Month.JUNE, 29, 06, 30, 00);
    BankAccount account = new BankAccount("Tom", opened, 1000.0f); 
    account.toString(); 
```

注意，我们的类现在有两个构造函数。一个显式无参数构造函数和一个参数化构造函数。

我们可以创建尽可能多的构造函数，但是我们可能不想创建太多。这可能会有点混乱。

如果我们在代码中发现太多的构造函数，一些[创造性的设计模式](/web/20220722202440/https://www.baeldung.com/creational-design-patterns)可能会有帮助。

## 5.复制构造函数

构造函数不需要仅限于初始化。它们也可以用于以其他方式创建对象。想象一下，我们需要能够从现有账户创建一个新账户。

新账户应该和旧账户同名，有今天的创建日期，没有资金。**我们可以使用`copy constructor` :** 来实现

```java
public BankAccount(BankAccount other) {
    this.name = other.name;
    this.opened = LocalDateTime.now();
    this.balance = 0.0f;
} 
```

现在我们有以下行为:

```java
LocalDateTime opened = LocalDateTime.of(2018, Month.JUNE, 29, 06, 30, 00);
BankAccount account = new BankAccount("Tim", opened, 1000.0f);
BankAccount newAccount = new BankAccount(account);

assertThat(account.getName()).isEqualTo(newAccount.getName());
assertThat(account.getOpened()).isNotEqualTo(newAccount.getOpened());
assertThat(newAccount.getBalance()).isEqualTo(0.0f); 
```

## 6.链式构造函数

当然，我们也许能够推断出一些构造函数参数或者**给它们一些默认值。**

例如，我们可以创建一个只有名字的新银行账户。

因此，让我们创建一个带有`name`参数的构造函数，并为其他参数赋予默认值:

```java
public BankAccount(String name, LocalDateTime opened, double balance) {
    this.name = name;
    this.opened = opened;
    this.balance = balance;
}
public BankAccount(String name) {
    this(name, LocalDateTime.now(), 0.0f);
}
```

用关键字`this,` 我们调用另一个构造函数。

我们必须记住**如果我们想要链接一个超类构造函数，我们必须使用`super`而不是`this`。**

另外，记住 **`this`或`super`表达式应该总是第一个语句。**

## 7.值类型

Java 中构造函数的一个有趣用途是在`Value Objects`的创建中。**值对象是初始化后内部状态不变的对象。**

**即对象是不可变的**。Java 中的不变性有点[微妙](/web/20220722202440/https://www.baeldung.com/java-immutable-object)，在创建对象时应该小心。

让我们继续创建一个不可变的类:

```java
class Transaction {
    final BankAccount bankAccount;
    final LocalDateTime date;
    final double amount;

    public Transaction(BankAccount account, LocalDateTime date, double amount) {
        this.bankAccount = account;
        this.date = date;
        this.amount = amount;
    }
} 
```

注意，我们现在在定义类的成员时使用了`final`关键字。这意味着每个成员只能在类的构造函数中初始化。它们以后不能在任何其他方法中被重新分配。我们可以读取这些值，但不能改变它们。

**如果我们为`Transaction`类创建多个构造函数，每个构造函数都需要初始化每个最终变量。**不这样做将导致编译错误。

## 8.结论

我们已经浏览了构造函数构建对象的不同方式。如果使用得当，构造构成了 Java 中面向对象设计的基本构件。

和往常一样，代码样本可以在 GitHub 上找到。