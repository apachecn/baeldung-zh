# Java 字符串 equalsIgnoreCase()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-equalsignorecase>

## 1。概述

在这个快速教程中，我们将看看当我们忽略大小写时，两个`String`值是否相同。

## 2。使用`equalsIgnoreCase()`

`equalsIgnoreCase()`接受另一个`String`并返回一个`boolean`值:

```java
String lower = "equals ignore case";
String UPPER = "EQUALS IGNORE CASE";

assertThat(lower.equalsIgnoreCase(UPPER)).isTrue(); 
```

## 3。使用 Apache Commons Lang

[Apache Commons Lang](/web/20221205210828/https://www.baeldung.com/string-processing-commons-lang) 库包含一个名为 [`StringUtils`](/web/20221205210828/https://www.baeldung.com/string-processing-commons-lang) 的类，该类提供了一个类似于上述方法的方法，但是它具有处理`null`值的额外好处:

```java
String lower = "equals ignore case"; 
String UPPER = "EQUALS IGNORE CASE"; 

assertThat(StringUtils.equalsIgnoreCase(lower, UPPER)).isTrue();
assertThat(StringUtils.equalsIgnoreCase(lower, null)).isFalse();
```

## 4。结论

在本文中，我们快速查看了当我们忽略大小写时，确定两个`String`值是否相同。现在，当我们国际化时，事情变得有点棘手，因为区分大小写是特定于一种语言的——请继续关注更多信息。

和往常一样，所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20221205210828/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-2)