# Java 中的规则引擎列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-rule-engines>

## 1。概述

在本文中，我们将介绍一些最流行的 Java 规则引擎。

在任务关键型应用程序中，在源代码中维护业务逻辑的过程可能会变得过于复杂。通过将业务逻辑从源代码中分离出来，可以使用业务规则来简化开发和维护。

在 Java 世界中，大多数规则引擎库都实现了 JSR94 标准，称为 [Java 规则 API 引擎](https://web.archive.org/web/20220627082047/https://jcp.org/en/jsr/detail?id=94)。

## 2。流口水

Drools 是一个业务规则管理系统(BRMS)解决方案。Drools 可以与 jBPM 集成，jBPM 是一个业务流程管理工具，用于流程、事件、活动、任务等的标准化。

如果你想了解更多，Drools 的介绍可以在这里[找到](/web/20220627082047/https://www.baeldung.com/drools)，还有一篇关于[与 Spring](/web/20220627082047/https://www.baeldung.com/drools-spring-integration) 集成的文章。

## 3。OpenL 平板电脑

[OpenL Tablets](https://web.archive.org/web/20220627082047/http://openl-tablets.org/) 是基于 Excel 决策表的业务规则管理系统和业务规则引擎。由于这个框架使用的表格格式是业务用户所熟悉的，它在业务用户和开发人员之间架起了一座桥梁。

下面是一个简单的例子，通过使用一个包含决策表的 Excel 文件来说明这个框架是如何工作的。首先，让我们导入依赖于 [org.openl.core](https://web.archive.org/web/20220627082047/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.openl%22%20AND%20a%3A%22org.openl.core%22) 和 [org.openl.rules](https://web.archive.org/web/20220627082047/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.openl.rules%22%20AND%20a%3A%22org.openl.rules%22) 模块的依赖关系:

```java
<dependency>
    <groupId>org.openl</groupId>
    <artifactId>org.openl.core</artifactId>
    <version>5.19.4</version>
</dependency>
<dependency>
    <groupId>org.openl.rules</groupId>
    <artifactId>org.openl.rules</artifactId>
    <version>5.19.4</version>
</dependency>
```

现在，一个`User` POJO:

```java
public class User {
    private String name;
    // getters and setters
}
```

以及将表示应用规则的结果的枚举:

```java
public enum Greeting {
    // ...
}
```

`Case`类用导致结果的变量包装`User`对象:

```java
public class Case {
    // Variables to infer outcomes
    // getters and setters
}
```

界面`IRule`包含 Excel 文件注入的规则:

```java
public interface IRule {
    void helloUser(Case aCase, final Response response);
}
```

`Response`类处理应用规则的返回:

```java
public class Response {
    private String result;
    private Map<String, String> map = new HashMap<>();
}
```

触发规则执行的主类:

```java
public class Main {
    private IRule instance;

    public static void main(String[] args) {
        Main rules = new Main();
        // setup user and case here
        rules.process(aCase);
    }

    public void process(Case aCase) {
        EngineFactory<IRule> engineFactory = new RulesEngineFactory<IRule>(
          getClass().getClassLoader()
            .getResource("openltablets/HelloUser.xls"), IRule.class);
        instance = engineFactory.newEngineInstance();
        instance.helloUser(aCase, new Response());
    }
}
```

## 4。简单规则

Easy Rules 是一个简单的 Java 规则引擎，提供了一个轻量级的基于 POJO 的框架来定义业务。通过使用复合模式，它可以从原始规则创建复杂的规则。

与最传统的规则引擎相比，这个框架不使用 XML 文件或任何特定于领域的语言文件来将规则与应用程序分离。它使用基于注释的类和方法将业务逻辑注入到应用程序中。

简单的规则对于开发人员创建和维护业务逻辑完全独立于应用程序本身的应用程序来说非常方便。另一方面，由于**这个框架没有实现 JSR94 标准**，业务逻辑必须直接编码成 Java 代码。

这里我们提供一个“Hello，world”的例子。让我们根据 [easy-rules-core](https://web.archive.org/web/20220627082047/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.jeasy%22%20AND%20a%3A%22easy-rules-core%22) 模块导入所需的依赖项:

```java
<dependency>
    <groupId>org.jeasy</groupId>
    <artifactId>easy-rules-core</artifactId>
    <version>3.0.0</version>
</dependency>
```

接下来，我们创建一个定义规则的类:

```java
@Rule(name = "Hello World rule", description = "Always say hello world")
public class HelloWorldRule {

    @Condition
    public boolean when() {
        return true;
    }

    @Action
    public void then() throws Exception {
        System.out.println("hello world");
    }
}
```

最后，我们创建主类:

```java
public class Launcher {
    public static void main(String... args) {
        // create facts
        Facts facts = new Facts();

        // create rules
        Rules rules = new Rules();
        rules.register(new HelloWorldRule());

        // create a rules engine and fire rules on known facts
        RulesEngine rulesEngine = new DefaultRulesEngine();
        rulesEngine.fire(rules, facts);
    }
}
```

## 5。规则手册

RuleBook 是一个 Java 框架，它利用 Java 8 lambdas 和责任链模式，使用简单的 BDD 方法来定义规则。

像大多数规则引擎一样，RuleBook 使用了“`Facts`”的概念，这是提供给规则的数据。RuleBook 允许规则修改事实的状态，然后可以被链中更下游的规则读取和修改。对于那些读入一种类型的数据(`Facts`)并输出不同类型的结果的规则，RuleBook 有`Decisions`。

RuleBook 可以使用 Java DSL 与 Spring 集成。

这里，我们使用 RuleBook 提供了一个简单的“Hello，world”示例。让我们添加依赖于 [rulebook-core](https://web.archive.org/web/20220627082047/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.deliveredtechnologies%22%20AND%20a%3A%22rulebook-core%22) 模块的依赖关系:

```java
<dependency>
    <groupId>com.deliveredtechnologies</groupId>
    <artifactId>rulebook-core</artifactId>
    <version>0.6.2</version>
</dependency>
```

现在，我们创建规则:

```java
public class HelloWorldRule {
    public RuleBook<Object> defineHelloWorldRules() {
        return RuleBookBuilder
          .create()
            .addRule(rule -> rule.withNoSpecifiedFactType()
              .then(f -> System.out.print("Hello ")))
            .addRule(rule -> rule.withNoSpecifiedFactType()
              .then(f -> System.out.println("World")))
            .build();
    }
} 
```

最后，主类:

```java
public static void main(String[] args) {
    HelloWorldRule ruleBook = new HelloWorldRule();
    ruleBook
      .defineHelloWorldRules()
      .run(new FactMap<>());
} 
```

## 6。结论

在这篇简短的文章中，我们讨论了一些为业务逻辑抽象提供引擎的著名库。

和往常一样，本文中的例子可以在我们的 [GitHub 资源库](https://web.archive.org/web/20220627082047/https://github.com/eugenp/tutorials/tree/master/rule-engines)中找到。