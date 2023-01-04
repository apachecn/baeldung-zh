# Java @Override 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-override>

## 1.概观

在这个快速教程中，我们将看看如何使用`@Override`注释。

## 2。`@Override`注解

在子类中，我们可以用[覆盖或重载](/web/20220628061654/https://www.baeldung.com/java-classes-initialization-questions)实例方法。重写表示子类正在替换继承的行为。重载是指子类增加新的行为。

有时，当我们实际上打算覆盖时，我们会意外过载。在 Java 中很容易犯这样的错误:

```
public class Machine {
    public boolean equals(Machine obj) {
        return true;
    }

    @Test
    public void whenTwoDifferentMachines_thenReturnTrue() {
        Object first = new Machine();
        Object second = new Machine();
        assertTrue(first.equals(second));
    }
}
```

令人惊讶的是，上面的测试失败了。这是因为这个`equals` 方法重载了`Object#equals`，而不是覆盖它。

**我们可以在继承的方法上使用`@Override`注释来避免这种错误。**

在这个例子中，我们可以在`equals`方法之上添加`@Override` 注释:

```
@Override
public boolean equals(Machine obj) {
    return true;
}
```

此时，编译器会抛出一个错误，通知我们没有像我们想的那样覆盖`equals`。

然后，我们可以纠正我们的错误:

```
@Override
public boolean equals(Object obj) {
    return true;
}
```

因为很容易意外重载，**通常建议在所有继承的方法上使用`@Override`注释。**

## 3.结论

在本指南中，我们看到了@Override 注释在 Java 中是如何工作的。

示例的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220628061654/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-annotations)