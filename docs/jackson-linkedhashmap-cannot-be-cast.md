# Jackson:Java . util . linked hashmap 不能转换为 X

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-linkedhashmap-cannot-be-cast>

## 1.概观

Jackson 是一个广泛使用的 Java 库，它允许我们方便地序列化/反序列化 JSON 或 XML。

有时，当我们试图将 JSON 或 XML 反序列化为一个对象集合时，我们可能会遇到`java.lang.ClassCastException: java.util.LinkedHashMap cannot be cast to X`。

在本教程中，我们将讨论为什么会出现这个异常，以及如何解决这个问题。

## 2.理解问题

让我们创建一个简单的 Java 应用程序来重现这个异常，以了解异常何时会发生。

### 2.1.创建 POJO 类

让我们从一个简单的 POJO 类开始:

```java
public class Book {
    private Integer bookId;
    private String title;
    private String author;
    //getters, setters, constructors, equals and hashcode omitted
}
```

假设我们有一个由 JSON 数组组成的`books.json`文件，其中包含三本书:

```java
[ {
    "bookId" : 1,
    "title" : "A Song of Ice and Fire",
    "author" : "George R. R. Martin"
}, {
    "bookId" : 2,
    "title" : "The Hitchhiker's Guide to the Galaxy",
    "author" : "Douglas Adams"
}, {
    "bookId" : 3,
    "title" : "Hackers And Painters",
    "author" : "Paul Graham"
} ] 
```

接下来，我们将看到当我们尝试将 JSON 实例反序列化到`List<Book>`时会发生什么。

### 2.2.将 JSON 反序列化到`List<Book>`

让我们看看是否可以通过将这个 JSON 文件反序列化为一个`List<Book>`对象并从中读取元素来重现类转换问题:

```java
@Test
void givenJsonString_whenDeserializingToList_thenThrowingClassCastException() 
  throws JsonProcessingException {
    String jsonString = readFile("/to-java-collection/books.json");
    List<Book> bookList = objectMapper.readValue(jsonString, ArrayList.class);
    assertThat(bookList).size().isEqualTo(3);
    assertThatExceptionOfType(ClassCastException.class)
      .isThrownBy(() -> bookList.get(0).getBookId())
      .withMessageMatching(".*java.util.LinkedHashMap cannot be cast to .*com.baeldung.jackson.tocollection.Book.*");
}
```

