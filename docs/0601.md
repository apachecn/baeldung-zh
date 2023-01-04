# Java 中的接口分离原则

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-interface-segregation>

## 1.介绍

在本教程中，我们将讨论界面分离原理，这是[固体原理](/web/20220526053728/https://www.baeldung.com/solid-principles)之一。代表“固体”中的“我”，界面分离仅仅意味着我们应该把较大的界面分成较小的界面。

从而确保实现类不需要实现不需要的方法。

## 2.界面分离原理

这个原则首先由 Robert C. Martin 定义为:“**客户不应该被迫依赖他们不使用的接口**”。

这个原则的目标是**通过将应用程序接口分解成较小的接口**来减少使用较大接口的副作用。这类似于[的单一责任原则](/web/20220526053728/https://www.baeldung.com/java-single-responsibility-principle)，其中每个类或接口服务于一个单一的目的。

精确的应用程序设计和正确的抽象是接口分离原则背后的关键。虽然在应用程序的设计阶段会花费更多的时间和精力，并可能增加代码的复杂性，但最终，我们会得到一个灵活的代码。

我们将在后面的章节中研究一些违反原则的例子，然后通过正确应用原则来解决问题。

## 3.示例接口和实现

让我们看看这样一种情况，我们有一个由实现`BankPayment`使用的`Payment`接口:

```
public interface Payment { 
    void initiatePayments();
    Object status();
    List<Object> getPayments();
}
```

以及实现:

```
public class BankPayment implements Payment {

    @Override
    public void initiatePayments() {
       // ...
    }

    @Override
    public Object status() {
        // ...
    }

    @Override
    public List<Object> getPayments() {
        // ...
    }
}
```

为了简单起见，让我们忽略这些方法的实际业务实现。

这非常清楚——到目前为止，实现类`BankPayment`需要`Payment`接口中的所有方法。因此，它不违反原则。

## 4.污染界面

现在，随着时间的推移，越来越多的功能出现，有必要添加一个`LoanPayment`服务。这个服务也是一种`Payment `但是多了一些操作。

为了开发这个新特性，我们将向`Payment`接口添加新方法:

```
public interface Payment {

    // original methods
    ...
    void intiateLoanSettlement();
    void initiateRePayment();
}
```

接下来，我们将实现`LoanPayment`:

```
public class LoanPayment implements Payment {

    @Override
    public void initiatePayments() {
        throw new UnsupportedOperationException("This is not a bank payment");
    }

    @Override
    public Object status() {
        // ...
    }

    @Override
    public List<Object> getPayments() {
        // ...
    }

    @Override
    public void intiateLoanSettlement() {
        // ...
    }

    @Override
    public void initiateRePayment() {
        // ...
    }
}
```

现在，由于`Payment `接口已经改变并且添加了更多的方法，所有的实现类现在都必须实现新的方法。**问题是，实施它们是多余的，可能会导致许多副作用。**在这里，`LoanPayment`实现类必须实现 `initiatePayments()`,而实际上并不需要这样做。所以，这个原则被违反了。

那么，我们的`BankPayment `类会发生什么呢:

```
public class BankPayment implements Payment {

    @Override
    public void initiatePayments() {
        // ...
    }

    @Override
    public Object status() {
        // ...
    }

    @Override
    public List<Object> getPayments() {
        // ...
    }

    @Override
    public void intiateLoanSettlement() {
        throw new UnsupportedOperationException("This is not a loan payment");
    }

    @Override
    public void initiateRePayment() {
        throw new UnsupportedOperationException("This is not a loan payment");
    }
}
```

注意，`BankPayment `实现现在已经实现了新的方法。因为它不需要它们，也没有它们的逻辑，所以**只是抛出一个`UnsupportedOperationException`。这就是我们开始违反原则的地方。**

在下一节中，我们将看到如何解决这个问题。

## 5.运用原则

在上一节中，我们故意污染了界面，违反了原则。在本节中，我们将探讨如何在不违背原则的情况下添加贷款支付的新功能。

让我们来分解一下每种支付类型的界面。当前形势:

[![](img/97dbc446f76e4c537195abec430ba390.png)](/web/20220526053728/https://www.baeldung.com/wp-content/uploads/2020/07/interface_segregation_poor.png)

注意在类图中，并参考前面部分的接口，在两个实现中都需要`status()`和`getPayments() `方法。另一方面，`initiatePayments()`只在`BankPayment`中需要，`initiateLoanSettlement()`和`initiateRePayment()`方法只针对`LoanPayment`。

分类之后，让我们分解接口并应用接口分离原则。因此，我们现在有了一个通用接口:

```
public interface Payment {
    Object status();
    List<Object> getPayments();
}
```

以及两种支付类型的两个接口:

```
public interface Bank extends Payment {
    void initiatePayments();
}
```

```
public interface Loan extends Payment {
    void intiateLoanSettlement();
    void initiateRePayment();
}
```

以及各自的实现，从`BankPayment`开始:

```
public class BankPayment implements Bank {

    @Override
    public void initiatePayments() {
        // ...
    }

    @Override
    public Object status() {
        // ...
    }

    @Override
    public List<Object> getPayments() {
        // ...
    }
}
```

最后，我们修改后的`LoanPayment`实施:

```
public class LoanPayment implements Loan {

    @Override
    public void intiateLoanSettlement() {
        // ...
    }

    @Override
    public void initiateRePayment() {
        // ...
    }

    @Override
    public Object status() {
        // ...
    }

    @Override
    public List<Object> getPayments() {
        // ...
    }
}
```

现在，让我们回顾一下新的类图:

[![](img/404c74c4538f8a40d0ddca4f77413cdd.png)](/web/20220526053728/https://www.baeldung.com/wp-content/uploads/2020/07/interface_segregation_fixed.png)

如我们所见，这些接口并没有违反原则。实现不必提供空方法。这保持了代码的整洁，减少了出错的机会。

## 6.结论

在本教程中，我们看了一个简单的场景，在这个场景中，我们首先背离了接口分离原则，并看到了这种背离所导致的问题。然后，我们展示了如何正确应用该原则，以避免这些问题。

如果我们正在处理我们无法修改的被污染的遗留接口，那么[适配器模式](/web/20220526053728/https://www.baeldung.com/java-adapter-pattern)可以派上用场。

界面分离原则是设计和开发应用程序时的一个重要概念。遵循这一原则有助于避免具有多重职责的臃肿接口。这最终也有助于我们遵循单一责任原则。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220526053728/https://github.com/eugenp/tutorials/tree/master/patterns/solid)