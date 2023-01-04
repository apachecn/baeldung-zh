# 如何找到所有返回 Null 的 Getters

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-getters-returning-null>

## 1。概述

在这篇简短的文章中，我们将使用 Java 8 Stream API 和 [`Introspector`](https://web.archive.org/web/20220630134703/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/beans/Introspector.html) 类来调用 POJO 中的所有 getters。

我们将创建一个 getters 流，检查返回值并查看字段值是否为`null.`

## 2。设置

我们需要的唯一设置是创建一个简单的 POJO 类:

```java
public class Customer {

    private Integer id;
    private String name;
    private String emailId;
    private Long phoneNumber;

    // standard getters and setters
}
```

## 3。调用 Getter 方法

我们将使用`Introspector`来分析`Customer`类；这为发现目标类支持的属性、事件和方法提供了一种简单的方法。

我们将首先收集我们的`Customer` 类的所有`PropertyDescriptor`实例。`PropertyDescriptor`捕获 Java Bean 属性的所有信息:

```java
PropertyDescriptor[] propDescArr = Introspector
  .getBeanInfo(Customer.class, Object.class)
  .getPropertyDescriptors(); 
```

现在让我们检查所有的`PropertyDescriptor` 实例，并为每个属性调用 read 方法:

```java
return Arrays.stream(propDescArr)
  .filter(nulls(customer))
  .map(PropertyDescriptor::getName)
  .collect(Collectors.toList()); 
```

我们上面使用的`nulls` 谓词检查属性是否可读，调用 getter 并只过滤空值:

```java
private static Predicate<PropertyDescriptor> nulls(Customer customer) { 
    return = pd -> { 
        Method getterMethod = pd.getReadMethod(); 
        boolean result = false; 
        return (getterMethod != null && getterMethod.invoke(customer) == null); 
    }; 
} 
```

最后，现在让我们创建一个`Customer`的实例，将一些属性设置为 null，并测试我们的实现:

```java
@Test
public void givenCustomer_whenAFieldIsNull_thenFieldNameInResult() {
    Customer customer = new Customer(1, "John", null, null);

    List<String> result = Utils.getNullPropertiesList(customer);
    List<String> expectedFieldNames = Arrays
      .asList("emailId","phoneNumber");

    assertTrue(result.size() == expectedFieldNames.size());
    assertTrue(result.containsAll(expectedFieldNames));      
}
```

## 4。结论

在这个简短的教程中，我们很好地利用了 Java 8 Stream API 和一个`Introspector` 实例——来调用所有的 getters 并检索空属性列表`.`

像往常一样，代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220630134703/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams)