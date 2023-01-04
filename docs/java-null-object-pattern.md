# 空对象模式介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-null-object-pattern>

## 1.概观

在这个快速教程中，我们将看看空对象模式，这是[策略模式](/web/20221208203735/https://www.baeldung.com/java-strategy-pattern)的一个特例。我们将描述它的用途以及何时应该考虑使用它。

像往常一样，我们还将提供一个简单的代码示例。

## 2.空对象模式

在大多数面向对象的编程语言中，我们不允许使用`null`引用。这就是为什么我们经常被迫开`null`支票:

```java
Command cmd = getCommand();
if (cmd != null) {
    cmd.execute();
}
```

有时，如果这种`if`语句的数量增加，代码可能会变得难看、难以阅读并且容易出错。这就是空对象模式派上用场的时候了。

**空对象模式的目的是最小化那种`null`检查。**相反，我们可以识别空行为，并将其封装在客户端代码所期望的类型中。通常情况下，这种中立的逻辑非常简单——什么都不做。这样我们不再需要对`null`引用进行特殊处理。

我们可以简单地对待空对象，就像对待给定类型的任何其他实例一样，这些实例实际上包含一些更复杂的业务逻辑。因此，客户端代码保持更干净。

因为空对象不应该有任何状态，所以没有必要多次创建相同的实例。因此，我们通常将**实现的** **空对象称为** [**单线态**](/web/20221208203735/https://www.baeldung.com/java-singleton) 。

## 3.空对象模式的 UML 图

让我们直观地看看这个模式:

[![NOP](img/c53bac6d3958a1839b21d68a5878c5b8.png)](/web/20221208203735/https://www.baeldung.com/wp-content/uploads/2019/03/NOP.jpg)

如我们所见，我们可以确定以下参与者:

*   `Client`需要一个`AbstractObject`的实例
*   `AbstractObject`定义了`Client`期望的契约——它可能还包含实现类的共享逻辑
*   `RealObject`实现`AbstractObject`并提供真实的行为
*   `NullObject`实现`AbstractObject`并提供中性行为

## 4.履行

现在我们对这个理论有了一个清晰的概念，让我们来看一个例子。

假设我们有一个消息路由器应用程序。应为每条消息分配有效的优先级。我们的系统应该将高优先级的消息路由到 SMS 网关，而中等优先级的消息应该路由到 JMS 队列。

然而，**有时，具有“未定义”或空优先级的消息可能会**到达我们的应用程序。这样的消息应该被丢弃，不再进一步处理。

首先，我们将创建`Router`接口:

```java
public interface Router {
    void route(Message msg);
}
```

接下来，让我们创建上述接口的两个实现——一个负责路由到 SMS 网关，另一个将消息路由到 JMS 队列:

```java
public class SmsRouter implements Router {
    @Override
    public void route(Message msg) {
        // implementation details
    }
}
```

```java
public class JmsRouter implements Router {
    @Override
    public void route(Message msg) {
        // implementation details
    }
}
```

最后，**让我们实现我们的空对象:**

```java
public class NullRouter implements Router {
    @Override
    public void route(Message msg) {
        // do nothing
    }
}
```

我们现在准备把所有的部分放在一起。让我们看看示例客户端代码可能是什么样子的:

```java
public class RoutingHandler {
    public void handle(Iterable<Message> messages) {
        for (Message msg : messages) {
            Router router = RouterFactory.getRouterForMessage(msg);
            router.route(msg);
        }
    }
}
```

正如我们所看到的，**我们以同样的方式对待所有的`Router`对象，**不管`RouterFactory.`返回什么实现，这允许我们保持代码的整洁和可读性。

## 5.何时使用空对象模式

**当`Client`检查`null` 只是为了跳过执行或执行默认动作时，我们应该使用空对象模式。在这种情况下，我们可以将中性逻辑封装在一个空对象中，并将其返回给客户端，而不是返回`null`值。这样，客户机的代码不再需要知道给定的实例是否是`null`。**

这种方法遵循一般的面向对象原则，比如[告诉-不要问](https://web.archive.org/web/20221208203735/https://martinfowler.com/bliki/TellDontAsk.html)。

为了更好地理解我们什么时候应该使用空对象模式，让我们假设我们必须实现如下定义的`CustomerDao`接口:

```java
public interface CustomerDao {
    Collection<Customer> findByNameAndLastname(String name, String lastname);
    Customer getById(Long id);
}
```

大多数开发人员会从`findByNameAndLastname()`返回`Collections.emptyList()`，以防没有客户匹配提供的搜索标准。这是遵循空对象模式的一个很好的例子。

相反，`get` `ById()` 应该返回给定 id 的客户。调用此方法的人希望获得特定的客户实体。**如果不存在这样的客户，我们应该明确返回`null`以表明所提供的 id 有问题。**

与所有其他模式一样，**在盲目实现空对象模式**之前，我们需要考虑我们的特定用例。否则，我们可能会无意中在代码中引入一些难以发现的错误。

## 6.结论

在本文中，我们了解了什么是空对象模式，以及何时可以使用它。我们还实现了一个简单的设计模式示例。

像往常一样，所有的代码样本都可以在 [GitHub](https://web.archive.org/web/20221208203735/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-behavioral) 上找到。