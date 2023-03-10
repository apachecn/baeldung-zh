# 合并 java.util.Properties 对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-merging-properties>

## 1。简介

在这个简短的教程中，**我们将关注如何将两个或更多的 Java `Properties`对象合并成一个。**

我们将探索三种解决方案，首先从一个使用迭代的例子开始。接下来，我们将研究如何使用`putAll()`方法，并以此结束本教程，我们将研究一种使用 Java 8 流的更现代的方法。

要了解如何开始使用 Java 属性，请查看我们的[介绍文章](/web/20221206080936/https://www.baeldung.com/java-properties)。

## 2。使用属性的快速回顾

让我们从提醒自己一些关键的属性概念开始。

**我们通常在应用程序中使用属性来定义配置值**。在 Java 中，我们使用简单的键/值对来表示这些值。此外，键和值都是这些对中的`String`值。

通常我们使用`java.util.Properties`类来表示和管理这些值对。**需要注意的是，这个类继承自`Hashtable`。**

要了解更多关于`Hashtable`数据结构的信息，请阅读我们的[Java . util . hashtable](/web/20221206080936/https://www.baeldung.com/java-hash-table)简介。

### 2.1。设置属性

为了简单起见，我们将以编程方式为我们的示例设置属性:

```java
private Properties propertiesA() {
    Properties properties = new Properties();
    properties.setProperty("application.name", "my-app");
    properties.setProperty("application.version", "1.0");
    return properties;
}
```

在上面的例子中，我们创建了一个`Properties`对象，并使用`setProperty()`方法来设置两个属性。在内部，这从`Hashtable`类调用了`put()` 方法，但确保对象是`String`值。

**注意，强烈建议不要直接使用`put()`方法，因为它允许调用者插入键或值不是`Strings`的条目。**

## 3。使用迭代合并属性

现在让我们看看如何使用迭代合并两个或更多的 properties 对象:

```java
private Properties mergePropertiesByIteratingKeySet(Properties... properties) {
    Properties mergedProperties = new Properties();
    for (Properties property : properties) {
        Set<String> propertyNames = property.stringPropertyNames();
        for (String name : propertyNames) {
            String propertyValue = property.getProperty(name);
            mergedProperties.setProperty(name, propertyValue);
        }
    }
    return mergedProperties;
} 
```

让我们把这个例子分成几个步骤:

1.  首先，我们创建一个`Properties`对象来保存所有合并的属性
2.  接下来，我们循环遍历将要合并的`Properties`对象
3.  然后我们调用`stringPropertyNames() `方法来获得一组属性名
4.  然后我们遍历所有的属性名，并获取每个名称的属性值
5.  最后，我们将属性值设置到我们在步骤 1 中创建的变量中

## 4。使用`putAll()`方法

现在我们来看看另一个使用`putAll()`方法合并属性的常见解决方案:

```java
private Properties mergePropertiesByUsingPutAll(Properties... properties) {
    Properties mergedProperties = new Properties();
    for (Properties property : properties) {
        mergedProperties.putAll(property);
    }
    return mergedProperties;
} 
```

在我们的第二个例子中，我们再次首先创建一个`Properties`对象来保存我们所有的合并属性，称为`mergedProperties`。同样，我们然后遍历将要合并的`Properties`对象，但是这次我们使用`putAll()`方法将每个`Properties`对象添加到`mergedProperties `变量中。

`putAll()` 方法是继承自`Hashtable`的另一种方法。**这个方法允许我们将指定的`Properties`中的所有映射复制到新的`Properties`对象中。**

值得一提的是，不鼓励将`putAll() `与任何类型的`Map `一起使用，因为我们最终可能会得到键或值不是`Strings`的条目

## 5。用流 API 合并属性

最后，我们将看看如何使用流 API 来合并多个`Properties`对象:

```java
private Properties mergePropertiesByUsingStreamApi(Properties... properties) {
    return Stream.of(properties)
        .collect(Properties::new, Map::putAll, Map::putAll);
} 
```

在最后一个例子中，我们从属性列表中创建了一个`Stream `，然后使用`collect `方法将流中的值序列减少到一个新的`Collection`中。**第一个参数是一个`Supplier` 函数，用于创建一个新的结果容器，在我们的例子中是一个新的`Properties`对象。**

流 API 是在 Java 8 中引入的，我们有一个关于[开始使用这个 API 的指南。](/web/20221206080936/https://www.baeldung.com/java-8-streams-introduction)

## 6。结论

在这个简短的教程中，我们介绍了三种不同的方法来合并两个或更多的`Properties`对象。

和往常一样，这些例子可以在我们的 [GitHub 库](https://web.archive.org/web/20221206080936/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java)中找到。