# Java 中的干净编码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-clean-code>

## 1.概观

在本教程中，我们将介绍干净的编码原则。我们还将理解为什么干净的代码很重要，以及如何在 Java 中实现这一点。此外，我们将看看是否有任何工具可以帮助我们。

## 2.什么是干净代码？

所以，在我们进入干净代码的细节之前，让我们理解干净代码是什么意思。老实说，对此不可能有一个好的答案。在编程中，一些关注点相互交叉，因此产生了通用原则。但是，每种编程语言和范式都有自己的一套细微差别，这要求我们采用合适的实践。

广义来说，**干净的代码可以概括为任何开发人员都可以轻松阅读和更改的代码**。虽然这听起来像是一个过于简单的概念，但我们将在本教程的后面看到这是如何建立的。无论我们在哪里听到干净的代码，我们可能会遇到一些马丁福勒的参考。以下是他在其中一个地方描述干净代码的方式:

> `Any fool can write code that a computer can understand. Good programmers write code that humans can understand.`

## 3.我们为什么要关心干净的代码？

编写干净的代码既是个人习惯问题，也是技能问题。作为一名开发人员，我们随着时间的推移通过经验和知识而成长。但是，我们必须问为什么我们要投资开发干净的代码？我们知道其他人可能会发现阅读我们的代码更容易，但是这就足够了吗？让我们来了解一下！

干净的编码原则帮助我们实现许多与我们打算生产的软件相关的理想目标。让我们通过它们来更好地理解它:

*   我们开发的任何软件都有一个有效的生命周期，在此期间都需要修改和维护。干净的代码**可以帮助开发软件，随着时间的推移，软件容易改变和维护**。
*   由于各种内部或外部因素，软件可能会表现出非预期的行为。在修复和可用性方面，它可能经常需要快速周转。用干净的编码原则开发的软件更容易解决问题。
*   软件在其生命周期中会有许多开发人员创建、更新和维护它，开发人员在不同的时间点加入进来。这需要**更快的入职以保持高生产率**，干净的代码有助于实现这一目标。

## 4.干净代码的特征

用干净编码原则编写的代码库展示了几个使它们与众不同的特征。让我们来了解一下这些特征:

*   `Focused`:应该写一段**代码来解决一个特定的问题**。它不应该做任何与解决给定问题严格无关的事情。这适用于代码库中的所有抽象层次，如方法、类、包或模块。
*   这是干净代码最重要也是最容易被忽视的特征。软件**的设计和实现必须尽可能简单**，这可以帮助我们实现预期的结果。代码库中不断增加的复杂性使得它们容易出错，难以阅读和维护。
*   干净的代码虽然简单，但必须解决手边的问题。测试代码库必须**直观且容易，最好是以自动化的方式**。这有助于建立代码库的基线行为，并使得在不破坏任何东西的情况下更改它变得更加容易。

这些都有助于我们实现上一节讨论的目标。与以后重构相比，开始开发时考虑这些特征是有益的。这使得软件生命周期的总拥有成本更低。

## 5.Java 中的干净编码

现在我们已经了解了足够多的背景知识，让我们看看如何在 Java 中融入干净的编码原则。Java 提供了许多可以帮助我们编写干净代码的最佳实践。我们将把它们归类到不同的类别中，并理解如何用代码示例编写干净的代码。

### 5.1.项目结构

