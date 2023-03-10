# Drools 使用 Excel 文件中的规则

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/drools-excel>

## 1。概述

[Drools](https://web.archive.org/web/20220627073939/https://www.drools.org/) 支持以电子表格格式管理业务规则。

在本文中，我们将看到一个使用 Drools 通过 Excel 文件管理业务规则的快速示例。

## 2。Maven 依赖关系

让我们将所需的 Drools 依赖项添加到我们的应用程序中:

```java
<dependency>
    <groupId>org.kie</groupId>
    <artifactId>kie-ci</artifactId>
    <version>7.1.0.Beta2</version>
</dependency>
<dependency>
    <groupId>org.drools</groupId>
    <artifactId>drools-decisiontables</artifactId>
    <version>7.1.0.Beta2</version>
</dependency>
```

这些依赖关系的最新版本可以在 [kie-ci](https://web.archive.org/web/20220627073939/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.kie%22%20AND%20a%3A%22kie-ci%22) 和 [drools-decisiontables](https://web.archive.org/web/20220627073939/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.drools%22%20AND%20a%3A%22drools-decisiontables%22) 找到。

## 3。在 Excel 中定义规则

对于我们的示例，让我们根据客户类型和作为客户的年数来定义确定折扣的规则:

*   超过 3 年的个人客户享受 15%的折扣
*   不满 3 年的个人客户享受 5%的折扣
*   所有商业客户都享受 20%的折扣

### 3.1。Excel 文件

首先根据 Drools 要求的特定结构和关键字创建我们的 [excel 文件](https://web.archive.org/web/20220627073939/https://github.com/eugenp/tutorials/blob/master/drools/src/main/resources/com/baeldung/drools/rules/Discount.xls):

[![Drools Excel](img/69d30dc9b4e4b81ae3cd8a726264dfad.png)](/web/20220627073939/https://www.baeldung.com/wp-content/uploads/2017/06/Drools_Excel.png)

对于我们的简单示例，我们使用了最相关的一组关键字:

*   `RuleSet`–表示决策表的开始
*   `Import`–规则中使用的 Java 类
*   `RuleTable`–表示规则集的开始
*   `Name`–规则的名称
*   `CONDITION`–根据输入数据检查条件的代码片段。一个规则应该至少包含一个条件
*   `ACTION`–满足规则条件时要采取的操作的代码片段。一个规则应该至少包含一个操作。在这个例子中，我们在`Customer`对象上调用`setDiscount`

另外，我们已经在 Excel 文件中使用了`Customer`类。那么，让我们现在就创建它。

### 3.2。`Customer` 班

从 excel 表中的条件和动作可以看出，我们使用了一个`Customer`类的对象作为输入数据(`type`和`years`)并存储结果(`discount`)。

`Customer`类:

```java
public class Customer {
    private CustomerType type;

    private int years;

    private int discount;

    // Standard getters and setters

    public enum CustomerType {
        INDIVIDUAL,
        BUSINESS;
    }
}
```

## 4。 **创建 Drools 规则引擎实例**

在执行我们已经定义的规则之前，我们必须使用 Drools 规则引擎的一个实例。为此，我们必须使用 Kie 核心组件。

### 4.1。`KieServices`

`KieServices`类提供了对 所有 Kie 构建和运行时设施的访问。它提供了几个工厂、服务和实用方法。所以，让我们先来看看一个`KieServices`实例:

```java
KieServices kieServices = KieServices.Factory.get();
```

使用 KieServices，我们将创建新的`KieFileSystem`、`KieBuilder`和`KieContainer`实例。

### 4.2。`KieFileSystem`

`KieFileSystem`是一个虚拟文件系统。让我们将 Excel 电子表格添加到其中:

```java
Resource dt 
  = ResourceFactory
    .newClassPathResource("com/baeldung/drools/rules/Discount.xls",
      getClass());

KieFileSystem kieFileSystem = kieServices.newKieFileSystem().write(dt); 
```

### 4.3。`KieBuilder`

现在，通过将内容传递给`KieBuilder`来构建`KieFileSystem`的内容:

```java
KieBuilder kieBuilder = kieServices.newKieBuilder(kieFileSystem);
kieBuilder.buildAll();
```

如果构建成功，它会创建一个`KieModule (`任何 Maven 生成的 jar，其中包含 kmodule.xml，都是一个`KieModule`。

### 4.4。`KieRepository`

框架自动将`KieModule`(由构建产生)添加到`KieRepository`:

```java
KieRepository kieRepository = kieServices.getRepository();
```

### 4.5。`KieContainer`

现在可以使用这个`KieModule`的`ReleaseId`创建一个新的`KieContainer`。在这种情况下，Kie 会分配一个默认值`ReleaseId:`

```java
ReleaseId krDefaultReleaseId = kieRepository.getDefaultReleaseId();
KieContainer kieContainer 
  = kieServices.newKieContainer(krDefaultReleaseId);
```

### 4.6。`KieSession`

我们现在可以从`KieContainer`中获得`KieSession`。我们的应用程序与`KieSession`交互，后者存储并执行运行时数据:

```java
KieSession kieSession = kieContainer.newKieSession();
```

## 5。执行规则

最后，是时候提供输入数据并启动规则了:

```java
Customer customer = new Customer(CustomerType.BUSINESS, 2);
kieSession.insert(customer);

kieSession.fireAllRules();
```

## 6。测试用例

现在让我们添加一些测试用例:

```java
public class DiscountExcelIntegrationTest {

    private KieSession kSession;

    @Before
    public void setup() {
        Resource dt 
          = ResourceFactory
            .newClassPathResource("com/baeldung/drools/rules/Discount.xls",
              getClass());
        kSession = new DroolsBeanFactory().getKieSession(dt);
    }

    @Test
    public void 
      giveIndvidualLongStanding_whenFireRule_thenCorrectDiscount() 
        throws Exception {
        Customer customer = new Customer(CustomerType.INDIVIDUAL, 5);
        kSession.insert(customer);

        kSession.fireAllRules();

        assertEquals(customer.getDiscount(), 15);
    }

    @Test
    public void 
      giveIndvidualRecent_whenFireRule_thenCorrectDiscount() 
      throws Exception {
        Customer customer = new Customer(CustomerType.INDIVIDUAL, 1);
        kSession.insert(customer);

        kSession.fireAllRules();

        assertEquals(customer.getDiscount(), 5);
    }

    @Test
    public void 
      giveBusinessAny_whenFireRule_thenCorrectDiscount() 
        throws Exception {
        Customer customer = new Customer(CustomerType.BUSINESS, 0);
        kSession.insert(customer);

        kSession.fireAllRules();

        assertEquals(customer.getDiscount(), 20);
    }
}
```

## 7。故障排除

Drools 将决策表转换为 [DRL](/web/20220627073939/https://www.baeldung.com/drools) 。因此，处理 Excel 文件中的错误和打字错误可能会很困难。这些错误通常与 DRL 的内容有关。因此，打印和分析 DRL 有助于故障排除:

```java
Resource dt 
  = ResourceFactory
    .newClassPathResource("com/baeldung/drools/rules/Discount.xls",
      getClass());

DecisionTableProviderImpl decisionTableProvider 
  = new DecisionTableProviderImpl();

String drl = decisionTableProvider.loadFromResource(dt, null);
```

## 8。结论

在本文中，我们看到了一个使用 Drools 在 Excel 电子表格中管理业务规则的快速示例。我们已经看到了在 Excel 文件中定义规则时使用的结构和最小关键字集。接下来，我们使用 Kie 组件来读取和触发规则。最后，我们编写测试用例来验证结果。

和往常一样，本文中使用的例子可以在 Github 项目中找到。