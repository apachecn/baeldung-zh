# 什么是春豆？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-bean>

## 1。概述

Bean 是 Spring 框架的一个关键概念。所以理解这个概念对于掌握框架的窍门并有效地使用它是至关重要的。

不幸的是，对于春豆到底是什么这个简单的问题，没有明确的答案。有些解释过于浅显，以至于忽略了全局，而另一些解释则过于模糊。

本教程将试图阐明这个主题，从官方文档中的描述开始。

## 延伸阅读:

## 为什么选择 Spring 作为你的 Java 框架？

A quick and practical overview of the main value proposition of Spring framework.[Read more](/web/20220913201843/https://www.baeldung.com/spring-why-to-choose) →

## [了解春天的 getBean()](/web/20220913201843/https://www.baeldung.com/spring-getbean)

Learn about the different variants of Spring's BeanFactory.getBean() method for retrieving a bean instance from the Spring container[Read more](/web/20220913201843/https://www.baeldung.com/spring-getbean) →

## 2。Bean 定义

下面是 Spring 框架文档中对 beans 的定义:

`In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container.`

这个定义简明扼要，但是没有详细说明一个重要的元素:Spring IoC 容器。让我们仔细看看它是什么，以及它带来的好处。

## 3。控制反转

简单来说， [I](/web/20220913201843/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring) [控制转换](/web/20220913201843/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring) (IoC)就是**一个对象定义其依赖关系而不创建依赖关系的过程。这个对象将构建这种依赖关系的工作委托给一个 IoC 容器。**

在深入研究 IoC 之前，让我们从声明几个域类开始。

### 3.1。领域类别

假设我们有一个类声明:

```
public class Company {
    private Address address;

    public Company(Address address) {
        this.address = address;
    }

    // getter, setter and other properties
}
```

该类需要一个类型为`Address`的合作者:

```
public class Address {
    private String street;
    private int number;

    public Address(String street, int number) {
        this.street = street;
        this.number = number;
    }

    // getters and setters
}
```

### 3.2。传统方法

通常，我们用类的构造函数创建对象:

```
Address address = new Address("High Street", 1000);
Company company = new Company(address);
```

这种方法没有错，但是以更好的方式管理依赖关系不是很好吗？

想象一个有几十甚至几百个类的应用程序。有时我们希望在整个应用程序中共享一个类的实例，有时我们需要为每个用例创建一个单独的对象，等等。

管理如此多的对象简直是一场噩梦。这就是控制反转的作用。

对象可以从 IoC 容器中检索它的依赖关系，而不是自己构造依赖关系。我们需要做的就是为容器提供适当的配置元数据。

### 3.3。Bean 配置

首先，让我们用`@Component`注释来装饰`Company`类:

```
@Component
public class Company {
    // this body is the same as before
}
```

下面是一个为 IoC 容器提供 bean 元数据的配置类:

```
@Configuration
@ComponentScan(basePackageClasses = Company.class)
public class Config {
    @Bean
    public Address getAddress() {
        return new Address("High Street", 1000);
    }
}
```

配置类产生一个类型为`Address`的 bean。它还带有`@ComponentScan`注释，指示容器在包含`Company`类的包中寻找 beans。

当一个 Spring IoC 容器构造这些类型的对象时，所有的对象都被称为 Spring beans，因为它们是由 IoC 容器管理的。

### 3.4。国际奥委会在行动

因为我们在配置类中定义了 beans，**我们将需要一个`AnnotationConfigApplicationContext`类的实例来构建一个容器**:

```
ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
```

一个快速测试验证了 beans 的存在和属性值:

```
Company company = context.getBean("company", Company.class);
assertEquals("High Street", company.getAddress().getStreet());
assertEquals(1000, company.getAddress().getNumber());
```

结果证明 IoC 容器已经正确地创建和初始化了 beans。

## 4。结论

本文简要描述了 Spring beans 及其与 IoC 容器的关系。

完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220913201843/https://github.com/eugenp/tutorials/tree/master/spring-core)