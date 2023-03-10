# Java 中的 transient 关键字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-transient-keyword>

## 1.介绍

在本文中，我们将首先理解`transient`关键字，然后通过例子来看看它的行为。

## 2.`transient`的用法

在转到`transient`之前，让我们先了解一下序列化，因为它是在序列化的上下文中使用的。

**[序列化](/web/20220901175137/https://www.baeldung.com/java-serialization)是将对象转换成字节流的过程，反序列化与之相反**。

**当我们将任何变量标记为`transient,`时，该变量就不会被序列化**。因为`transient`字段不存在于对象的序列化形式中，所以在序列化形式之外创建对象时，反序列化过程将使用这些字段的默认值。

`transient`关键字在一些情况下很有用:

*   我们可以将它用于派生字段
*   这对于不表示对象状态的字段很有用
*   我们将它用于任何不可序列化的引用

## 3.例子

为了查看它的运行情况，让我们首先创建一个我们想要序列化其对象的`Book`类:

```java
public class Book implements Serializable {
    private static final long serialVersionUID = -2936687026040726549L;
    private String bookName;
    private transient String description;
    private transient int copies;

    // getters and setters
}
```

这里，我们将`description`和`copies`标记为`transient`字段。

创建类后，我们将创建该类的一个对象:

```java
Book book = new Book();
book.setBookName("Java Reference");
book.setDescription("will not be saved");
book.setCopies(25);
```

现在，我们将对象序列化到一个文件中:

```java
public static void serialize(Book book) throws Exception {
    FileOutputStream file = new FileOutputStream(fileName);
    ObjectOutputStream out = new ObjectOutputStream(file);
    out.writeObject(book);
    out.close();
    file.close();
}
```

现在让我们从文件中反序列化对象:

```java
public static Book deserialize() throws Exception {
    FileInputStream file = new FileInputStream(fileName);
    ObjectInputStream in = new ObjectInputStream(file);
    Book book = (Book) in.readObject();
    in.close();
    file.close();
    return book;
}
```

最后，我们将验证`book`对象的值:

```java
assertEquals("Java Reference", book.getBookName());
assertNull(book.getDescription());
assertEquals(0, book.getCopies());
```

这里我们看到`bookName`已经被正确地持久化了。另一方面，`copies`字段的值是`0`，而`description`是`null –`，这是它们各自数据类型的默认值，而不是原始值。

## 4.使用`final`的行为

现在，让我们看一个案例，我们将把`transient`和`final`关键字一起使用。为此，首先，我们将在我们的`Book`类中添加一个`final transient`元素，然后创建一个空的`Book`对象:

```java
public class Book implements Serializable {
    // existing fields    

    private final transient String bookCategory = "Fiction";

    // getters and setters
}
```

```java
Book book = new Book();
```

**最后一个修饰符没有影响**–因为字段是`transient`，没有为该字段保存任何值。在反序列化过程中，新的`Book`对象获得在`Book`类中定义的默认值`Fiction`，但是该值不是来自序列化数据:

```java
assertEquals("Fiction", book.getBookCategory());
```

## 5.结论

在本文中，我们看到了关键字`transient`的用法及其在序列化和反序列化中的行为。

和往常一样，GitHub 上的所有代码[都是可用的。](https://web.archive.org/web/20220901175137/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-3)