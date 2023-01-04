# Java 内部接口指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-inner-interfaces>

## 1.介绍

在这个简短的教程中，我们将会看到 Java 的内部接口。它们主要用于:

*   当接口有一个公共名称时，解决命名空间问题
*   增加封装
*   通过将相关界面集中在一个地方来提高可读性

一个众所周知的例子是在`Map`接口中声明的`Entry`接口。这样定义，接口不在全局范围内，引用为`Map.Entry` ，以区别于其他`Entry` 接口，并使其与`Map` 的关系显而易见。

## 2.内部接口

根据定义，内部接口的声明出现在另一个接口或类的主体中。

当在另一个接口中声明时(类似于顶级接口中的字段声明)，它们以及它们的字段都是隐式公共和静态的，并且它们可以在任何地方实现:

```java
public interface Customer {
    // ...
    interface List {
        // ...
    }
}
```

在另一个类中声明的内部接口也是静态的，但是它们可以有访问说明符来约束它们在哪里实现:

```java
public class Customer {
    public interface List {
        void add(Customer customer);
        String getCustomerNames();
    }
    // ...
}
```

在上面的例子中，我们有一个`List` 接口，它将在`Customers` 列表上声明一些操作，比如添加新的，获得一个`String`表示等等。

`List` 是一个流行的名字，为了与定义这个接口的其他库一起工作，我们需要分离我们的声明，即`namespace` it。

如果我们不想使用像`CustomerList.`这样的新名字，这就是我们使用内部接口的地方

我们还将两个相关的接口放在一起，这提高了封装性。

最后，我们可以继续实现它:

```java
public class CommaSeparatedCustomers implements Customer.List {
    // ...
}
```

## 3.结论

我们快速浏览了一下 Java 的内部接口。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220523153642/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-inheritance)