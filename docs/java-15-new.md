# Java 15 的新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-15-new>

[This article is part of a series:](javascript:void(0);)[• New Features in Java 8](/web/20220824084043/https://www.baeldung.com/java-8-new-features)
[• New Features in Java 9](/web/20220824084043/https://www.baeldung.com/new-java-9)
[• New Features in Java 10](/web/20220824084043/https://www.baeldung.com/java-10-overview)
[• New Features in Java 11](/web/20220824084043/https://www.baeldung.com/java-11-new-features)
[• New Features in Java 12](/web/20220824084043/https://www.baeldung.com/java-12-new-features)
[• New Features in Java 13](/web/20220824084043/https://www.baeldung.com/java-13-new-features)
[• New Features in Java 14](/web/20220824084043/https://www.baeldung.com/java-14-new-features)
• What's New in Java 15 (current article)[• New Features in Java 16](/web/20220824084043/https://www.baeldung.com/java-16-new-features)
[• New Features in Java 17](/web/20220824084043/https://www.baeldung.com/java-17-new-features)

## 1.介绍

Java 15 将于 2020 年 9 月全面上市，是 JDK 平台的下一个短期版本。它建立在早期版本的几个特性之上，并提供了一些新的增强功能。

在本帖中，我们将看看 Java 15 的一些新特性，以及 Java 开发者感兴趣的其他变化。

## 2.记录(JEP 384)

**`record`是 Java 中的一种新类型的类，它使得创建不可变数据对象变得容易。**

最初在 Java 14 中引入作为早期预览，Java 15 的目标是在成为正式产品功能之前[细化几个方面](https://web.archive.org/web/20220824084043/https://openjdk.java.net/jeps/384)。

让我们看一个使用当前 Java 的例子，以及它如何随着记录`.`而改变

### 2.1.没有记录

在记录之前，我们将创建一个不可变的数据传输对象(DTO ),如下所示:

```java
public class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```

请注意，这里有很多代码来创建一个不可变的对象，它实际上只保存状态。我们所有的字段都是使用`final`显式定义的，我们有一个单一的全参数构造函数，并且我们对每个字段都有一个访问器方法。在某些情况下，我们甚至可以将类本身声明为`final`以防止任何子类化。

在许多情况下，我们还会更进一步，覆盖`toString`方法来提供有意义的日志输出。我们可能还想覆盖`equals`和`hashCode`方法，以避免在比较这些对象的两个实例时出现意外的结果。

### 2.2.有记录

使用新的`record`类，我们可以用更简洁的方式定义相同的不可变数据对象:

```java
public record Person(String name, int age) {
}
```

这里发生了一些事情。首先也是最重要的是，**类定义有一个新的语法，它是专门针对`record` s** 的。在这个标题中，我们提供了记录中字段的详细信息。

使用这个头，编译器可以推断出内部字段。这意味着我们不需要定义特定的成员变量和访问器，因为它们是默认提供的。我们也不必提供构造函数。

此外，编译器为`toString`、`equals`和`hashCode`方法提供了合理的实现。

虽然`record` s 消除了许多样板代码，**它们确实允许我们覆盖一些默认行为**。例如，我们可以定义一个进行验证的规范构造函数:

```java
public record Person(String name, int age) {
    public Person {
        if(age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
    }
}
```

值得一提的是`record` s 确实有一些限制。除此之外，它们总是`final`，不能被声明为`abstract`，也不能使用[原生方法](/web/20220824084043/https://www.baeldung.com/java-native)。

## 3.密封类(JEP 360)

目前， **Java 没有提供对继承**的细粒度控制。像`public`、`protected`、`private`这样的访问修饰符，以及默认的包私有，提供了非常粗粒度的控制。

为此， [`sealed`类](https://web.archive.org/web/20220824084043/https://openjdk.java.net/jeps/360)的目标是允许单个类声明哪些类型可以用作子类型。这也适用于接口和确定哪些类型可以实现它们。

密封类包含两个新关键字— `sealed`和`permits`:

```java
public abstract sealed class Person
    permits Employee, Manager {

    //...
}
```

在这个例子中，我们声明了一个名为`Person.` 的`abstract`类，我们还指定了唯一可以扩展它的类是`Employee`和`Manager`。使用关键字`extends`扩展`sealed`类就像今天在 Java 中一样:

```java
public final class Employee extends Person {
}

public non-sealed class Manager extends Person {
}
```

需要注意的是，**任何扩展了`sealed`类的类本身必须声明为`sealed`、`non-sealed`或`final`** 。这确保了类的层次结构是有限的，并且为编译器所知。

**这种有限且详尽的层次结构是使用`sealed`类**的最大好处之一。让我们来看一个实际例子:

```java
if (person instanceof Employee) {
    return ((Employee) person).getEmployeeId();
} 
else if (person instanceof Manager) {
    return ((Manager) person).getSupervisorId();
}
```

没有一个密封的类，**编译器不能合理地确定所有可能的子类都被我们的 `if-else`语句覆盖了**。如果末尾没有一个`else`子句，编译器可能会发出警告，指出我们的逻辑没有涵盖所有情况。

## 4.隐藏类(JEP 371)

Java 15 中引入的一个新特性叫做[隐藏类](https://web.archive.org/web/20220824084043/https://openjdk.java.net/jeps/371)。虽然大多数开发人员不会从中直接受益，但任何使用动态字节码或 JVM 语言的人都可能会发现它们很有用。

隐藏类的目标是允许运行时创建不可发现的类。这意味着它们不能被其他类链接，也不能通过[反射](/web/20220824084043/https://www.baeldung.com/java-reflection)被发现。诸如此类的类通常具有较短的生命周期，因此，隐藏类被设计为在加载和卸载时都是高效的。

注意，当前版本的 Java 确实允许创建类似于隐藏类的匿名类。然而，他们依赖于 [`Unsafe`](/web/20220824084043/https://www.baeldung.com/java-unsafe) API。隐藏类没有这种依赖性。

## 5.模式匹配类型检查(JEP 375)

在 Java 14 中预览了[模式匹配特性](https://web.archive.org/web/20220824084043/https://openjdk.java.net/jeps/375)，Java 15 旨在继续其预览状态，没有新的增强。

回顾一下，这个特性的目标是删除大量通常带有`instanceof`操作符的样板代码:

```java
if (person instanceof Employee) {
    Employee employee = (Employee) person;
    Date hireDate = employee.getHireDate();
    //...
}
```

这是 Java 中非常常见的模式。每当我们检查一个变量是否是某种类型时，我们几乎总是在它后面加上一个到该类型的强制转换。

模式匹配特性通过引入新的`binding variable`简化了这一过程:

```java
if (person instanceof Employee employee) {
    Date hireDate = employee.getHireDate();
    //...
}
```

注意我们如何提供一个新的变量名`employee`，作为类型检查的一部分。如果类型检查是`true`，那么**JVM 自动为我们转换变量，并将结果赋给新的绑定变量**。

我们还可以将新的绑定变量与条件语句结合起来:

```java
if (person instanceof Employee employee && employee.getYearsOfService() > 5) {
    //...
}
```

在未来的 Java 版本中，目标是将模式匹配扩展到其他语言特性，比如`switch`语句。

## 6.外部内存 API (JEP 383)

[外来内存访问](https://web.archive.org/web/20220824084043/https://openjdk.java.net/jeps/383)已经是 Java 14 的一个孵化特性。在 Java 15 中，目标是在添加几个新特性的同时继续其孵化状态:

*   一个新的*变量句柄* API，用于定制内存访问变量句柄
*   使用*分割符*接口支持内存段的并行处理
*   增强了对`mapped`内存段的支持
*   操纵和取消引用来自本机调用的地址的能力

外来内存通常是指位于托管 JVM 堆之外的内存。因此，它不受垃圾收集的影响，通常可以处理非常大的内存段。

虽然这些新的 API 可能不会直接影响大多数开发人员，但它们将为处理外来内存的第三方库提供很多价值。这包括分布式缓存、非规范化文档存储、大型任意字节缓冲区、内存映射文件等等。

## 7.垃圾收集者

在 Java 15 中， [ZGC](/web/20220824084043/https://www.baeldung.com/jvm-zgc-garbage-collector) (JEP 377)和谢南多阿(JEP 379)都将不再是实验。两者都是支持的配置，团队可以选择使用，而 G1 收集器将保持默认设置。

这两种方法以前都是使用实验性特征标志来实现的。这种方法允许开发人员测试新的垃圾收集器并提交反馈，而无需下载单独的 JDK 或插件。

关于 [Shenandoah](https://web.archive.org/web/20220824084043/https://openjdk.java.net/jeps/379) 有一点需要注意:它并不是所有厂商的 JDK 都提供的——最明显的是，甲骨文 JDK 公司并不包括它。

## 8.其他变化

Java 15 中还有其他几个值得注意的变化。

经过 Java 13 和 14 的多轮预览，[文本块](/web/20220824084043/https://www.baeldung.com/java-text-blocks)将是 Java 15 中完全支持的产品特性。

[有用的空指针异常](/web/20220824084043/https://www.baeldung.com/java-14-nullpointerexception)，最初在 JEP 358 下的 Java 14 中提供，现在默认启用。

传统的`DatagramSocket` API 已经被重写。这是在 Java 14 中重写`Socket` API 的后续。虽然它不会影响大多数开发人员，但它很有趣，因为它是[项目的先决条件。](/web/20220824084043/https://www.baeldung.com/openjdk-project-loom)

同样值得注意的是，Java 15 包括对 Edwards 曲线数字签名算法的加密支持。EdDSA 是一个现代椭圆曲线签名方案，它比 JDK 中现有的签名方案有几个优点。

最后，Java 15 中有几个东西已经被弃用了。在未来的版本中，偏向锁定、Solaris/SPARC 端口和 RMI 激活都被删除或计划删除。

值得注意的是，最初在 Java 8 中引入的 Nashorn JavaScript 引擎现在被删除了。随着最近 GraalVM 和其他 VM 技术的引入，Nashorn 在 JDK 生态系统中显然不再有一席之地。

## 9.结论

Java 15 建立在过去版本的几个特性之上，包括记录、文本块、新的垃圾收集算法等等。**还增加了新的预览功能，包括密封类和隐藏类**。

由于 Java 15 不是一个长期支持的版本，我们预计对它的支持将在 2021 年 3 月结束。到那时，我们可以期待 Java 16，随后很快会有一个新的长期支持版本 Java 17。

Next **»**[New Features in Java 16](/web/20220824084043/https://www.baeldung.com/java-16-new-features)**«** Previous[New Features in Java 14](/web/20220824084043/https://www.baeldung.com/java-14-new-features)