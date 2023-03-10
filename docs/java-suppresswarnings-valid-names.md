# 有效的@SuppressWarnings 警告名称

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-suppresswarnings-valid-names>

## 1.概观

在本教程中，我们将看看与 [`@SuppressWarnings`](/web/20220526053704/https://www.baeldung.com/java-suppresswarnings) Java 注释一起使用的不同警告名称，它允许我们抑制编译器警告。这些警告名称允许我们隐藏特定的警告。可用的警告名称将取决于我们的 IDE 或 Java 编译器。Eclipse IDE 是本文的参考资料。

## 2.警告名称

以下是`@SuppressWarnings`注释中可用的有效警告名称列表:

*   这是一种抑制所有警告的通配符
*   `**boxing**`:取消与装箱/拆箱操作相关的警告
*   `**unused**`:抑制未使用代码的警告
*   `**cast**`:抑制与对象转换操作相关的警告
*   `**deprecation**`:取消与弃用相关的警告，如弃用的类或方法
*   `**restriction**`:取消与使用不鼓励或禁止的引用相关的警告
*   `**dep-ann**`:取消与不推荐使用的注释相关的警告
*   `**fallthrough**`:取消与`switch`语句中缺少`break`语句相关的警告
*   `**finally**`:抑制与不返回的`finally`块相关的警告
*   `**hiding**`:抑制与隐藏变量的局部变量相关的警告
*   `**incomplete-switch**`:取消与`switch`语句中缺失条目相关的警告(`enum`案例)
*   `**nls**`:取消与非 nls 字符串文字相关的警告
*   `**null**`:抑制与`null`分析相关的警告
*   `**serial**`:取消与缺少`serialVersionUID`字段相关的警告，该字段通常出现在`Serializable`类中
*   `**static-access**`:抑制与不正确的静态变量访问相关的警告
*   `**synthetic-access**`:抑制与内部类的非优化访问相关的警告
*   `**unchecked**`:抑制与未检查操作相关的警告
*   `**unqualified-field-access**`:取消与不合格字段访问相关的警告
*   `**javadoc**`:取消与 Javadoc 相关的警告
*   `**rawtypes**` : 取消与使用原始类型相关的警告
*   `**resource**` : 抑制与`Closeable`类型的资源使用相关的警告
*   `**super**` : 取消与在没有`super`调用的情况下重写方法相关的警告
*   `**sync-override**` : 在覆盖`synchronized`方法时，因缺少`synchronize`而取消警告

## 3.使用警告名称

本节将展示不同警告名称的使用示例。

### 3.1.`@SuppressWarnings(“unused”)`

在下面的示例中，警告名称在方法中取消了对`unusedVal`的警告:

```java
@SuppressWarnings("unused")
void suppressUnusedWarning() {
    int usedVal = 5;
    int unusedVal = 10;  // no warning here
    List<Integer> list = new ArrayList<>();
    list.add(usedVal);
}
```

### 3.2.`@SuppressWarnings(“deprecated”)`

在下面的示例中，警告名称取消了对使用`@deprecated`方法的警告:

```java
@SuppressWarnings("deprecated")
void suppressDeprecatedWarning() {
    ClassWithSuppressWarningsNames cls = new ClassWithSuppressWarningsNames();
    cls.deprecatedMethod(); // no warning here
}

@Deprecated
String deprecatedMethod() {
    return "deprecated method";
}
```

### 3.3.`@SuppressWarnings(“fallthrough”)`

在下面的示例中，警告名称取消了对缺少的`break`语句的警告——我们将它们包含在这里，并将其注释掉，以显示否则我们会在哪里得到警告:

```java
@SuppressWarnings("fallthrough")
String suppressFallthroughWarning() {
    int day = 5;
    switch (day) {
        case 5:
            return "This is day 5";
//          break; // no warning here
        case 10:
            return "This is day 10";
//          break; // no warning here   
        default:
            return "This default day";
    }
}
```

### 3.4.`@SuppressWarnings(“serial”)`

该警告名称位于类级别。在下面的例子中，警告名称取消了对一个`Serializable`类中缺少`serialVersionUID`(我们已经注释掉了)的警告:

```java
@SuppressWarnings("serial")
public class ClassWithSuppressWarningsNames implements Serializable {
//    private static final long serialVersionUID = -1166032307853492833L; // no warning even though this is commented
```

## 4.组合多个警告名称

`@SuppressWarnings`注释需要一个`String`的数组，因此我们可以组合多个警告名称:

```java
@SuppressWarnings({"serial", "unchecked"})
```

## 5.结论

本文提供了有效的`@SuppressWarnings`警告名称列表。像往常一样，本教程中展示的所有代码示例都可以在 GitHub 的[上获得。](https://web.archive.org/web/20220526053704/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-annotations)