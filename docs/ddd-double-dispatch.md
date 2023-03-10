# DDD 的双重派遣

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ddd-double-dispatch>

## 1。概述

双重分派是一个技术术语，用来描述基于接收者和参数类型选择调用方法的**过程。**

很多开发人员经常混淆双重分派和[策略模式](https://web.archive.org/web/20220813055833/https://en.wikipedia.org/wiki/Strategy_pattern)。

Java 不支持双重分派，但是我们可以使用一些技术来克服这个限制。

在本教程中，我们将重点展示在领域驱动设计(DDD)和策略模式的背景下双重分派的例子。

## 2。双重调度

在我们讨论双重分派之前，让我们回顾一些基础知识，并解释什么是单一分派。

### 2.1.单一调度

**Single dispatch 是一种基于接收方运行时类型来选择实现方法的方式。**在 Java 中，这基本上和多态性是一回事。

例如，让我们看看这个简单的折扣政策界面:

```java
public interface DiscountPolicy {
    double discount(Order order);
}
```

`DiscountPolicy`接口有两个实现。平坦的，总是返回相同的折扣:

```java
public class FlatDiscountPolicy implements DiscountPolicy {
    @Override
    public double discount(Order order) {
        return 0.01;
    }
}
```

第二个实现根据订单的总成本返回折扣:

```java
public class AmountBasedDiscountPolicy implements DiscountPolicy {
    @Override
    public double discount(Order order) {
        if (order.totalCost()
            .isGreaterThan(Money.of(CurrencyUnit.USD, 500.00))) {
            return 0.10;
        } else {
            return 0;
        }
    }
}
```

为了这个例子的需要，让我们假设`Order`类有一个`totalCost()`方法。

**现在，Java 中的单一分派只是一个众所周知的多态行为，在下面的测试中得到了证明:**

```java
@DisplayName(
    "given two discount policies, " +
    "when use these policies, " +
    "then single dispatch chooses the implementation based on runtime type"
    )
@Test
void test() throws Exception {
    // given
    DiscountPolicy flatPolicy = new FlatDiscountPolicy();
    DiscountPolicy amountPolicy = new AmountBasedDiscountPolicy();
    Order orderWorth501Dollars = orderWorthNDollars(501);

    // when
    double flatDiscount = flatPolicy.discount(orderWorth501Dollars);
    double amountDiscount = amountPolicy.discount(orderWorth501Dollars);

    // then
    assertThat(flatDiscount).isEqualTo(0.01);
    assertThat(amountDiscount).isEqualTo(0.1);
}
```

如果这一切看起来很简单，请继续关注。我们稍后会用同样的例子。

我们现在准备引入双重调度。

### 2.2.双重分派与方法重载

**Double dispatch 根据接收方类型和参数类型确定运行时调用的方法**。

Java 不支持双重调度。

**注意，双重分派经常与方法重载混淆，两者不是一回事**。方法重载只根据编译时信息选择要调用的方法，比如变量的声明类型。

以下示例详细解释了这种行为。

我们来介绍一个新的折扣界面，名为`SpecialDiscountPolicy`:

```java
public interface SpecialDiscountPolicy extends DiscountPolicy {
    double discount(SpecialOrder order);
}
```

`SpecialOrder`只是扩展了`Order`，没有添加新的行为。

现在，当我们创建一个`SpecialOrder`的实例，但将其声明为普通的`Order`时，则不使用特殊折扣方法:

```java
@DisplayName(
    "given discount policy accepting special orders, " +
    "when apply the policy on special order declared as regular order, " +
    "then regular discount method is used"
    )
@Test
void test() throws Exception {
    // given
    SpecialDiscountPolicy specialPolicy = new SpecialDiscountPolicy() {
        @Override
        public double discount(Order order) {
            return 0.01;
        }

        @Override
        public double discount(SpecialOrder order) {
            return 0.10;
        }
    };
    Order specialOrder = new SpecialOrder(anyOrderLines());

    // when
    double discount = specialPolicy.discount(specialOrder);

    // then
    assertThat(discount).isEqualTo(0.01);
}
```

因此，方法重载不是双重分派。

即使 Java 不支持双重分派，我们也可以用一个模式来实现类似的行为: [Visitor](https://web.archive.org/web/20220813055833/https://en.wikipedia.org/wiki/Visitor_pattern) 。

### 2.3.访问者模式

访问者模式允许我们在不修改现有类的情况下向它们添加新的行为。多亏了模仿双重分派的巧妙技术，这才成为可能。

让我们暂时离开折扣的例子，这样我们可以介绍访问者模式。

**假设我们想要为每种订单使用不同的模板生成 HTML 视图**。我们可以将这种行为直接添加到 order 类中，但这不是最好的主意，因为这违反了 SRP。

相反，我们将使用访问者模式。

首先，我们需要介绍一下`Visitable`接口:

```java
public interface Visitable<V> {
    void accept(V visitor);
}
```

我们还将使用一个访问者接口，在我们名为`OrderVisitor`的案例中:

```java
public interface OrderVisitor {
    void visit(Order order);
    void visit(SpecialOrder order);
}
```

然而，访问者模式的一个缺点是它需要可访问的类来感知访问者。

如果类不是为支持访问者而设计的，那么应用这种模式可能会很困难(如果没有源代码，甚至是不可能的)。

每个订单类型都需要实现`Visitable`接口，并提供自己的看似相同的实现，这是另一个缺点。

注意添加到`Order`和`SpecialOrder`的方法是相同的:

```java
public class Order implements Visitable<OrderVisitor> {
    @Override
    public void accept(OrderVisitor visitor) {
        visitor.visit(this);        
    }
}

public class SpecialOrder extends Order {
    @Override
    public void accept(OrderVisitor visitor) {
        visitor.visit(this);
    }
}
```

**不在子类中重新实现`accept` 可能很诱人。然而，如果我们不这样做，那么`OrderVisitor.visit(Order)`方法将总是被使用，当然，由于多态性。**

最后，让我们看看负责创建 HTML 视图的`OrderVisitor`的实现:

```java
public class HtmlOrderViewCreator implements OrderVisitor {

    private String html;

    public String getHtml() {
        return html;
    }

    @Override
    public void visit(Order order) {
        html = String.format("<p>Regular order total cost: %s</p>", order.totalCost());
    }

    @Override
    public void visit(SpecialOrder order) {
        html = String.format("<h1>Special Order</h1><p>total cost: %s</p>", order.totalCost());
    }

}
```

以下示例演示了`HtmlOrderViewCreator`的使用:

```java
@DisplayName(
        "given collection of regular and special orders, " +
        "when create HTML view using visitor for each order, " +
        "then the dedicated view is created for each order"   
    )
@Test
void test() throws Exception {
    // given
    List<OrderLine> anyOrderLines = OrderFixtureUtils.anyOrderLines();
    List<Order> orders = Arrays.asList(new Order(anyOrderLines), new SpecialOrder(anyOrderLines));
    HtmlOrderViewCreator htmlOrderViewCreator = new HtmlOrderViewCreator();

    // when
    orders.get(0)
        .accept(htmlOrderViewCreator);
    String regularOrderHtml = htmlOrderViewCreator.getHtml();
    orders.get(1)
        .accept(htmlOrderViewCreator);
    String specialOrderHtml = htmlOrderViewCreator.getHtml();

    // then
    assertThat(regularOrderHtml).containsPattern("<p>Regular order total cost: .*</p>");
    assertThat(specialOrderHtml).containsPattern("<h1>Special Order</h1><p>total cost: .*</p>");
}
```

## 3.DDD 的双重派遣

在前面的部分中，我们讨论了双重分派和访问者模式。

现在我们终于准备好展示如何在 DDD 中使用这些技术了。

让我们回到订单和折扣政策的例子。

### 3.1.作为战略模式的折扣政策

前面，我们介绍了计算所有订单行项目总和的`Order`类及其`totalCost()`方法:

```java
public class Order {
    public Money totalCost() {
        // ...
    }
}
```

还有一个`DiscountPolicy`接口来计算订单的折扣。引入该接口是为了允许使用不同的折扣策略，并在运行时更改它们。

这种设计比简单地将所有可能的折扣策略硬编码到 *Order* 类中灵活得多:

```java
public interface DiscountPolicy {
    double discount(Order order);
}
```

到目前为止，我们还没有明确提到这一点，但是这个例子使用了[策略模式](https://web.archive.org/web/20220813055833/https://en.wikipedia.org/wiki/Strategy_pattern)。DDD 经常使用这种模式来符合[无处不在的语言](https://web.archive.org/web/20220813055833/https://martinfowler.com/bliki/UbiquitousLanguage.html)原则，实现低耦合。在 DDD 世界，战略模式通常被称为政策。

让我们看看如何结合双派遣技术和折扣政策。

### 3.2.双重派遣和折扣政策

为了正确使用策略模式，将它作为参数传递通常是个好主意。这种方法遵循支持更好封装的[告诉，不要问](https://web.archive.org/web/20220813055833/https://martinfowler.com/bliki/TellDontAsk.html)原则。

例如，`Order`类可以像这样实现`totalCost` :

```java
public class Order /* ... */ {
    // ...
    public Money totalCost(SpecialDiscountPolicy discountPolicy) {
        return totalCost().multipliedBy(1 - discountPolicy.discount(this), RoundingMode.HALF_UP);
    }
    // ...
}
```

现在，让我们假设我们想要不同地处理每种类型的订单。

例如，在计算特殊订单的折扣时，有一些其他规则需要针对`SpecialOrder`类的信息。我们希望避免投射和反射，同时能够使用正确应用的折扣计算每个`Order`的总成本。

我们已经知道方法重载发生在编译时。因此，一个自然的问题出现了:我们如何根据订单的运行时类型动态地将订单折扣逻辑分派给正确的方法？

**答案？我们需要稍微修改订单类。**

根`Order`类需要在运行时分派给折扣策略参数。实现这一点最简单的方法是添加一个受保护的`applyDiscountPolicy`方法:

```java
public class Order /* ... */ {
    // ...
    public Money totalCost(SpecialDiscountPolicy discountPolicy) {
        return totalCost().multipliedBy(1 - applyDiscountPolicy(discountPolicy), RoundingMode.HALF_UP);
    }

    protected double applyDiscountPolicy(SpecialDiscountPolicy discountPolicy) {
        return discountPolicy.discount(this);
    }
   // ...
}
```

由于这种设计，我们避免了在`Order`子类的`totalCost`方法中复制业务逻辑。

让我们演示一下用法:

```java
@DisplayName(
    "given regular order with items worth $100 total, " +
    "when apply 10% discount policy, " +
    "then cost after discount is $90"
    )
@Test
void test() throws Exception {
    // given
    Order order = new Order(OrderFixtureUtils.orderLineItemsWorthNDollars(100));
    SpecialDiscountPolicy discountPolicy = new SpecialDiscountPolicy() {

        @Override
        public double discount(Order order) {
            return 0.10;
        }

        @Override
        public double discount(SpecialOrder order) {
            return 0;
        }
    };

    // when
    Money totalCostAfterDiscount = order.totalCost(discountPolicy);

    // then
    assertThat(totalCostAfterDiscount).isEqualTo(Money.of(CurrencyUnit.USD, 90));
}
```

这个例子仍然使用 Visitor 模式，只是稍微修改了一下。订单类知道`SpecialDiscountPolicy`(访问者)有某种意义并计算折扣。

**如前所述，我们希望能够基于`Order`的运行时类型应用不同的折扣规则。因此，我们需要在每个子类中覆盖受保护的`applyDiscountPolicy`方法。**

让我们在`SpecialOrder`类中覆盖这个方法:

```java
public class SpecialOrder extends Order {
    // ...
    @Override
    protected double applyDiscountPolicy(SpecialDiscountPolicy discountPolicy) {
        return discountPolicy.discount(this);
    }
   // ...
}
```

我们现在可以使用折扣政策中关于`SpecialOrder`的额外信息来计算正确的折扣:

```java
@DisplayName(
    "given special order eligible for extra discount with items worth $100 total, " +
    "when apply 20% discount policy for extra discount orders, " +
    "then cost after discount is $80"
    )
@Test
void test() throws Exception {
    // given
    boolean eligibleForExtraDiscount = true;
    Order order = new SpecialOrder(OrderFixtureUtils.orderLineItemsWorthNDollars(100), 
      eligibleForExtraDiscount);
    SpecialDiscountPolicy discountPolicy = new SpecialDiscountPolicy() {

        @Override
        public double discount(Order order) {
            return 0;
        }

        @Override
        public double discount(SpecialOrder order) {
            if (order.isEligibleForExtraDiscount())
                return 0.20;
            return 0.10;
        }
    };

    // when
    Money totalCostAfterDiscount = order.totalCost(discountPolicy);

    // then
    assertThat(totalCostAfterDiscount).isEqualTo(Money.of(CurrencyUnit.USD, 80.00));
}
```

此外，由于我们在订单类中使用多态行为，我们可以很容易地修改总成本计算方法。

## 4。结论

在本文中，我们学习了如何在领域驱动设计中使用双重分派技术和`Strategy`(又名`Policy`)模式。

GitHub 上的[提供了所有示例的完整源代码。](https://web.archive.org/web/20220813055833/https://github.com/eugenp/tutorials/tree/master/ddd)