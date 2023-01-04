# 对 Jackson 使用 Optional

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-optional>

## 1。简介

在本文中，我们将对 [`Optional`](https://web.archive.org/web/20230102111125/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html) 类进行概述，然后解释在 Jackson 中使用它时可能会遇到的一些问题。

接下来，我们将引入一个解决方案，让 Jackson 将`Optionals`视为普通的可空对象。

## 2。问题概述

首先，让我们看看当我们试图用 Jackson 序列化和反序列化`Optionals`时会发生什么。

### 2.1。Maven 依赖关系

要使用 Jackson，让我们确保使用的是最新版本的:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.13.3</version>
</dependency>
```

### 2.2。我们的图书对象

然后，让我们创建一个包含一个普通字段和一个字段的类`Book,` :

```java
public class Book {
   String title;
   Optional<String> subTitle;

   // getters and setters omitted
}
```

请记住,`Optionals`不应该用作字段，我们这样做是为了说明这个问题。

### 2.3。序列化

现在，让我们实例化一个`Book`:

```java
Book book = new Book();
book.setTitle("Oliver Twist");
book.setSubTitle(Optional.of("The Parish Boy's Progress"));
```

最后，让我们尝试使用杰克逊 [`ObjectMapper`](https://web.archive.org/web/20230102111125/https://github.com/FasterXML/jackson-databind/blob/master/docs/javadoc/2.2/com/fasterxml/jackson/databind/ObjectMapper.html) 来序列化它:

```java
String result = mapper.writeValueAsString(book);
```

我们将看到,`Optional`字段的输出不包含它的值，而是一个嵌套的 JSON 对象，它有一个名为`present`的字段:

```java
{"title":"Oliver Twist","subTitle":{"present":true}}
```

虽然这看起来很奇怪，但实际上这是我们应该期待的。

在这种情况下，`isPresent()` 是`Optional` 类的公共 getter。这意味着它将被序列化为值`true`或`false`，这取决于它是否为空。这是 Jackson 默认的序列化行为。

如果我们仔细想想，我们想要的是实际的被序列化的字段的值。

### 2.4。反序列化

现在，让我们颠倒一下之前的例子，这次尝试将一个对象反序列化为一个`Optional.` ,我们会看到现在我们得到了一个`[JsonMappingException](https://web.archive.org/web/20230102111125/https://github.com/FasterXML/jackson-databind/blob/master/docs/javadoc/2.0/com/fasterxml/jackson/databind/JsonMappingException.html):`

```java
@Test(expected = JsonMappingException.class)
public void givenFieldWithValue_whenDeserializing_thenThrowException
    String bookJson = "{ \"title\": \"Oliver Twist\", \"subTitle\": \"foo\" }";
    Book result = mapper.readValue(bookJson, Book.class);
} 
```

让我们来查看堆栈跟踪:

```java
com.fasterxml.jackson.databind.JsonMappingException:
  Can not construct instance of java.util.Optional:
  no String-argument constructor/factory method to deserialize from String value ('The Parish Boy's Progress')
```

这种行为也是有意义的。本质上，Jackson 需要一个构造函数，它可以将`subtitle`的值作为参数。我们的`Optional` 字段不是这种情况。

## 3。解决方案

我们想要的是，Jackson 将一个空的`Optional` 视为`null,` ，将一个当前的`Optional` 视为一个表示其值的字段。

好在这个问题已经为我们解决了。 [Jackson 有一套处理 JDK 8 数据类型](https://web.archive.org/web/20230102111125/https://github.com/FasterXML/jackson-modules-java8)的模块，包括`Optional`。

### 3.1。Maven 依赖和注册

首先，让我们添加最新版本作为 Maven 依赖项:

```java
<dependency>
   <groupId>com.fasterxml.jackson.datatype</groupId>
   <artifactId>jackson-datatype-jdk8</artifactId>
   <version>2.13.3</version>
</dependency>
```

现在，我们需要做的就是用我们的`ObjectMapper`注册模块:

```java
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new Jdk8Module());
```

### 3.2。序列化

现在，让我们来测试一下。如果我们再次尝试序列化我们的`Book` 对象，我们会看到现在有一个`subtitle,` 而不是嵌套的 JSON:

```java
Book book = new Book();
book.setTitle("Oliver Twist");
book.setSubTitle(Optional.of("The Parish Boy's Progress"));
String serializedBook = mapper.writeValueAsString(book);

assertThat(from(serializedBook).getString("subTitle"))
  .isEqualTo("The Parish Boy's Progress");
```

如果我们尝试序列化一本空书，它将被存储为`null`:

```java
book.setSubTitle(Optional.empty());
String serializedBook = mapper.writeValueAsString(book);

assertThat(from(serializedBook).getString("subTitle")).isNull();
```

### 3.3。反序列化

现在，让我们重复我们的反序列化测试。如果我们重读我们的书，我们会发现我们不再得到一个`JsonMappingException:`

```java
Book newBook = mapper.readValue(result, Book.class);

assertThat(newBook.getSubTitle()).isEqualTo(Optional.of("The Parish Boy's Progress"));
```

最后，让我们再次重复测试，这次用`null.` 我们将再次看到我们没有得到`JsonMappingException,` ，事实上，有一个空的`Optional:`

```java
assertThat(newBook.getSubTitle()).isEqualTo(Optional.empty());
```

## 4。结论

我们已经展示了如何通过利用 JDK 8 数据类型模块来解决这个问题，演示了它如何使 Jackson 能够将一个空的`Optional` 作为`null,` ，将一个当前的`Optional` 作为一个普通字段。

这些例子的实现可以在 GitHub 上找到[；这是一个基于 Maven 的项目，所以应该很容易运行。](https://web.archive.org/web/20230102111125/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-core)