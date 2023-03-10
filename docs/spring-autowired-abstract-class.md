# 在抽象类中使用@Autowired

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-autowired-abstract-class>

## 1.介绍

在这个快速教程中，我们将解释如何在[抽象类](/web/20221121145441/https://www.baeldung.com/java-abstract-class)中使用 [`@Autowired`](/web/20221121145441/https://www.baeldung.com/spring-autowire) 注释。

我们将把`@Autowired`应用到一个抽象类中，并关注我们应该考虑的要点。

## 2.定型剂注射

我们可以在 setter 方法上使用`@Autowired`:

```java
public abstract class BallService {

    private LogRepository logRepository;

    @Autowired
    public final void setLogRepository(LogRepository logRepository) {
        this.logRepository = logRepository;
    }
}
```

当我们在 setter 方法上使用`@Autowired`时，我们应该使用**[`final`关键字](/web/20221121145441/https://www.baeldung.com/java-final)，这样子类就不能覆盖 setter 方法**。否则，注释不会像我们预期的那样工作。

## 3.构造函数注入

**我们不能在抽象类的构造函数上使用`@Autowired`。**

Spring 不评估抽象类的构造函数上的`@Autowired`注释。子类应该为`super`构造函数提供必要的参数。

相反，我们应该在子类的构造函数上使用`@Autowired`:

```java
public abstract class BallService {

    private RuleRepository ruleRepository;

    public BallService(RuleRepository ruleRepository) {
        this.ruleRepository = ruleRepository;
    }
}
```

```java
@Component
public class BasketballService extends BallService {

    @Autowired
    public BasketballService(RuleRepository ruleRepository) {
        super(ruleRepository);
    }
}
```

## 4.小抄

让我们总结一些需要记住的规则。

首先，**抽象类不会被组件扫描**,因为没有具体的子类它就不能被实例化。

第二， **setter 注入在抽象类**中是可能的，但是如果我们不对 setter 方法使用`final`关键字，这是有风险的。如果子类覆盖了 setter 方法，应用程序可能会不稳定。

第三，由于 Spring 不支持抽象类中的构造函数注入，**我们通常应该让具体的子类提供构造函数参数**。这意味着**我们需要依靠构造函数注入具体的子类**。

最后，对必需的依赖项使用构造函数注入，对可选的依赖项使用 setter 注入[是一个很好的经验法则](https://web.archive.org/web/20221121145441/https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/core.html#beans-factory-collaborators)。然而，正如我们可以看到的抽象类的一些细微差别，**构造函数注入在这里更有利**。

所以，实际上我们可以说**一个具体的子类决定了它的抽象父类如何获得它的依赖**。只要 Spring 连接了子类，Spring 就会执行注入**。**

## 5.结论

在本文中，我们练习了在抽象类中使用`@Autowired`,并解释了一些重要的要点。

本教程的源代码照常可以在[Github 项目](https://web.archive.org/web/20221121145441/https://github.com/eugenp/tutorials/tree/master/spring-core-4)中找到。