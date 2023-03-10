# Drools 中反向链接的一个例子

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/drools-backward-chaining>

## 1。概述

在本文中，我们将了解什么是反向链接，以及如何在 Drools 中使用它。

本文是展示 [Drools 业务规则引擎](/web/20220523230533/https://www.baeldung.com/drools)系列文章的一部分。

## 2。Maven 依赖关系

让我们从导入 drools-core [依赖关系](https://web.archive.org/web/20220523230533/https://search.maven.org/classic/#search%7Cga%7C1%7Cdrools-core)开始:

```java
<dependency>
    <groupId>org.drools</groupId>
    <artifactId>drools-core</artifactId>
    <version>7.4.1.Final</version>
</dependency>
```

## 3。正向链接

首先，使用正向链接，我们从分析数据开始，并朝着特定的结论前进。

应用前向链接的一个例子是通过检查节点之间已知的连接来发现新路由的系统。

## 4。反向链接

与正向链接相反，**反向链接直接从结论(假设)开始，并通过回溯一系列事实来验证它。**

比较正向链接和反向链接时，**前者可以描述为“数据驱动”(数据作为输入)，后者可以描述为“事件(或目标)驱动”(目标作为输入)。**

应用反向链接的一个例子是验证是否有连接两个节点的路由。

## 5。Drools 反向链接

Drools 项目最初是作为一个正向链接系统创建的。但是，从版本 5.2.0 开始，它也支持向后链接。

让我们创建一个简单的应用程序，并尝试验证一个简单的假设—`if the Great Wall of China is on Planet Earth`。

### 5.1。数据

让我们创建一个简单的事实库来描述事物及其位置:

1.  `Planet Earth`
2.  `Asia, Planet Earth`
3.  `China, Asia`
4.  `Great Wall of China, China`

### 5.2。定义规则

现在，让我们创建一个”。名为`BackwardChaining.drl`的 drl”文件，我们将把它放在`/resources/com/baeldung/drools/rules/`中。这将包含示例中使用的所有必要的查询和规则。

利用反向链接的主`belongsTo`查询可以写成:

```java
query belongsTo(String x, String y)
    Fact(x, y;)
    or
    (Fact(z, y;) and belongsTo(x, z;))
end
```

此外，让我们添加两条规则，以便轻松查看我们的结果:

```java
rule "Great Wall of China BELONGS TO Planet Earth"
when
    belongsTo("Great Wall of China", "Planet Earth";)
then
    result.setValue("Decision one taken: Great Wall of China BELONGS TO Planet Earth");
end

rule "print all facts"
when
    belongsTo(element, place;)
then
    result.addFact(element + " IS ELEMENT OF " + place);
end
```

### 5.3。创建应用程序

现在，我们需要一个 Java 类来表示事实:

```java
public class Fact {

    @Position(0)
    private String element;

    @Position(1)
    private String place;

    // getters, setters, constructors, and other methods ...    
} 
```

这里我们使用`@Position`注释告诉应用程序 Drools 将按照什么顺序为这些属性提供值。

此外，我们将创建表示结果的 POJO:

```java
public class Result {
    private String value;
    private List<String> facts = new ArrayList<>();

    //... getters, setters, constructors, and other methods
}
```

现在，我们可以运行示例:

```java
public class BackwardChainingTest {

    @Before
    public void before() {
        result = new Result();
        ksession = new DroolsBeanFactory().getKieSession();
    }

    @Test
    public void whenWallOfChinaIsGiven_ThenItBelongsToPlanetEarth() {

        ksession.setGlobal("result", result);
        ksession.insert(new Fact("Asia", "Planet Earth"));
        ksession.insert(new Fact("China", "Asia"));
        ksession.insert(new Fact("Great Wall of China", "China"));

        ksession.fireAllRules();

        assertEquals(
          result.getValue(),
          "Decision one taken: Great Wall of China BELONGS TO Planet Earth");
    }
}
```

当测试用例被执行时，它们添加给定的事实(“`Asia belongs to Planet Earth`”、“中国`belongs to` 亚洲”、“中国的长城属于中国”)。

之后，用`BackwardChaining.drl`中描述的规则处理事实，这提供了一个递归查询`belongsTo(String x, String y).`

该查询由使用反向链接的规则调用，以发现假设(`“Great Wall of China BELONGS TO Planet Earth”`)是真还是假。

## 6。结论

我们已经展示了反向链接的概述，这是 Drools 的一个特性，用于检索一系列事实来验证决策是否正确。

和往常一样，完整的例子可以在我们的 [GitHub 库](https://web.archive.org/web/20220523230533/https://github.com/eugenp/tutorials/tree/master/drools)中找到。