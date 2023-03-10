# 用 Gson 将数据保存到 JSON 文件中

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gson-save-file>

## 1。概述

Gson 是一个 Java 库，它允许我们将 Java 对象转换成 JSON 表示。我们也可以反过来使用它，将 JSON 字符串转换成等价的 Java 对象。

在这个快速教程中，我们将了解如何将各种 Java 数据类型保存为文件中的 JSON。

## 2。Maven 依赖关系

首先，我们需要在`pom.xml`中添加 Gson 依赖。这可在 [Maven Central](https://web.archive.org/web/20220524022511/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.code.gson%22%20AND%20a%3A%22gson%22) 获得:

```java
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
```

## 3。将数据保存到 JSON 文件

我们将使用 [`Gson`](https://web.archive.org/web/20220524022511/https://static.javadoc.io/com.google.code.gson/gson/2.8.5/com/google/gson/Gson.html) 类中的`[toJson(Object src, Appendable writer)](https://web.archive.org/web/20220524022511/https://static.javadoc.io/com.google.code.gson/gson/2.8.5/com/google/gson/Gson.html#toJson-java.lang.Object-java.lang.Appendable-) `方法将 Java 数据类型转换成 JSON 并存储在文件中。`Gson()`构造函数创建一个默认配置的`Gson`对象:

```java
Gson gson = new Gson();
```

现在，我们可以调用`toJson`()来转换和存储 Java 对象。

让我们探索一些 Java 中不同数据类型的例子。

### 3.1。原语

使用 GSON 将原语保存到 JSON 文件非常简单:

```java
gson.toJson(123.45, new FileWriter(filePath));
```

这里，`filePath`表示文件的位置。文件输出将只包含原始值:

```java
123.45
```

### 3.2。自定义对象

同样，我们可以将对象存储为 JSON。

首先，我们将创建一个简单的`User`类:

```java
public class User {
    private int id;
    private String name;
    private transient String nationality;

    public User(int id, String name, String nationality) {
        this.id = id;
        this.name = name;
        this.nationality = nationality;
    }

    public User(int id, String name) {
        this(id, name, null);
    }
}
```

现在，我们将把一个`User`对象存储为 JSON:

```java
User user = new User(1, "Tom Smith", "American");
gson.toJson(user, new FileWriter(filePath));
```

文件输出将是:

```java
{"id":1,"name":"Tom"}
```

如果一个字段被标记为`transient`，默认情况下它会被忽略，并且不会包含在 JSON 序列化或反序列化中。因此，`nationality` 字段不会出现在 JSON 输出中。

同样默认情况下，Gson 在序列化过程中省略空字段。所以如果我们考虑这个例子:

```java
gson.toJson(new User(1, null, "Unknown"), new FileWriter(filePath));
```

文件输出将是:

```java
{"id":1}
```

稍后我们将看到如何在序列化中包含空字段。

### 3.3。收藏

我们可以用类似的方式存储对象集合:

```java
User[] users = new User[] { new User(1, "Mike"), new User(2, "Tom") };
gson.toJson(users, new FileWriter(filePath));
```

在这种情况下，文件输出将是一组`User`对象:

```java
[{"id":1,"name":"Mike"},{"id":2,"name":"Tom"}]
```

## 4。使用`GsonBuilder`

为了调整默认的 Gson 配置设置，我们可以利用 [`GsonBuilder`](https://web.archive.org/web/20220524022511/https://static.javadoc.io/com.google.code.gson/gson/2.8.5/com/google/gson/GsonBuilder.html) 类。

该类遵循 builder 模式，通常首先调用各种配置方法来设置所需的选项，最后调用`create()`方法:

```java
Gson gson = new GsonBuilder()
  .setPrettyPrinting()
  .create();
```

这里，我们设置了默认设置为`false`的漂亮打印选项。类似地，为了在序列化中包含空值，我们可以调用`serializeNulls()`。这里的列出了可用的选项[。](https://web.archive.org/web/20220524022511/https://static.javadoc.io/com.google.code.gson/gson/2.8.5/com/google/gson/Gson.html#method.detail)

## 5。结论

在这篇简短的文章中，我们了解了如何将各种 Java 数据类型序列化到一个 JSON 文件中。为了探索关于 JSON 的各种文章，看看我们关于这个主题的其他教程。

和往常一样，代码片段可以在 GitHub 库中找到。