虽然 Java 不强制任何项目结构，**遵循一致的模式来组织我们的源文件、测试、配置、数据和其他代码工件总是有用的**。Maven，一个流行的 Java 构建工具，[规定了一个特定的项目结构](https://web.archive.org/web/20220625172621/https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)。虽然我们可能不使用 Maven，但是遵守约定总是好的。

让我们看看 Maven 建议我们创建的一些文件夹:

*   `src/main/java`:针对源文件
*   `src/main/resources`:对于资源文件，如属性
*   `src/test/java`:测试源文件
*   `src/test/resources`:用于测试资源文件，如属性

类似的，还有其他流行的项目结构，比如 Bazel T1，我们应该根据自己的需求和受众来选择。

### 5.2.命名约定

遵循**命名约定可以使我们的代码可读性更强，从而更易维护**。Spring 的创造者 Rod Johnson 强调了[命名约定](https://web.archive.org/web/20220625172621/https://blog.atomist.com/eighteen-years-of-spring/)在 Spring 中的重要性:

> `“… if you know what something does, you got a pretty good chance guessing the name of the Spring class or interface for it …”`

Java [规定了一套规则](https://web.archive.org/web/20220625172621/https://www.oracle.com/technetwork/java/codeconventions-135099.html)在用 Java 命名任何东西时都要遵守。一个格式良好的名称不仅有助于阅读代码，而且还传达了许多关于代码意图的信息。让我们举一些例子:

*   面向对象概念中的类是对象的蓝图，这些对象通常代表现实世界中的对象。因此，使用名词来命名充分描述它们的类是有意义的:

```java
public class Customer {
}
```

*   `Variables`:Java 中的变量捕获从类中创建的对象的状态。变量的名称应该清楚地描述变量的意图:

```java
public class Customer {
    private String customerName;
}
```

*   Java 中的方法总是类的一部分，因此通常表示对从类中创建的对象的状态的动作。因此，使用动词命名方法[很有用:](/web/20220625172621/https://www.baeldung.com/java-pojo-class#javabeans)

```java
public class Customer {
    private String customerName;
    public String getCustomerName() {
        return this.customerName;
    }
}
```

虽然我们只讨论了如何在 Java 中命名标识符，但是请注意还有其他的最佳实践，比如 camel case，为了可读性，我们应该遵守这一点。还有更多关于命名接口、枚举、常量的约定。

### 5.3.源文件结构

源文件可以包含不同的元素。虽然 Java **编译器强制执行一些结构，但是大部分是不固定的**。但是坚持在源文件中放置元素的特定顺序可以显著提高代码的可读性。有多种流行的风格指南可以从中获取灵感，比如 T2 谷歌 T3 和 T4 春天 T5。

让我们看看源文件中元素的典型排序应该是什么样子:

*   程序包语句
*   导入报表
    *   所有静态导入
    *   所有非静态导入
*   恰好一个顶级类
    *   类别变量
    *   实例变量
    *   构造器
    *   方法

除此之外，**方法可以根据它们的功能或范围**进行分组。没有一个好的惯例，这个想法应该是**决定一次，然后始终如一地遵循。**

让我们来看一个格式良好的源文件:

```java
# /src/main/java/com/baeldung/application/entity/Customer.java
package com.baeldung.application.entity;

import java.util.Date;

public class Customer {
    private String customerName;
    private Date joiningDate;
    public Customer(String customerName) {
        this.customerName = customerName;
        this.joiningDate = new Date();
    }

    public String getCustomerName() { 
        return this.customerName; 
    }

    public Date getJoiningDate() {
        return this.joiningDate;
    }
}
```

### 5.4.空白

我们都知道，与大段文字相比，短段落更容易阅读和理解。在阅读代码方面也没有太大的不同。放置恰当且一致的空白和空白行可以增强代码的可读性。

这里的想法是在代码中引入逻辑分组，这有助于在试图通读时组织思维过程。这里没有一个单一的规则可以采用，而是一组通用的指导原则和保持可读性的内在意图:

*   开始静态块、字段、构造函数和内部类之前的两个空行
*   多行方法签名后的一个空行
*   用一个空格将 if、for、catch 等保留关键字与左括号分隔开
*   用一个空格分隔保留的关键字，如 else、catch 和右括号

这里的列表并不详尽，但应该给我们一个前进的方向。

### 5.5.刻痕

虽然很琐碎，但是几乎所有的开发人员都可以保证这样一个事实:一个良好缩进的代码更容易阅读和理解。Java 中没有单一的代码缩进约定。这里的关键是要么采用一个流行的约定，要么定义一个私有的约定，然后在整个组织中一致地遵循它。

让我们看看一些重要的缩进标准:

*   典型的最佳实践是使用四个空格，一个缩进单位。请注意，有些指南建议用制表符代替空格。虽然这里没有绝对的最佳实践，但关键仍然是一致性！
*   通常情况下，应该有一个超过行长度的上限，但由于开发人员现在使用更大的屏幕，这可以设置得比传统的 80 更高。
*   最后，由于许多表达式不适合放在一行中，我们必须始终如一地拆分它们:
    *   在逗号后中断方法调用
    *   在运算符前中断表达式
    *   缩进换行以获得更好的可读性(我们在 Baeldung 更喜欢两个空格)

让我们看一个例子:

```java
List<String> customerIds = customer.stream()
  .map(customer -> customer.getCustomerId())
  .collect(Collectors.toCollection(ArrayList::new));
```

### 5.6.方法参数

参数对于方法按照规范工作是必不可少的。但是，**一长串的参数会让人难以阅读和理解代码**。那么，我们应该在哪里划线呢？让我们了解一下可能对我们有所帮助的最佳实践:

*   尝试限制一个方法接受的参数数量，三个参数可能是一个不错的选择
*   如果方法需要比推荐的参数更多的参数，考虑[重构](/web/20220625172621/https://www.baeldung.com/cs/refactoring)该方法，通常一个长的参数列表也表明该方法可能在做多件事情
*   我们可以考虑将参数绑定到自定义类型中，但是必须注意不要将不相关的参数转储到单一类型中
*   最后，虽然我们应该使用这个建议来判断代码的可读性，但我们不能对此过于迂腐

让我们来看一个例子:

```java
public boolean setCustomerAddress(String firstName, String lastName, String streetAddress, 
  String city, String zipCode, String state, String country, String phoneNumber) {
}

// This can be refactored as below to increase readability

public boolean setCustomerAddress(Address address) {
}
```

### 5.7.硬编码

在代码中硬编码值通常会导致多种副作用。例如，**它会导致重复，这使得改变更加困难**。如果值需要是动态的，这通常会导致不良行为。在大多数情况下，硬编码值可以通过以下方式之一进行重构:

*   考虑用 Java 中定义的常量或枚举替换
*   否则，用在类级别或在单独的类文件中定义的常数替换
*   如果可能，替换为可以从配置或环境中选取的值

让我们看一个例子:

```java
private int storeClosureDay = 7;

// This can be refactored to use a constant from Java

private int storeClosureDay = DayOfWeek.SUNDAY.getValue()
```

同样，这方面也没有严格的指导方针可以遵循。但是我们必须认识到这样一个事实，有些人以后需要阅读和维护这些代码。我们应该选择一个适合我们的约定，并保持一致。

### 5.8.代码注释

[代码注释](/web/20220625172621/https://www.baeldung.com/cs/clean-code-comments)可以**有益于阅读代码，了解非琐碎方面**。同时，注意**不要在评论**中包含明显的东西。这可能会使注释膨胀，使阅读相关部分变得困难。

Java 允许两种类型的注释:实现注释和文档注释。它们有不同的目的和不同的格式。让我们更好地理解它们:

*   文档/JavaDoc 注释
    *   这里的观众是代码库的用户
    *   这里的细节通常与实现无关，更侧重于规范
    *   通常独立于代码库使用
*   实施/阻止注释
    *   这里的观众是开发代码库的开发人员
    *   这里的细节是特定于实现的
    *   通常与代码库一起使用

那么，我们应该如何优化使用它们，使它们变得有用和有意义呢？

*   注释应该只是对代码的补充，如果没有注释我们无法理解代码，也许我们需要重构它
*   我们应该很少使用块注释，可能是为了描述非平凡的设计决策
*   我们应该对大多数类、接口、公共和受保护的方法使用 JavaDoc 注释
*   为了可读性，所有的注释都应该是格式良好的，并带有适当的缩进

让我们看一个有意义的文档注释的例子:

```java
/**
* This method is intended to add a new address for the customer.
* However do note that it only allows a single address per zip
* code. Hence, this will override any previous address with the
* same postal code.
*
* @param address an address to be added for an existing customer
*/
/*
* This method makes use of the custom implementation of equals 
* method to avoid duplication of an address with same zip code.
*/
public addCustomerAddress(Address address) {
}
```

### 5.9.记录

任何曾经接触过产品代码进行调试的人都渴望在某个时间点获得更多的日志。日志的**重要性在一般的开发和特别的维护中不能被过分强调**。

Java 中有很多用于日志记录的库和框架，包括 SLF4J、Logback。虽然它们使代码库中的日志记录变得非常简单，但是必须注意日志记录的最佳实践。否则完成的日志记录可能会成为维护的噩梦，而不是任何帮助。让我们来看一下这些最佳实践:

*   避免过多的日志记录，考虑哪些信息可能有助于故障排除
*   明智地选择日志级别，我们可能希望在生产中有选择地启用日志级别
*   日志消息中的上下文数据要非常清晰和具有描述性
*   使用外部工具跟踪、聚合和过滤日志消息，以加快分析速度

让我们看一个具有正确级别的描述性日志记录的示例:

```java
logger.info(String.format("A new customer has been created with customer Id: %s", id));
```

## 6.就这些吗？

虽然上一节强调了几种代码格式约定，但这些并不是我们应该知道和关心的唯一约定。随着时间的推移，大量额外的最佳实践会使可读和可维护的代码受益。

随着时间的推移，我们可能会遇到一些有趣的缩写。他们**本质上把学到的东西作为一个或一组原则，可以帮助我们写出更好的代码**。但是，请注意，我们不应该仅仅因为它们的存在就全部遵循它们。大多数时候，它们提供的好处与代码库的大小和复杂性成正比。在采用任何原则之前，我们必须访问我们的代码库。更重要的是，我们必须与他们保持一致。

### 6.1.固体

SOLID 是一个助记首字母缩写词，它从[中提取了五个原则，这五个原则是为了编写可理解和可维护的软件而提出的](/web/20220625172621/https://www.baeldung.com/solid-principles):

*   `Single Responsibility Principle`:我们定义的每一个**接口、类或方法都应该有一个明确定义的目标**。本质上，它应该理想地做一件事，并且做得很好。这有效地导致了同样可测试的更小的方法和类。
*   `Open-Closed Principle`:我们写的代码应该是理想的**对扩展开放，对修改关闭**。这实际上意味着一个类应该以一种不需要修改的方式来编写。然而，它应该允许通过继承或组合进行更改。
*   [`Liskov Substitution Principle`](/web/20220625172621/https://www.baeldung.com/cs/liskov-substitution-principle) :这个原则陈述的是**每一个子类或派生类都应该可以替换它们的父类或基类**。这有助于减少代码库中的耦合，从而提高跨。
*   实现一个接口是为我们的类提供特定行为的一种方式。然而，**一个类不需要实现不需要**的方法。这要求我们定义更小、更集中的接口。
*   根据这个原则，**类应该只依赖于抽象，而不是它们的具体实现**。这实际上意味着一个类不应该负责为它们的依赖关系创建实例。相反，这种依赖关系应该注入到类中。

### 6.2.干&吻

DRY 代表“不要重复自己”。这个原则规定**一段代码不应该在软件**中重复。这一原则背后的基本原理是减少重复和增加可重用性。但是，请注意，我们应该小心，不要过于字面地理解它。一些重复实际上可以提高代码的可读性和可维护性。

KISS 代表“保持简单，笨蛋”。这个原则表明**我们应该尽量保持代码简单**。随着时间的推移，这使得理解和维护变得容易。遵循前面提到的一些原则，如果我们保持我们的类和方法集中和小，这将导致更简单的代码。

### 6.3.TimeDivisionDuplex 时分双工

TDD 代表“测试驱动开发”。这是一种编程实践，它要求我们只有在自动化测试失败时才编写代码。因此，我们不得不**从自动化测试的设计开发开始**。在 Java 中，有几个框架可以编写自动化单元测试，比如 JUnit 和 TestNG。

这种做法的好处是巨大的。这导致软件总是按预期工作。因为我们总是从测试开始，所以我们以小块的方式递增地添加工作代码。此外，我们只在新的或任何旧的测试失败时添加代码。这意味着它也导致了可重用性。

## 7.帮助工具

编写干净的代码不仅仅是原则和实践的问题，而是一种个人习惯。随着我们的学习和适应，我们会成长为更好的开发人员。然而，为了在一个大团队中保持一致性，我们还必须练习一些强制措施。代码审查一直是保持一致性的好工具，并通过建设性的反馈帮助开发人员成长。

然而，我们不一定要在代码评审期间手工验证所有这些原则和最佳实践。来自 Java OffHeap 的 Freddy Guime [谈到了自动进行一些质量检查的价值，以使代码质量始终达到某个阈值。](https://web.archive.org/web/20220625172621/https://www.javaoffheap.com/2019/10/episode-47-microsoft-flexing-its-java-muscle-javafx-is-alive-and-well-and-would-you-approve-my-low-quality-pr.html)

Java 生态系统中有几个可用的工具**，它们至少从代码审查者那里拿走了一些责任。让我们看看这些工具是什么:**

*   代码格式化器:大多数流行的 Java 代码编辑器，包括 Eclipse 和 IntelliJ，都允许自动格式化代码。我们可以使用默认格式规则，自定义它们，或者用自定义格式规则替换它们。这涉及到许多结构代码约定。
*   静态分析工具:Java 有几个静态代码分析工具，包括[sonar cube](/web/20220625172621/https://www.baeldung.com/sonar-qube)、 [Checkstyle](/web/20220625172621/https://www.baeldung.com/checkstyle-java) 、 [PMD](/web/20220625172621/https://www.baeldung.com/pmd) 和 [SpotBugs](https://web.archive.org/web/20220625172621/https://spotbugs.github.io/) 。它们有一套丰富的规则，可以原样使用，也可以针对特定项目进行定制。它们能很好地检测出许多[代码气味](/web/20220625172621/https://www.baeldung.com/cs/code-smells)，比如违反命名约定和资源泄漏。

## 8.结论

在本教程中，我们已经了解了干净代码原则的重要性以及干净代码所展示的特征。我们看到了如何在实践中采用这些在 Java 中开发的原则。我们还讨论了有助于保持代码可读性和可维护性的其他最佳实践。最后，我们讨论了一些可以帮助我们完成这项工作的工具。

总而言之，重要的是要注意到所有这些原则和实践都是为了使我们的代码更干净。这是一个更主观的术语，因此，必须根据上下文来评估。

虽然有许多规则可供采用，但我们必须意识到我们的成熟度、文化和需求。我们可能需要定制，或者为此设计一套新的规则。但是，不管是什么情况，重要的是在整个组织中保持一致以获得收益。