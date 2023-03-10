# Javadoc: @see 和@link

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javadoc-see-vs-link>

## 1.概观

Javadoc 是一种从 Java 源代码生成 HTML 格式的现代 Java 文档的好方法。

在本教程中，我们将关注文档注释中的`@see`和@link 标签。

## 2.`@see`

[`@see`](/web/20221208143856/https://www.baeldung.com/javadoc-linking-external-url) 标签的格式相当简单:

```java
@see reference
```

例如，我们可以用它来标记到官方 Java 文档的外部链接:

```java
/**
 * @see <a href="https://docs.oracle.com/en/java/">Java Dcoumentation</a>
 */
```

简而言之，**当我们需要指向引用的链接或文本条目时，我们使用`@see`标签。**这个标签给引用添加了一个“另请参阅”的标题。一个文档注释可以包含任意数量的`@see`标签，所有这些标签都可以分组在同一个标题下。甲骨文文档给了我们一些如何使用它的详细说明。这个标记是有效的，可以用在任何文档注释中，包括包、概述、构造函数、类和接口。`@see`标签有三种变体，将在下面讨论。

### 2.1。 `@see`“文本串”

**这为一个文本字符串添加了一个文本条目，但没有生成任何类型的链接。**该字符串可以引用一本书的页面或任何其他需要为任何进一步的上下文提供的附加信息。Javadoc 工具通过寻找双引号(")作为第一个字符来区别文本字符串。让我们考虑一个例子:

```java
/**
 * @see "This method performs some function."
 */
public void someMethod() {
  // do Something...
}
```

这将呈现为:

[![Screenshot-2022-01-23-at-12.58.34-PM](img/16c18d2969190205040ad22877f83c0c.png)](/web/20221208143856/https://www.baeldung.com/wp-content/uploads/2022/01/Screenshot-2022-01-23-at-12.58.34-PM.png)

### 2.2。`@see`标签

**这将添加一个定义 URL 的链接。**URL 可以是相对或绝对的 URL 值。Javadoc 工具通过将小于号(<)作为第一个字符来区分文本字符串，然后添加一个定义`URL` # `value`的链接。`URL` # `value`是一个相对或绝对的网址。Javadoc 工具通过寻找小于号(`<`)作为第一个字符来区分这种情况和其他情况。考虑以下在锚标记中显示链接的示例:

```java
/**
 * @see <a href="http://www.baeldung.com">Baeldung</a>
 */
public void someMethodV2() {
  // do Something...
}
```

这将生成如下文本:

```java
[![Screenshot-2022-01-23-at-1.00.28-PM](img/6fac15fa1f84d5781c007a84b6957783.png)](/web/20221208143856/https://www.baeldung.com/wp-content/uploads/2022/01/Screenshot-2022-01-23-at-1.00.28-PM.png) 
```

### `**2.3\. @see package.class#member label**`

这将添加一个定义函数的链接。标签是可选的，并且在未定义时使用类中的原始成员名。-no 限定符选项从可见文本中删除包名。package . class #成员指的是元素名，如包、类、接口或字段名。考虑以下例子:

```java
/**
 * @see String#toLowerCase() convertToLowerCase
 */
public void addingSeeToAMethod() {
  // do Something...
}
```

为上面的注释生成的标准 HTML 看起来会像这样:

```java
<dl> 
<dt><b>See Also:</b> 
<dd><a href="../../java/lang/String#toLowerCase()"><code>convertToLowerCase<code></a> 
</dl>
```

这将在浏览器中转换为以下输出:

[![Screenshot-2022-01-23-at-1.01.54-PM](img/bd238f2bd547519e9a0e82efc42d60ea.png)](/web/20221208143856/https://www.baeldung.com/wp-content/uploads/2022/01/Screenshot-2022-01-23-at-1.01.54-PM.png)

注意:我们可以在一个标签中使用多个空格。

## 3.`@link`

这是一个内嵌标签。 [`@link`](/web/20221208143856/https://www.baeldung.com/java-method-in-javadoc) 标签的格式相当简单明了:

```java
{@link  package.class#member  label}
```

让我们看看下面这个使用`@link`标签的例子:

```java
/**
 * Use the {@link String#equalsIgnoreCase(String) equalsMethod} to check if two strings are equal.
 */
```

**插入一个带有可见文本标签的内联链接。**文本标签指向指定包、类或成员名的文档。这个标签可以在任何地方使用，包括概述、包、类、方法、字段等。这也可以用在标签的文本部分，如`@return`、`@param`和`@deprecated`。这个标签与`@see`非常相似，因为两者都需要相同的引用，并接受相同的`package.class#member`和标签语法。

为上面的注释生成的标准 HTML 看起来会像这样:

```java
Use the <code>equalsMethod</code> to check if two strings are equal.
```

这将在浏览器中呈现如下:

[![Screenshot-2022-01-23-at-1.13.20-PM](img/687ab624047f9f18aeca410e67b32600.png)](/web/20221208143856/https://www.baeldung.com/wp-content/uploads/2022/01/Screenshot-2022-01-23-at-1.13.20-PM.png)

## 4.`@see`和`@link`的相似之处

在这一节中，我们来看看 *@see* 和`@link`标签之间的相似之处。

**我们可以在一个类、包或方法中多次使用`@see`和`@link`标签。**`@see`标签声明了指向外部链接、类或方法的引用。`@link`标签也可以多次用于声明内联链接，或者与其他块标签形成对比。考虑下面显示`@see`和`@link`标签语法的例子:

```java
/**
 * Use {@link String#toLowerCase()} for conversion to lower case alphabets.
 * @deprecated As of Java 1.8 {@link java.security.Certificate} is deprecated.
 * @return {@link String} object
 * @see <a href="http://www.baeldung.com">Baeldung</a>
 * @see String#hashCode() hashCode
 */
public String someMethod() {
  // perform some function
  return "";
}
```

## 5.`@see`和`@link`的区别

在这一节中，让我们看看`@see`和`@link`标签之间的区别。

### 5.1.两者属于不同的标签

我们可以将文档注释分为两种类型:

*   阻止标签
*   内嵌标签

块标记具有出现在行首的@标记形式。它忽略前导星号、空白和分隔符(`/**`)。`@see`就是块标签的一个例子。

内联标签出现在大括号内，并且具有`{@tag}`的形式。并且它应该在任何允许文本的地方被允许和解释。我们可以在内嵌标签中使用其他标签。它可以存在于描述或注释中的任何地方:

```java
/**
 * Some description here.
 *
 * @see java.lang.Object
 * @return  This will return {@link #toString() string} response.
 */
```

**`@see`和`@link`标签的主要区别在于，一个生成内嵌链接，而另一个在“参见”部分显示链接。**此外，`@link`标签以花括号开始和结束，将它与行内文本的其余部分分开。由于`@see`标签是一个块标签，我们将显式地创建它:

考虑以下显示`@see`和`@link`标签输出的示例:

```java
/**
 *
 * Use {@link String#toLowerCase()} for conversion to lower case alphabets.
 * @deprecated As of Java 1.8 {@link java.security.Certificate} is deprecated.
 * @return {@link String} object
 * @see <a href="http://www.baeldung.com">Baeldung</a>
 * @see String#hashCode() hashCode
 */
public String someMethod() {
  // perform some function
  return "";
}
```

这将在 Javadoc 中生成以下输出:

```java
![Screenshot-2022-01-23-at-1.46.46-PM](img/f1c10a8042ba0902a3cdbdc4290a9789.png)

```

### 5.3.使用`@see`和`@link`标签

**一个 block 标签 是 独立使用，不能 是 与其他标签一起使用 。**另一方面，内嵌标签 是用在文档注释里面或者作为内嵌链接。**我们也可以将`@link`标签与其他块标签一起使用。**考虑下面的例子，我们将`@link`标签与其他块标签一起使用:

```java
/**
 * Some description here.
 *
 * @see java.lang.Object
 * This is a {@link #getClass() } method.
 * @see #getClass() Use {@link #toString()} for String conversion.
 * @deprecated As of JDK 1.1, replaced by {@link #someMethod()}
 */
```

## 6.结论

在本教程中，我们讨论了如何正确使用`@see`和`@link`标签。然后我们描述了评论标签的类型和使用`@see`的不同方式。最后，我们还描述了`@see`和`@link`标签之间的一些主要区别。简而言之，我们可以使用`@see`标签来显示指向引用的链接或文本，而`@link`标签将文本中的链接或其他标签描述为 inline。