我们已经使用了 [AssertJ](/web/20221208143917/https://www.baeldung.com/assertj-exception-assertion) 库来验证当我们调用`bookList.get(0).getBookId()`时，预期的异常被抛出，并且它的消息与我们的问题语句中记录的消息相匹配。

测试通过，意味着我们成功地重现了问题。

### 2.3.引发异常的原因

如果我们仔细看看异常消息`class java.util.LinkedHashMap cannot be cast to class … Book`，可能会出现几个问题。

我们已经用类型`List<Book>`声明了变量`bookList`，但是为什么 Jackson 试图将`LinkedHashMap` 类型转换成我们的`Book` 类？再者，`LinkedHashMap`从何而来？

首先，我们确实用类型`List<Book>`声明了`bookList`。然而，当我们调用 [`objectMapper.readValue()`](https://web.archive.org/web/20221208143917/https://fasterxml.github.io/jackson-databind/javadoc/2.7/com/fasterxml/jackson/databind/ObjectMapper.html#readValue(java.lang.String,%20java.lang.Class)) 方法时，**我们将`ArrayList.class`作为`Class`对象传递。因此，Jackson 会将 JSON 内容反序列化为一个 [`ArrayList`](/web/20221208143917/https://www.baeldung.com/java-arraylist) 对象，但是它不知道`ArrayList` 对象中应该有什么类型的元素。**

其次，**当 Jackson 试图在 JSON 中反序列化一个对象，但没有给出目标类型信息时，它将使用默认类型:`[LinkedHashMap](/web/20221208143917/https://www.baeldung.com/java-linked-hashmap)`。**换句话说，在反序列化之后，我们将得到一个`ArrayList<LinkedHashMap>`对象。在`Map`中，关键字是属性的名称，例如`“bookId”`、`“title”`等。

这些值是相应属性的值:

[![booksMap](img/f9ea49112a4e15439363eef011c9535a.png)](/web/20221208143917/https://www.baeldung.com/wp-content/uploads/2021/01/booksMap.png)

既然我们了解了问题的原因，让我们来讨论如何解决它。

## 3.通过`TypeReference`到`objectMapper.readValue()`

为了解决这个问题，我们需要以某种方式让 Jackson 知道元素的类型。但是，编译器不允许我们做类似于`objectMapper.readValue(jsonString, ArrayList<Book>.class)`的事情。

相反，**我们可以将一个`[TypeReference](https://web.archive.org/web/20221208143917/https://fasterxml.github.io/jackson-core/javadoc/2.2.0/com/fasterxml/jackson/core/type/TypeReference.html)`对象传递给 [`objectMapper.readValue(String content, TypeReference valueTypeRef)`](https://web.archive.org/web/20221208143917/https://fasterxml.github.io/jackson-databind/javadoc/2.7/com/fasterxml/jackson/databind/ObjectMapper.html#readValue(java.lang.String,%20com.fasterxml.jackson.core.type.TypeReference)) 方法。**

在这种情况下，我们只需要将`new TypeReference<List<Book>>() {}`作为第二个参数传递:

```java
@Test
void givenJsonString_whenDeserializingWithTypeReference_thenGetExpectedList() 
  throws JsonProcessingException {
    String jsonString = readFile("/to-java-collection/books.json");
    List<Book> bookList = objectMapper.readValue(jsonString, new TypeReference<List<Book>>() {});
    assertThat(bookList.get(0)).isInstanceOf(Book.class);
    assertThat(bookList).isEqualTo(expectedBookList);
} 
```

如果我们进行测试，它会通过的。所以，传递一个`TypeReference`对象就解决了我们的问题。

## 4.通过`JavaType`到`objectMapper.readValue()`

在上一节中，我们谈到了将一个`Class`对象或一个`TypeReference`对象作为第二个参数来调用`objectMapper.readValue()`方法。

`objectMapper.readValue()`方法仍然接受一个`[JavaType](https://web.archive.org/web/20221208143917/https://fasterxml.github.io/jackson-databind/javadoc/2.7/com/fasterxml/jackson/databind/JavaType.html)`对象作为第二个参数。**`JavaType`是类型标记类的基类。它将被反序列化器使用，以便反序列化器在反序列化期间知道目标类型是什么。**

我们可以通过一个 [`TypeFactory`](https://web.archive.org/web/20221208143917/https://fasterxml.github.io/jackson-databind/javadoc/2.7/com/fasterxml/jackson/databind/type/TypeFactory.html) 实例构造一个`JavaType `对象，我们可以从`objectMapper.getTypeFactory()`中检索到`TypeFactory`对象。

让我们回到书中的例子。在这个例子中，我们想要的目标类型是`ArrayList<Book>`。

因此，我们可以用这个要求构造一个`CollectionType`:

```java
objectMapper.getTypeFactory().constructCollectionType(ArrayList.class, Book.class);
```

现在让我们编写一个单元测试，看看向`readValue()`方法传递一个`JavaType`是否能解决我们的问题:

```java
@Test
void givenJsonString_whenDeserializingWithJavaType_thenGetExpectedList() 
  throws JsonProcessingException {
    String jsonString = readFile("/to-java-collection/books.json");
    CollectionType listType = 
      objectMapper.getTypeFactory().constructCollectionType(ArrayList.class, Book.class);
    List<Book> bookList = objectMapper.readValue(jsonString, listType);
    assertThat(bookList.get(0)).isInstanceOf(Book.class);
    assertThat(bookList).isEqualTo(expectedBookList);
} 
```

如果我们运行它，测试就通过了。所以，这个问题也可以这样解决。

## 5.使用`JsonNode`对象和`objectMapper.convertValue()`方法

我们已经看到了向`objectMapper.readValue()`方法传递一个`TypeReference`或`JavaType`对象的解决方案。

或者，我们可以[处理 Jackson](/web/20221208143917/https://www.baeldung.com/jackson-json-node-tree-model) 中的树模型节点，然后**通过调用`[objectMapper.convertValue()](https://web.archive.org/web/20221208143917/https://fasterxml.github.io/jackson-databind/javadoc/2.7/com/fasterxml/jackson/databind/ObjectMapper.html#convertValue(java.lang.Object,%20com.fasterxml.jackson.core.type.TypeReference))`方法将 [`JsonNode`](https://web.archive.org/web/20221208143917/https://fasterxml.github.io/jackson-databind/javadoc/2.7/com/fasterxml/jackson/databind/JsonNode.html) 对象转换成所需的类型。**

类似地，我们可以将一个对象`TypeReference`或`JavaType`传递给`objectMapper.convertValue()`方法。

让我们看看每种方法的实际应用。

首先，让我们使用一个`TypeReference `对象和`objectMapper.convertValue()`方法创建一个测试方法:

```java
@Test
void givenJsonString_whenDeserializingWithConvertValueAndTypeReference_thenGetExpectedList() 
  throws JsonProcessingException {
    String jsonString = readFile("/to-java-collection/books.json");
    JsonNode jsonNode = objectMapper.readTree(jsonString);
    List<Book> bookList = objectMapper.convertValue(jsonNode, new TypeReference<List<Book>>() {});
    assertThat(bookList.get(0)).isInstanceOf(Book.class);
    assertThat(bookList).isEqualTo(expectedBookList);
} 
```

现在让我们看看当我们将一个`JavaType`对象传递给`objectMapper.convertValue()`方法时会发生什么:

```java
@Test
void givenJsonString_whenDeserializingWithConvertValueAndJavaType_thenGetExpectedList() 
  throws JsonProcessingException {
    String jsonString = readFile("/to-java-collection/books.json");
    JsonNode jsonNode = objectMapper.readTree(jsonString);
    List<Book> bookList = objectMapper.convertValue(jsonNode, 
      objectMapper.getTypeFactory().constructCollectionType(ArrayList.class, Book.class));
    assertThat(bookList.get(0)).isInstanceOf(Book.class);
    assertThat(bookList).isEqualTo(expectedBookList);
} 
```

如果我们运行这两个测试，它们都会通过。因此，使用`objectMapper.convertValue()`方法是解决问题的替代方法。

## 6.创建泛型反序列化方法

到目前为止，我们已经讨论了如何解决将 JSON 数组反序列化为 Java 集合时的类转换问题。在现实世界中，我们可能希望创建一个通用方法来处理不同的元素类型。

现在对我们来说这不是一件难事。

**我们可以在调用`objectMapper.readValue()` 方法**时传递一个`JavaType`对象:

```java
public static <T> List<T> jsonArrayToList(String json, Class<T> elementClass) throws IOException {
    ObjectMapper objectMapper = new ObjectMapper();
    CollectionType listType = 
      objectMapper.getTypeFactory().constructCollectionType(ArrayList.class, elementClass);
    return objectMapper.readValue(json, listType);
} 
```

接下来，让我们创建一个单元测试方法来验证它是否如我们预期的那样工作:

```java
@Test
void givenJsonString_whenCalljsonArrayToList_thenGetExpectedList() throws IOException {
    String jsonString = readFile("/to-java-collection/books.json");
    List<Book> bookList = JsonToCollectionUtil.jsonArrayToList(jsonString, Book.class);
    assertThat(bookList.get(0)).isInstanceOf(Book.class);
    assertThat(bookList).isEqualTo(expectedBookList);
} 
```

如果我们运行它，测试将通过。

**为什么不使用`TypeReference`方法**来构建泛型方法，因为它看起来更紧凑？

让我们创建一个通用的实用方法，并将相应的`TypeReference`对象传递给`objectMapper.readValue()`方法:

```java
public static <T> List<T> jsonArrayToList(String json, Class<T> elementClass) throws IOException {
    return new ObjectMapper().readValue(json, new TypeReference<List<T>>() {});
} 
```

这个方法看起来很简单。

如果我们再次运行测试方法，我们将得到以下结果:

```java
java.lang.ClassCastException: class java.util.LinkedHashMap cannot be cast to class com.baeldung...Book ...
```

糟糕，出现异常！

我们已经将一个`TypeReference`对象传递给了`readValue()`方法，我们之前已经看到这种方式将解决类的类型转换问题。那么，**为什么在这种情况下我们会看到同样的异常**？

因为我们的方法是通用的。**类型参数`T`不能在运行时**具体化，即使我们用类型参数`T`传递一个`TypeReference`实例。

## 7.用 Jackson 进行 XML 反序列化

除了 JSON 序列化/反序列化，Jackson 库也可以用来序列化/反序列化 XML。

让我们举一个简单的例子来检查将 XML 反序列化为 Java 集合时是否会发生同样的问题。

首先，让我们创建一个 XML 文件`books.xml`:

```java
<ArrayList>
    <item>
        <bookId>1</bookId>
        <title>A Song of Ice and Fire</title>
        <author>George R. R. Martin</author>
    </item>
    <item>
        <bookId>2</bookId>
        <title>The Hitchhiker's Guide to the Galaxy</title>
        <author>Douglas Adams</author>
    </item>
    <item>
        <bookId>3</bookId>
        <title>Hackers And Painters</title>
        <author>Paul Graham</author>
    </item>
</ArrayList> 
```

接下来，就像我们对 JSON 文件所做的那样，我们创建另一个单元测试方法来验证是否会抛出类转换异常:

```java
@Test
void givenXml_whenDeserializingToList_thenThrowingClassCastException() 
  throws JsonProcessingException {
    String xml = readFile("/to-java-collection/books.xml");
    List<Book> bookList = xmlMapper.readValue(xml, ArrayList.class);
    assertThat(bookList).size().isEqualTo(3);
    assertThatExceptionOfType(ClassCastException.class)
      .isThrownBy(() -> bookList.get(0).getBookId())
      .withMessageMatching(".*java.util.LinkedHashMap cannot be cast to .*com.baeldung.jackson.tocollection.Book.*");
} 
```

如果我们试一试，我们的测试就会通过。也就是说，在 XML 反序列化中也会出现同样的问题。

然而，如果我们知道如何解决 JSON 反序列化，那么在 XML 反序列化中解决它就相当简单了。

**由于`[XmlMapper](https://web.archive.org/web/20221208143917/https://fasterxml.github.io/jackson-dataformat-xml/javadoc/2.7/com/fasterxml/jackson/dataformat/xml/XmlMapper.html)`是`ObjectMapper`的子类，我们针对 JSON 反序列化提出的所有解决方案也适用于 XML 反序列化。**

例如，我们可以将一个`TypeReference`对象传递给`xmlMapper.readValue()`方法来解决问题:

```java
@Test
void givenXml_whenDeserializingWithTypeReference_thenGetExpectedList() 
  throws JsonProcessingException {
    String xml = readFile("/to-java-collection/books.xml");
    List<Book> bookList = xmlMapper.readValue(xml, new TypeReference<List<Book>>() {});
    assertThat(bookList.get(0)).isInstanceOf(Book.class);
    assertThat(bookList).isEqualTo(expectedBookList);
} 
```

## 8.结论

在本文中，我们已经讨论了为什么在使用 Jackson 反序列化 JSON 或 XML 时会出现`java.util.LinkedHashMap cannot be cast to X`异常。

在那之后，我们通过例子提出了解决问题的不同方法。

和往常一样，这篇文章中的代码都可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions-2)