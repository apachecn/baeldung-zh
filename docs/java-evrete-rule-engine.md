# Evrete 规则引擎简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-evrete-rule-engine>

## 1.介绍

本文提供了 Evette 的第一手概述，Evette 是一个新的开源 Java 规则引擎。

从历史上看， [Evrete](https://web.archive.org/web/20221226104059/https://www.evrete.org/ "Evrete Home") 已经被开发为**的轻量级替代** [Drools 规则引擎](/web/20221226104059/https://www.baeldung.com/java-rule-engines#drools)。它完全符合 [Java 规则引擎规范](https://web.archive.org/web/20221226104059/https://jcp.org/en/jsr/detail?id=94)，并使用经典的前向链接 RETE 算法和一些调整和特性来处理大量数据。

它需要 Java 8 及更高版本，零依赖，无缝操作 JSON 和 XML 对象，**允许功能接口作为规则的条件和动作**。

它的大多数组件都可以通过服务提供者接口进行扩展，其中一个 SPI 实现将带注释的 Java 类转化为可执行的规则集。我们今天也会尝试一下。

## 2.Maven 依赖性

在我们跳到 Java 代码之前，我们需要在项目的`pom.xml`中声明 [evrete-core](https://web.archive.org/web/20221226104059/https://search.maven.org/search?q=g:org.evrete%20a:evrete-core) Maven 依赖关系:

```java
<dependency>
    <groupId>org.evrete</groupId>
    <artifactId>evrete-core</artifactId>
    <version>2.1.04</version>
</dependency> 
```

## 3.用例场景

为了使介绍不那么抽象，让我们假设我们经营一个小企业，今天是财政年度的结束，我们想计算每个客户的总销售额。

我们的领域数据模型将包括两个简单的类— `Customer`和`Invoice`:

```java
public class Customer {
    private double total = 0.0;
    private final String name;

    public Customer(String name) {
        this.name = name;
    }

    public void addToTotal(double amount) {
        this.total += amount;
    }
    // getters and setters
}
```

```java
public class Invoice {
    private final Customer customer;
    private final double amount;

    public Invoice(Customer customer, double amount) {
        this.customer = customer;
        this.amount = amount;
    }
    // getters and setters
} 
```

另一方面，该引擎支持开箱即用的 [Java 记录](/web/20221226104059/https://www.baeldung.com/java-16-new-features#records-jep-395)，并且**允许开发人员将任意类属性声明为函数接口**。

在本简介的后面，我们将获得发票和客户的集合，逻辑表明我们需要两个规则来处理数据:

*   第一个规则清除每个客户的总销售额
*   第二个规则匹配发票和客户，并更新每个客户的总数。

同样，我们将使用 fluid rule builder 接口和带注释的 Java 类来实现这些规则。让我们从规则构建器 API 开始。

## 4.规则生成器 API

规则构建器是为规则开发领域特定语言(DSL)的核心构件。开发人员将在解析 Excel 源、纯文本或任何其他需要转换成规则的 DSL 格式时使用它们。

不过，在我们的例子中，我们主要感兴趣的是他们将规则直接嵌入开发人员代码的能力。

### 4.1.规则集声明

通过规则构建器，我们可以使用流畅的接口来声明我们的两个规则:

```java
KnowledgeService service = new KnowledgeService();
Knowledge knowledge = service
  .newKnowledge()
  .newRule("Clear total sales")
  .forEach("$c", Customer.class)
  .execute(ctx -> {
      Customer c = ctx.get("$c");
      c.setTotal(0.0);
  })
  .newRule("Compute totals")
  .forEach(
      "$c", Customer.class,
      "$i", Invoice.class
  )
  .where("$i.customer == $c")
  .execute(ctx -> {
      Customer c = ctx.get("$c");
      Invoice i = ctx.get("$i");
      c.addToTotal(i.getAmount());
  }); 
```

首先，我们创建了一个`KnowledgeService`的实例，它本质上是一个共享的执行器服务。通常，**我们应该为每个应用程序**创建一个`KnowledgeService`实例。

产生的`Knowledge`实例是我们两个规则的预编译版本。我们这样做的原因和我们编译源代码的原因一样——确保正确性和更快地启动代码。

熟悉 Drools 规则引擎的人会发现我们的规则声明在语义上等同于相同逻辑的以下`DRL`版本:

```java
rule "Clear total sales"
  when
    $c: Customer
  then
    $c.setTotal(0.0);
end

rule "Compute totals"
  when
    $c: Customer
    $i: Invoice(customer == $c)
  then
    $c.addToTotal($i.getAmount());
end
```

### 4.2.模拟测试数据

我们将在三个客户和 10 万张随机金额的发票上测试我们的规则集，并在客户中随机分配:

```java
List<Customer> customers = Arrays.asList(
  new Customer("Customer A"),
  new Customer("Customer B"),
  new Customer("Customer C")
);

Random random = new Random();
Collection<Object> sessionData = new LinkedList<>(customers);
for (int i = 0; i < 100_000; i++) {
    Customer randomCustomer = customers.get(random.nextInt(customers.size()));
    Invoice invoice = new Invoice(randomCustomer, 100 * random.nextDouble());
    sessionData.add(invoice);
} 
```

现在， *sessionData* 变量包含了一组*客户*和*发票*实例，我们将把它们插入到一个规则会话中。

### 4.3.规则执行

我们现在需要做的就是将所有 100，003 个对象(100，000 张发票和三个客户)提供给一个新的会话实例，并调用它的`fire()`方法:

```java
knowledge
  .newStatelessSession()
  .insert(sessionData)
  .fire();

for(Customer c : customers) {
    System.out.printf("%s:\t$%,.2f%n", c.getName(), c.getTotal());
}
```

最后几行将打印每个客户的最终销售量:

```java
Customer A:	$1,664,730.73
Customer B:	$1,666,508.11
Customer C:	$1,672,685.10 
```

## 5.带注释的 Java 规则

尽管我们前面的例子像预期的那样工作，但是它没有使库符合规范，规范期望规则引擎:

*   `“Promote declarative programming by externalizing business or application logic.”`
*   `“Include a documented file-format or tools to author rules, and rule execution sets external to the application.”`

简而言之，这意味着兼容的规则引擎必须能够执行在其运行时之外创作的规则。

而 [Evrete 的](https://web.archive.org/web/20221226104059/https://www.evrete.org/ "Evrete Home")带注释的 Java 规则扩展模块解决了这个需求。**该模块实际上是一个“展示”DSL，它完全依赖于库的核心 API。**

让我们看看它是如何工作的。

### 5.1.装置

带注释的 Java 规则是一个 [Evrete 的](https://web.archive.org/web/20221226104059/https://www.evrete.org/ "Evrete Home")服务提供者接口(SPI)的实现，需要一个额外的 [evrete-dsl-java](https://web.archive.org/web/20221226104059/https://search.maven.org/search?q=g:org.evrete%20a:evrete-dsl-java) Maven 依赖关系:

```java
<dependency>
    <groupId>org.evrete</groupId>
    <artifactId>evrete-dsl-java</artifactId>
    <version>2.1.04</version>
</dependency>
```

### 5.2.规则集声明

让我们使用注释创建相同的规则集。我们将选择普通的 Java 源代码，而不是类和捆绑的 jar:

```java
public class SalesRuleset {

    @Rule
    public void rule1(Customer $c) {
        $c.setTotal(0.0);
    }

    @Rule
    @Where("$i.customer == $c")
    public void rule2(Customer $c, Invoice $i) {
        $c.addToTotal($i.getAmount());
    }
}
```

这个源文件可以有任何名称，不需要遵循 Java 命名约定。引擎将动态编译源代码，我们需要确保:

*   我们的源文件包含所有必需的导入
*   第三方依赖项和域类位于引擎的类路径中

然后我们告诉引擎从外部位置读取我们的规则集定义:

```java
KnowledgeService service = new KnowledgeService();
URL rulesetUrl = new URL("ruleset.java"); // or file.toURI().toURL(), etc
Knowledge knowledge = service.newKnowledge(
  "JAVA-SOURCE",
  rulesetUrl
); 
```

仅此而已。假设我们的代码的其余部分保持不变，我们将得到相同的三个客户以及他们的随机销售量。

关于这个特殊例子的几点说明:

*   我们已经选择从普通 Java 构建规则(*“Java-SOURCE”*参数)，因此允许引擎从方法参数中推断事实名称。
*   如果我们选择了*。类别*或*。jar* 源，方法参数可能需要*@事实*注释。
*   引擎已自动按方法名称对规则进行排序。如果我们交换名称，重置规则将清除先前计算的体积。因此，我们将看到零销量。

### 5.3.它是如何工作的

每当创建一个新的会话时，引擎都会将它与一个带注释的规则类的新实例结合起来。本质上，我们可以将这些类的实例视为会话本身。

因此，如果定义了类变量，规则方法就可以访问它们。

如果我们定义了条件方法或者将新字段声明为方法，这些方法也可以访问类变量。

作为常规的 Java 类，这样的规则集可以扩展、重用和打包到库中。

### 5.4.附加功能

简单的例子非常适合介绍，但是会留下许多重要的话题。对于带注释的 Java 规则，这些规则包括:

*   作为类方法的条件
*   作为类方法的任意属性声明
*   阶段监听器、继承模型和对运行时环境的访问
*   最重要的是，全面使用类字段——从条件到动作和字段定义

## 6.结论

在本文中，我们简单测试了一个新的 Java 规则引擎。关键要点包括:

1.  其他引擎可能更擅长提供现成的 DSL 解决方案和规则库。
2.  Evrete 是为开发者构建任意 DSL 而设计的**。**
***   那些习惯于用 Java 编写规则的人可能会发现“带注释的 Java 规则”包是一个更好的选择。**

 **值得一提的是本文未涉及但在该库的 API 中提到的其他特性:

*   声明任意事实属性
*   作为 Java 谓词的条件
*   即时更改规则条件和操作
*   冲突解决技术
*   向实时会话附加新规则
*   库的扩展性接口的自定义实现

官方文件位于[https://www.evrete.org/docs/](https://web.archive.org/web/20221226104059/https://www.evrete.org/docs/)。

GitHub 上的[提供了代码样本和单元测试。](https://web.archive.org/web/20221226104059/https://github.com/eugenp/tutorials/tree/master/rule-engines-modules/evrete "GitHub link")**