# JSON 指针概述

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/json-pointer>

## 1.**概述**

在本教程中，**我们将展示如何使用 JSON 指针从 JSON 数据**中导航和获取信息。

我们还将展示如何执行诸如插入新数据或更新现有键值之类的操作。

## 2。依赖性设置

首先，我们需要向我们的`pom.xml`添加一些依赖项:

```java
<dependency>
    <groupId>org.glassfish</groupId>
    <artifactId>javax.json</artifactId>
    <version>1.1.2</version>
</dependency>
```

## 3。JSON 指针

JSON 是一种用于系统间交换数据的轻量级格式，最初由道格拉斯·克洛克福特制定。

虽然它使用了`JavaScript`语法，但是**是独立于语言的，因为结果是纯文本**。

JSON 指针( [RFC 6901](https://web.archive.org/web/20221126231301/https://tools.ietf.org/html/rfc6901) )是来自 JSON Processing 1.1 API([JSR 374](https://web.archive.org/web/20221126231301/https://jcp.org/en/jsr/detail?id=374))的一个特性。它定义了一个可以用来访问 JSON 文档上的值的`String`。它可以与 XPath 对 XML 文档所做的事情联系起来。

通过使用 JSON 指针，我们可以从 JSON 文件中获取数据并修改它。

## 4。访问数据

我们将看到一些如何通过实现一个名为`JsonPointerCrud`的类来执行操作的例子。

假设我们有一个名为`books.json`的 JSON 文件，其内容为:

```java
{
    "library": "My Personal Library",
    "books": [
        { "title":"Title 1", "author":"Jane Doe" },
        { "title":"Title 2", "author":"John Doe" }
    ]
}
```

为了访问该文件中的数据，我们需要读取它并将其解析为一个`JsonStructure`。我们可以使用接受一个`InputStream`或一个`FileReader`的`Json.createReader()`方法来实现它。

我们可以这样做:

```java
JsonReader reader = Json.createReader(new FileReader("books.json"));
JsonStructure jsonStructure = reader.read();
reader.close();
```

内容将存储在一个`JsonStructure`对象上。这是我们将用来执行下一步操作的对象。

### 4.1。从文件中获取数据

为了获取单个值，我们创建了一个`JsonPointer`，通知我们想要从哪个标签获取值:

```java
JsonPointer jsonPointer = Json.createPointer("/library");
JsonString jsonString = (JsonString) jsonPointer.getValue(jsonStructure);
System.out.println(jsonString.getString());
```

注意**这个** `String` **的第一个字符是一个'**`/'`**——这是一个句法要求**。

这个片段的结果是:

```java
My Personal Library
```

要从列表中获取一个值，我们需要指定它的索引(其中第一个索引为 0):

```java
JsonPointer jsonPointer = Json.createPointer("/books/1");
JsonObject jsonObject = (JsonObject) jsonPointer.getValue(jsonStructure);
System.out.println(jsonObject.toString());
```

这将输出:

```java
"title":"Title 2", "author":"John Doe"
```

### 4.2。检查文件中是否存在密钥

通过方法`containsValue`，我们可以检查用于创建指针的值是否存在于 JSON 文件中:

```java
JsonPointer jsonPointer = Json.createPointer("/library");
boolean found = jsonPointer.containsValue(jsonStructure);
System.out.println(found); 
```

这个片段的结果是:

```java
true
```

### 4.3。插入新键值

如果我们需要向 JSON 添加一个新值，那么`createValue`就是处理它的那个。**方法`createValue`被重载接受`String`、`int`、`long`、`double`、`BigDecimal`和`BigInteger:`、**

```java
JsonPointer jsonPointer = Json.createPointer("/total");
JsonNumber jsonNumber = Json.createValue(2);
jsonStructure = jsonPointer.add(jsonStructure, jsonNumber);
System.out.println(jsonStructure);
```

同样，我们的输出是:

```java
{
    "library": "My Personal Library",
    "total": 2,
    "books": [
        { "title":"Title 1", "author":"Jane Doe" },
        { "title":"Title 2", "author":"John Doe" }
    ]
}
```

### 4.4。更新键值

**要更新一个值，我们需要先创建新值**。在创建值之后，我们从使用 key 参数创建的指针中使用`replace`方法:

```java
JsonPointer jsonPointer = Json.createPointer("/total");
JsonNumber jsonNumberNewValue = Json.createValue(5);
jsonStructure = jsonPointer.replace(jsonStructure, jsonNumberNewValue);
System.out.println(jsonStructure);
```

输出:

```java
{
    "library": "My Personal Library",
    "total": 5,
    "books": [
        { "title":"Title 1", "author":"Jane Doe" },
        { "title":"Title 2", "author":"John Doe" }
    ]
}
```

### 4.5。取下一把钥匙

要删除一个键，我们首先创建一个指向该键的指针。然后我们使用 remove 方法:

```java
JsonPointer jsonPointer = Json.createPointer("/library");
jsonPointer.getValue(jsonStructure);
jsonStructure = jsonPointer.remove(jsonStructure);
System.out.println(jsonStructure);
```

导致:

```java
{
    "total": 5,
    "books": [
        { "title":"Title 1", "author":"Jane Doe" },
        { "title":"Title 2", "author":"John Doe" }
    ]
}
```

### 4.6。显示文件的全部内容

如果指针是用空的`String`创建的，则检索全部内容:

```java
JsonPointer jsonPointer = Json.createPointer("");
JsonObject jsonObject = (JsonObject) jsonPointer.getValue(jsonStructure);
System.out.println(jsonObject.toString());
```

这个代码示例将输出`jsonStructure`的全部内容。

## 5。结论

在这篇简短的文章中，我们介绍了如何使用 JSON 指针对 JSON 数据执行各种操作。

而且和往常一样，与本教程相关的代码在 GitHub 上 [over。](https://web.archive.org/web/20221126231301/https://github.com/eugenp/tutorials/tree/master/json-modules/json)