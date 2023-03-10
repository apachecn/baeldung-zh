# Javadoc: @version 和@since

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javadoc-version-since>

## 1.概观

Javadoc 是一种从 Java 源代码生成 HTML 格式文档的方法。

在本教程中，我们将关注文档注释中的`@version`和`@since`标签。

## 2。`@version`和`@since`和的用法

在本节中，我们将讨论如何正确使用`@version`和`@since`标签。

### 2.1.`@version`

`@version`标签的格式很简单:

```java
@version  version-text
```

例如，我们可以用它来表示 JDK 1.7:

```java
/**
 * @version JDK 1.7
 */
```

当我们使用`@version`标签时，它有两种不同的使用场景:

*   记录单个文件的版本
*   标记整个软件的版本

显然，我们可以看到这两个场景之间存在差异。这是因为单个文件的版本可能与软件的版本不兼容。此外，不同的文件可能有不同的文件版本。那么，我们应该如何使用`@version`标签呢？

**过去，Sun 使用`@version`标记来记录单个文件的版本。**它建议`@version`标签使用 SCCS 字符串“`%I%, %G%`”。然后，当我们签出文件时，SCCS 将用文件的当前版本替换“`%I%`”，用日期“mm/dd/yy”替换“`%G%`”。例如，它看起来像“1.39，02/28/97”(mm/DD/YY)。此外，每当我们`edit`和`delget` (delta + get)一个文件时，`%I%`就会递增。

SCCS 也被称为源代码控制系统。如果我们想了解更多关于 SCCS 的命令，我们可以参考这里的。此外，SCCS 是一个老式的源代码版本控制系统。

**目前我们倾向于用`@version`标签来表示整个软件的版本。**鉴于这一点，它使`@version`标签不必要地放在一个文件中。

是否意味着单个文件的版本不再重要？那实际上不是真的。现在，我们有现代化的版本控制软件，如 Git、SVN、CVS 等等。每个版本控制软件都有自己独特的方式记录每个文件的版本，不需要依赖于`@version`标签。

让我们以甲骨文 JDK 8 为例。如果我们查看`src.zip`文件中的源代码，我们可能会发现只有`java.awt.Color`类有一个`@version`标签:

```java
/**
 * @version     10 Feb 1997
 */
```

因此，我们可以推断使用`@version`标签来指示单个文件的版本正在消失。因此， [Oracle 文档](https://web.archive.org/web/20220628123130/https://docs.oracle.com/en/java/javase/11/docs/api/jdk.javadoc/com/sun/javadoc/package-summary.html)建议我们使用`@version`标签来记录软件的当前版本号。

### 2.2.`@since`

`@since`标签的格式非常简单:

```java
@since  since-text
```

例如，我们可以用它来标记 JDK 1.7 中引入的一个功能:

```java
/**
 * @since JDK 1.7
 */
```

简而言之，**我们使用`@since`标签来描述一个变化或特性首次出现的时间。**同样，它使用的是整个软件的发布版本，而不是单个文件的版本。[甲骨文文档](https://web.archive.org/web/20220628123130/https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html#@since)给了我们一些关于如何使用`@since`标签的详细说明:

*   当引入一个新的包时，我们应该在包描述和它的每个类中指定一个`@since`标签。
*   当添加一个新的类或接口时，我们应该在类描述中指定一个`@since`标签，而不是在类成员的描述中。
*   如果我们向现有的类添加新成员，我们应该只为新添加的成员指定`@since`标记，而不是在类描述中。
*   如果我们在以后的版本中把一个类成员从`protected`改为`public`，我们不应该改变`@since`标签。

有时候,`@since`标签相当重要，因为它提供了一个重要的提示，即软件用户应该只期待某个特定发布版本之后的特定特性。

如果我们再看一下`src.zip`文件，我们可能会发现很多`@since`标签的用法。让我们以`java.lang.FunctionalInterface`班为例:

```java
/**
 * @since 1.8
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface {}
```

从这段代码片段中，我们可以了解到`FunctionalInterface`类只在 JDK 8 及以上版本中可用。

## 3。`@version`与`@since`和的相似之处

在这一节中，让我们看看`@version`和`@since`标签之间的相似之处。

### 3.1.两者都属于块标签

**首先，`@version`和`@since`都属于 block 标签。**

在文档注释中，标签可以分为两种类型:

*   阻止标签
*   内嵌标签

块标签的形式为`@tag`。并且它应该出现在行首，忽略前导星号、空格和分隔符(`/**`)。例如，我们可以在标签部分使用`@version`和`@since`:

```java
/**
 * Some description here.
 * 
 * @version 1.2
 * @since 1.1
 */
```

然而，内联标签的形式是`{@tag}`。它可以存在于描述或注释中的任何地方。例如，如果我们有一个`{@link}`标签，我们可以在描述中使用它:

```java
/**
 * We can use a {@link java.lang.StringBuilder} class here.
 */
```

### 3.2.两者都可以多次使用

**其次，`@version`和`@since`都可以多次使用。**起初，我们可能会对这种用法感到震惊。然后，我们可能想知道`@version`标签怎么会在一个类中出现多次。但这是真的，在这里有记载[。它解释了我们可以在多个 API 中使用同一个程序元素。因此，我们可以用相同的程序元素附加不同的版本。](https://web.archive.org/web/20220628123130/https://docs.oracle.com/javase/1.5.0/docs/tooldocs/windows/javadoc.html#javadoctags)

例如，如果我们在 ADK 和 JDK 的不同版本中使用相同的类或接口，我们可以提供不同的`@version`和`@since`消息:

```java
/**
 * Some description here.
 *
 * @version ADK 1.6
 * @version JDK 1.7
 * @since ADK 1.3
 * @since JDK 1.4
 */
```

在生成的 HTML 页面中，Javadoc 工具将在名称之间插入逗号(，)和空格。因此，版本文本如下所示:

```java
ADK 1.6, JDK 1.7
```

并且，自文本看起来像:

```java
ADK 1.3, JDK 1.4
```

## 4.`@version`和`@since`的区别

在这一节中，让我们看看`@version`和`@since`标签之间的区别。

### 4.1.它们的内容是否改变

**`@version`文字是不断变化的，`@since`文字是稳定的。**随着时间的推移，软件在不断进化。新功能会加入，所以它的版本会不断变化。然而，`@since`标签仅仅标识了过去新的变化或特性出现的时间点。

### 4.2.它们可以被使用的地方

这两个标签的用法略有不同:

*   `@version`:概述、包、类、接口
*   `@since`:概述，包，类，接口，字段，构造函数，方法

**`@since`标签的使用范围更广，在任何单据注释**中都有效。相比之下，`@version`标签的使用范围更窄，我们不能在字段、构造函数或方法中使用它。

### 4.3。它们是否默认出现

默认情况下，这两个标签在生成的 HTML 页面中具有不同的行为:

*   默认情况下，`@version`文本不显示
*   默认情况下会出现`@since`文本

如果我们希望**在生成的文档中包含“版本文本”，我们可以使用`-version`选项**:

```java
javadoc -version -d docs/ src/*.java
```

同样，如果我们想在生成的文档中省略“自文本”,我们可以使用`-nosince`选项:

```java
javadoc -nosince -d docs/ src/*.java
```

## 5.结论

在本教程中，我们首先讨论了如何正确使用`@version`和`@since`标签。然后我们描述了它们之间的异同。简而言之，`@version`标签保存了软件的当前版本号，`@since`标签描述了一个变化或特性首次出现的时间。