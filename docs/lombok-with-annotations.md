# 使用带注释的@的 Lombok

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-with-annotations>

## 1.介绍

Lombok 是一个帮助我们在编写 Java 应用程序时显著减少样板代码的库。

在本教程中，我们将看到如何使用这个库来复制只改变一个属性的不可变对象。

## 2.使用

当处理不可变对象时，设计上不允许 setters，我们可能需要一个与当前对象相似的对象，但是只有一个属性不同。这可以通过使用 Lombok 的`@With`注释来实现:

```java
public class User {
    private final String username;
    private final String emailAddress;
    @With
    private final boolean isAuthenticated;

    //getters, constructors
}
```

上面的注释生成了以下内容:

```java
public class User {
    private final String username;
    private final String emailAddress;
    private final boolean isAuthenticated;

    //getters, constructors

    public User withAuthenticated(boolean isAuthenticated) {
        return this.isAuthenticated == isAuthenticated ? this : new User(this.username, this.emailAddress, isAuthenticated);
    }
}
```

然后，我们可以使用上面生成的方法来创建原始对象的变异副本:

```java
User immutableUser = new User("testuser", "[[email protected]](/web/20221126224530/https://www.baeldung.com/cdn-cgi/l/email-protection)", false);
User authenticatedUser = immutableUser.withAuthenticated(true);

assertNotSame(immutableUser, authenticatedUser);
assertFalse(immutableUser.isAuthenticated());
assertTrue(authenticatedUser.isAuthenticated());
```

此外，我们可以选择**注释整个类，这将为所有属性**生成`withX() `方法。

## 3.要求

为了正确使用`@With`注释，**我们需要提供一个全参数构造函数**。正如我们从上面的例子中看到的，生成的方法需要它来创建原始对象的克隆。

我们可以使用 Lombok 自己的`@AllArgsConstructor`或`@Value`注释来满足这个需求。或者，我们也可以手动提供这个构造函数，同时确保类中非静态属性的顺序与构造函数的顺序相匹配。

我们应该记住，如果用在静态字段上， **`@With`注释什么也不做。这是因为静态属性不被认为是对象状态的一部分。此外，对于以`$`符号**开头的字段，Lombok 跳过了**方法的生成。**

## 4.高级用法

让我们研究一下使用这个注释时的一些高级场景。

### 4.1.抽象类

我们可以在抽象类的字段上使用`@With`注释:

```java
public abstract class Device {
    private final String serial;
    @With
    private final boolean isInspected;

    //getters, constructor
}
```

然而，**我们将需要为生成的`withInspected() `方法**提供一个实现。这是因为 Lombok 不知道我们的抽象类的具体实现来创建它的克隆:

```java
public class KioskDevice extends Device {

    @Override
    public Device withInspected(boolean isInspected) {
        return new KioskDevice(getSerial(), isInspected);
    }

    //getters, constructor
}
```

### 4.2.命名规格

如上所述，Lombok 将跳过以`$`符号开头的字段。但是，如果字段以一个字符开头，那么它就是标题大小写的，最后，`with`作为前缀加到生成的方法上。

或者，如果字段以下划线开头，那么`with`只是作为生成方法的前缀:

```java
public class Holder {
    @With
    private String variableA;
    @With
    private String _variableB;
    @With
    private String $variableC;

    //getters, constructor excluding $variableC
}
```

根据上面的代码，我们看到只有前两个变量有为它们生成的`withX() `方法:

```java
Holder value = new Holder("a", "b");

Holder valueModifiedA = value.withVariableA("mod-a");
Holder valueModifiedB = value.with_variableB("mod-b");
// Holder valueModifiedC = value.with$VariableC("mod-c"); not possible
```

### 4.3.方法生成的例外

我们应该注意，除了以`$`符号开始的字段之外， **Lombok 不会生成一个`withX() `方法，如果它已经存在于我们的类**中:

```java
public class Stock {
    @With
    private String sku;
    @With
    private int stockCount;

    //prevents another withSku() method from being generated
    public Stock withSku(String sku) {
        return new Stock("mod-" + sku, stockCount);
    }

    //constructor
}
```

在上面的场景中，不会生成新的`withSku()`方法。

此外，在下面的场景中，Lombok **跳过** **方法生成:**

```java
public class Stock {
    @With
    private String sku;
    private int stockCount;

    //also prevents another withSku() method from being generated
    public Stock withSKU(String... sku) {
        return sku == null || sku.length == 0 ?
          new Stock("unknown", stockCount) :
          new Stock("mod-" + sku[0], stockCount);
    }

    //constructor
}
```

我们可以注意到上面的`withSKU()` 方法的不同命名。

基本上，如果出现以下情况，Lombok 将跳过方法生成:

*   存在与生成的方法名相同的方法(忽略大小写)
*   现有方法与生成的方法具有相同数量的参数(包括 var-args)

### 4.4.对生成的方法进行无效验证

与其他 Lombok 注释类似，我们可以对使用`@With`注释生成的方法进行`null`检查:

```java
@With
@AllArgsConstructor
public class ImprovedUser {
    @NonNull
    private final String username;
    @NonNull
    private final String emailAddress;
}
```

Lombok 将为我们生成以下代码以及所需的`null`检查:

```java
public ImprovedUser withUsername(@NonNull String username) {
    if (username == null) {
        throw new NullPointerException("username is marked non-null but is null");
    } else {
        return this.username == username ? this : new ImprovedUser(username, this.emailAddress);
    }
}

public ImprovedUser withEmailAddress(@NonNull String emailAddress) {
    if (emailAddress == null) {
        throw new NullPointerException("emailAddress is marked non-null but is null");
    } else {
        return this.emailAddress == emailAddress ? this : new ImprovedUser(this.username, emailAddress);
    }
}
```

## 5.结论

在本文中，我们已经看到了如何使用 Lombok 的`@With`注释来生成特定对象的克隆，其中一个字段发生了变化。

我们还了解了这个方法生成实际上是如何以及何时工作的，以及如何用额外的验证(比如`null`检查)来扩充它。

与往常一样，GitHub 上的[提供了代码示例。](https://web.archive.org/web/20221126224530/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok-2)