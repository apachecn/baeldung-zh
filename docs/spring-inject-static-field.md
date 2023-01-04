# 在春天的静态场中注入一个值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-inject-static-field>

## 1.概观

在本教程中，我们将看到如何用 Spring 将一个值从一个 Java 属性文件注入一个静态字段。

## 2.问题

首先，让我们假设我们为属性文件设置了一个属性:

```java
name = Inject a value to a static field
```

之后，我们希望将其值注入到一个实例变量中。

这个**通常可以通过在实例字段上使用 [@Value](/web/20221207175050/https://www.baeldung.com/spring-value-annotation) 注释**来完成:

```java
@Value("${name}")
private String name;
```

然而，当我们试图将**应用于静态场时，我们会发现它仍然是`null` :**

```java
@Value("${name}")
private static String NAME_NULL;
```

那是因为 **Spring 不支持静态字段**上的`@Value`。

现在，老实说，这是我们代码所处的一个奇怪的位置，我们应该首先考虑重构。但是，让我们来看看我们如何使这个工作。

## 3.解决办法

首先，让我们声明我们想要注入的静态变量`NAME_STATIC`。

之后，我们将创建一个名为`setNameStatic`的 setter 方法，并用 [`@Value`](/web/20221207175050/https://www.baeldung.com/spring-value-annotation) 注释对其进行注释:

```java
@RestController
public class PropertyController {

    @Value("${name}")
    private String name;

    private static String NAME_STATIC;

    @Value("${name}")
    public void setNameStatic(String name){
        PropertyController.NAME_STATIC = name;
    }
}
```

让我们试着理解上面发生的事情。

首先，`PropertyController`，也就是一个`RestController`，正在被 Spring 初始化。

之后，Spring 搜索`Value`带注释的字段和方法。

Spring 使用[依赖注入](/web/20221207175050/https://www.baeldung.com/spring-dependency-injection)来填充找到`@Value` 注释时的特定值。**然而，不是将值传递给实例变量，而是传递给隐式 setter。**这个 setter 然后处理我们的`NAME_STATIC `值的填充。

## 4.结论

在这个简短的教程中，我们已经了解了如何将属性文件中的值注入到静态变量中。当我们的重构尝试失败时，这是一条我们可以考虑的路线。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221207175050/https://github.com/eugenp/tutorials/tree/master/spring-di-2)