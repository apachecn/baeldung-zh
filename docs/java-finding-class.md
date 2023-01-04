# 在 Java 中查找对象的类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-finding-class>

## 1.概观

在本文中，我们将探索在 Java 中查找对象的类的不同方法。

## 2.使用`getClass()`方法

我们要检查的第一个方法是`getClass()`方法。

首先，让我们看看我们的代码。我们将编写一个`User`类:

```
public class User {

    // implementation details

}
```

现在，让我们创建一个扩展了`User`的`Lender`类:

```
public class Lender extends User {

    // implementation details

}
```

同样，我们将创建一个扩展了`User`的`Borrower`类:

```
public class Borrower extends User {

    // implementation details

}
```

`getClass()`方法仅仅是**返回我们正在评估的对象**的运行时类，因此，我们不考虑继承。

正如我们所见，`getClass()`表明我们的`lender`对象的类属于`Lender`类型，但不属于`User`类型:

```
@Test
public void givenLender_whenGetClass_thenEqualsLenderType() {
    User lender = new Lender();
    assertEquals(Lender.class, lender.getClass());
    assertNotEquals(User.class, lender.getClass());
}
```

## 3.使用`isInstance()`方法

当使用`isInstance()`方法时，**我们检查一个对象是否是一个特定的类型**，通过类型，我们或者谈论一个类或者一个接口。

如果我们作为方法参数发送的对象通过了类或接口类型的 IS-A 测试，这个方法将返回`true` **。**

我们可以使用`isInstance()`方法在运行时**检查对象的类。**此外，`isInstance()` **也处理[自动装箱](/web/20221208143830/https://www.baeldung.com/java-wrapper-classes#autoboxing-and-unboxing)。**

如果我们检查下面的代码，我们会发现代码无法编译:

```
@Ignore
@Test
public void givenBorrower_whenDoubleOrNotString_thenRequestLoan() {
    Borrower borrower = new Borrower();
    double amount = 100.0;

    if(amount instanceof Double) { // Compilation error, no autoboxing
        borrower.requestLoan(amount);
    }

    if(!(amount instanceof String)) { // Compilation error, incompatible operands
        borrower.requestLoan(amount);
    }

}
```

让我们使用`isInstance()`方法来检查自动装箱的运行情况:

```
@Test
public void givenBorrower_whenLoanAmountIsDouble_thenRequestLoan() {
    Borrower borrower = new Borrower();
    double amount = 100.0;

    if(Double.class.isInstance(amount)) { // No compilation error
        borrower.requestLoan(amount);
    }
    assertEquals(100, borrower.getTotalLoanAmount());
}
```

现在，让我们尝试在运行时评估我们的对象:

```
@Test
public void givenBorrower_whenLoanAmountIsNotString_thenRequestLoan() {
    Borrower borrower = new Borrower();
    Double amount = 100.0;

    if(!String.class.isInstance(amount)) { // No compilation error
        borrower.requestLoan(amount);
    }
    assertEquals(100, borrower.getTotalLoanAmount());
}
```

我们也可以使用`isInstance()`到**来验证一个对象在被转换成**之前是否可以被转换成另一个类:

```
@Test
public void givenUser_whenIsInstanceOfLender_thenDowncast() {
    User user = new Lender();
    Lender lender = null;

    if(Lender.class.isInstance(user)) {
        lender = (Lender) user;
    }

    assertNotNull(lender);
}
```

当我们使用`isInstance()`方法时，我们保护我们的程序不尝试非法的向下转换，尽管在这种情况下使用 **`instanceof`操作符**会更流畅。接下来我们来检查一下。

## 4.使用`instanceof`操作符

类似于`isInstance()`方法，如果被评估的对象属于给定的类型，那么`instanceof`操作符返回`true`——换句话说，如果我们在操作符左边引用的对象通过了右边的类或接口类型的 IS-A 测试，那么**。**

我们可以评估一个`Lender`对象是类型`Lender`还是类型 `User`:

```
@Test
public void givenLender_whenInstanceOf_thenReturnTrue() {
    User lender = new Lender();
    assertTrue(lender instanceof Lender);
    assertTrue(lender instanceof User);
}
```

为了深入了解`instanceof`操作符是如何工作的，我们可以在我们的 [Java instanceOf Operator 文章](/web/20221208143830/https://www.baeldung.com/java-instanceof)中找到更多信息。

## 5.结论

在本文中，我们回顾了在 Java 中查找对象类的三种不同方法:`getClass()`方法、`isInstance()`方法和`instanceof`操作符。

像往常一样，完整的代码样本可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators)