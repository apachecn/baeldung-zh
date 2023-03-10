# 带有 Lambda 表达式的 Java 流过滤器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-filter-lambda>

## 1。简介

在这个快速教程中，我们将探索在 Java 中使用 [`Streams`时`Stream.filter()`方法的用法。](/web/20220926153229/https://www.baeldung.com/java-8-streams-introduction)

我们将看看如何使用它，以及如何处理带有检查异常的特殊情况。

## 延伸阅读:

## [Java 8 流简介](/web/20220926153229/https://www.baeldung.com/java-8-streams-introduction)

A quick and practical introduction to Java 8 Streams.[Read more](/web/20220926153229/https://www.baeldung.com/java-8-streams-introduction) →

## [如何在 Java 中过滤收藏](/web/20220926153229/https://www.baeldung.com/java-collection-filtering)

A quick tutorial to filtering collections in Java using different approaches.[Read more](/web/20220926153229/https://www.baeldung.com/java-collection-filtering) →

## [Java 8 中的函数接口](/web/20220926153229/https://www.baeldung.com/java-8-functional-interfaces)

Quick and practical guide to Functional Interfaces present in Java 8.[Read more](/web/20220926153229/https://www.baeldung.com/java-8-functional-interfaces) →

## 2。使用`Stream.filter()`

`filter()`方法是`Stream`接口的中间操作，它允许我们过滤与给定的`Predicate:`匹配的流元素

```java
Stream<T> filter(Predicate<? super T> predicate)
```

为了了解这是如何工作的，让我们创建一个`Customer`类:

```java
public class Customer {
    private String name;
    private int points;
    //Constructor and standard getters
}
```

此外，让我们创建一个客户集合:

```java
Customer john = new Customer("John P.", 15);
Customer sarah = new Customer("Sarah M.", 200);
Customer charles = new Customer("Charles B.", 150);
Customer mary = new Customer("Mary T.", 1);

List<Customer> customers = Arrays.asList(john, sarah, charles, mary);
```

### 2.1。过滤收藏

`filter()`方法的一个常见用例是[处理集合](/web/20220926153229/https://www.baeldung.com/java-collection-filtering)。

让我们列出超过 100 个`points.`的客户。为此，我们可以使用 lambda 表达式:

```java
List<Customer> customersWithMoreThan100Points = customers
  .stream()
  .filter(c -> c.getPoints() > 100)
  .collect(Collectors.toList());
```

我们还可以使用[方法引用](/web/20220926153229/https://www.baeldung.com/java-8-double-colon-operator)，这是 lambda 表达式的简写:

```java
List<Customer> customersWithMoreThan100Points = customers
  .stream()
  .filter(Customer::hasOverHundredPoints)
  .collect(Collectors.toList());
```

在这种情况下，我们将`hasOverHundredPoints`方法添加到我们的`Customer`类中:

```java
public boolean hasOverHundredPoints() {
    return this.points > 100;
}
```

在这两种情况下，我们得到相同的结果:

```java
assertThat(customersWithMoreThan100Points).hasSize(2);
assertThat(customersWithMoreThan100Points).contains(sarah, charles);
```

### 2.2。使用多个标准过滤集合

此外，我们可以通过`filter()`使用多个条件。例如，我们可以通过`points`和`name`进行过滤:

```java
List<Customer> charlesWithMoreThan100Points = customers
  .stream()
  .filter(c -> c.getPoints() > 100 && c.getName().startsWith("Charles"))
  .collect(Collectors.toList());

assertThat(charlesWithMoreThan100Points).hasSize(1);
assertThat(charlesWithMoreThan100Points).contains(charles);
```

## 3。处理异常

到目前为止，我们一直使用不抛出异常的谓词过滤器。事实上，Java 中的**函数接口没有声明任何检查或未检查的异常**。

接下来我们将展示一些不同的方法来处理 lambda 表达式中的[异常。](/web/20220926153229/https://www.baeldung.com/java-lambda-exceptions)

### 3.1。使用自定义包装器

首先，我们将开始向我们的`Customer` `:`添加一个`profilePhotoUrl`

```java
private String profilePhotoUrl;
```

此外，让我们添加一个简单的`hasValidProfilePhoto()`方法来检查概要文件的可用性:

```java
public boolean hasValidProfilePhoto() throws IOException {
    URL url = new URL(this.profilePhotoUrl);
    HttpsURLConnection connection = (HttpsURLConnection) url.openConnection();
    return connection.getResponseCode() == HttpURLConnection.HTTP_OK;
}
```

我们可以看到，`hasValidProfilePhoto()`方法抛出了一个`IOException`。现在，如果我们尝试用这种方法过滤客户:

```java
List<Customer> customersWithValidProfilePhoto = customers
  .stream()
  .filter(Customer::hasValidProfilePhoto)
  .collect(Collectors.toList());
```

我们将看到以下错误:

```java
Incompatible thrown types java.io.IOException in functional expression
```

为了处理它，我们可以使用的替代方法之一是用 try-catch 块包装它:

```java
List<Customer> customersWithValidProfilePhoto = customers
  .stream()
  .filter(c -> {
      try {
          return c.hasValidProfilePhoto();
      } catch (IOException e) {
          //handle exception
      }
      return false;
  })
  .collect(Collectors.toList());
```

如果我们需要从谓词中抛出一个异常，我们可以像`RuntimeException`一样将它封装在一个未检查的异常中。

### 3.2。使用投掷功能

或者，我们可以使用 throwing 函数库。

ThrowingFunction 是一个开源库，允许我们在 Java 函数接口中处理检查异常。

让我们从将 [`throwing-function`依赖项](https://web.archive.org/web/20220926153229/https://search.maven.org/search?q=g:pl.touk%20AND%20a:throwing-function%26core%3Dgav)添加到 pom 开始:

```java
<dependency>
    <groupId>pl.touk</groupId>
    <artifactId>throwing-function</artifactId>
    <version>1.3</version>
</dependency>
```

为了处理谓词中的异常，这个库为我们提供了`ThrowingPredicate`类，它有`unchecked()`方法来包装被检查的异常。

让我们来看看它的实际应用:

```java
List customersWithValidProfilePhoto = customers
  .stream()
  .filter(ThrowingPredicate.unchecked(Customer::hasValidProfilePhoto))
  .collect(Collectors.toList());
```

## 4。结论

在本文中，我们看到了一个如何使用`filter()`方法处理流的例子。我们还探索了一些处理异常的替代方法。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926153229/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams)