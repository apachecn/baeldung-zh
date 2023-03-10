# 杰克逊@JsonFormat 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-jsonformat>

## 1。概述

在本教程中，我们将演示如何在 Jackson 中使用`@JsonFormat`。

`@JsonFormat`是一个 Jackson 注释，我们用它来指定如何格式化 JSON 输出的字段和/或属性。

具体来说，这个注释允许我们指定如何根据一个`SimpleDateFormat`格式来格式化`Date`和`Calendar`值。

## 2。Maven 依赖关系

`@JsonFormat`是在 [jackson-databind](https://web.archive.org/web/20220626202434/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22) 包中定义的，所以我们需要下面的 Maven 依赖关系:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

## 3。入门

### 3.1。使用默认格式

我们将从演示在代表用户的类中使用`@JsonFormat`注释的概念开始。

因为我们想解释注释的细节，所以将根据请求创建`User`对象(而不是从数据库中存储或加载)并序列化为 JSON:

```java
public class User {
    private String firstName;
    private String lastName;
    private Date createdDate = new Date();

    // standard constructor, setters and getters
} 
```

构建和运行此代码示例将返回以下输出:

```java
{"firstName":"John","lastName":"Smith","createdDate":1482047026009}
```

正如我们所见，`createdDate`字段显示为从 epoch 开始的秒数，这是用于`Date`字段的默认格式。

### 3.2。在 Getter 上使用注释

我们现在将使用`@JsonFormat`来指定序列化`createdDate`字段的格式。

让我们来看看针对这一变化而更新的用户类。我们注释了如图所示的`createdDate`字段，以指定日期格式。

用于`pattern`参数的数据格式由`[SimpleDateFormat](https://web.archive.org/web/20220626202434/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/SimpleDateFormat.html)`指定:

```java
@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "[[email protected]](/web/20220626202434/https://www.baeldung.com/cdn-cgi/l/email-protection):mm:ss.SSSZ")
private Date createdDate;
```

有了这个变更，我们再次构建项目并运行它。

这是输出结果:

```java
{"firstName":"John","lastName":"Smith","createdDate":"[[email protected]](/web/20220626202434/https://www.baeldung.com/cdn-cgi/l/email-protection):53:34.740+0000"}
```

这里我们已经使用指定的`SimpleDateFormat`格式和`@JsonFormat`注释格式化了`createdDate`字段。

上面的例子演示了在一个字段上使用注释。我们也可以在 getter 方法(一个属性)中使用它。

例如，我们可能有一个在调用时被计算的属性。在这种情况下，我们可以在 getter 方法上使用注释。

请注意，我们还更改了模式，只返回即时消息的日期部分:

```java
@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd")
public Date getCurrentDate() {
    return new Date();
}
```

这是输出结果:

```java
{ ... , "currentDate":"2016-12-18", ...}
```

### 3.3。指定区域设置

除了指定日期格式，我们还可以指定序列化的区域设置。

不指定此参数会导致使用默认区域设置执行序列化:

```java
@JsonFormat(
  shape = JsonFormat.Shape.STRING, pattern = "[[email protected]](/web/20220626202434/https://www.baeldung.com/cdn-cgi/l/email-protection):mm:ss.SSSZ", locale = "en_GB")
public Date getCurrentDate() {
    return new Date();
}
```

### 3.4。指定形状

在`shape`设置为`JsonFormat.Shape.NUMBER`的情况下使用`@JsonFormat`会导致`Date` 类型的默认输出——自`.`纪元以来的秒数

参数`pattern`不适用于这种情况，忽略不计:

```java
@JsonFormat(shape = JsonFormat.Shape.NUMBER)
public Date getDateNum() {
    return new Date();
}
```

让我们看看输出:

```java
{ ..., "dateNum":1482054723876 }
```

## 4。结论

综上所述，我们用`@JsonFormat`来控制`Date`和`Calendar`类型的输出格式。

上面显示的示例代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220626202434/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-annotations)