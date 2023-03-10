# 在 Javadoc 注释中引用方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-method-in-javadoc>

## 1.介绍

在本教程中，我们将讨论如何在 [Javadoc 注释](/web/20220821134523/https://www.baeldung.com/javadoc)中引用 Java 方法。此外，我们将讨论如何在不同的类和包中引用方法。

## 2.`@link`标签

**Javadoc 提供了 [`@link`](/web/20220821134523/https://www.baeldung.com/javadoc) 内联标签，用于引用 Java 类**中的成员。我们可以认为`@link`标签类似于 HTML 中的[锚标签](https://web.archive.org/web/20220821134523/https://www.w3schools.com/html/html_links.asp)，用于通过超链接将一个页面链接到另一个页面。

让我们看看在 Javadoc 注释中使用`@link`标签引用方法的语法:

```java
{@link path_to_member label}
```

类似于锚标记，`path_to_member`是目的地，`label`是显示文本。

`label` 是可选的，但是`path_to_member`需要引用一个方法。但是，最好总是使用标签名称来避免复杂的引用链接。`path_to_member`的语法根据我们引用的方法是否在同一个类中而不同。

需要注意的是**在左花括号`{`和`@link`之间不能有空格。**如果它们之间有空格，Javadoc 工具不会正确地生成引用。但是，`path_to_member`、`label`和右花括号之间没有空格限制。

## 3.引用同一类中的方法

引用方法最简单的方式是在同一个类中:

```java
{@link #methodName() LabelName}
```

假设我们正在记录一个方法，我们想引用同一个类中的另一个方法:

```java
/**
 * Also, check the {@link #move() Move} method for more movement details.
 */
public void walk() {
}

public void move() {
}
```

在这种情况下，`walk()` 方法引用同一个类中的`move()`实例方法。

如果被引用的方法有参数，**每当我们想要引用一个重载的或者参数化的方法**时，我们必须相应地指定它的参数类型。

考虑以下引用重载方法的示例:

```java
/**
 * Check this {@link #move(String) Move} method for direction-oriented movement.
 */
public void move() {

}

public void move(String direction) {

}
```

`move()`方法引用了一个带一个`String`参数的重载方法。

## 4.引用另一个类中的方法

要引用另一个类中的方法，我们将使用类名，后跟一个 hashtag，然后是方法名:

```java
{@link ClassName#methodName() LabelName}
```

语法类似于引用同一个类中的方法，除了在 **#** 符号前提到类名。

现在，让我们考虑引用另一个类中的方法的例子:

```java
/**
 * Additionally, check this {@link Animal#run(String) Run} method for direction based run.
 */
public void run() {

}
```

被引用的方法在`Animal`类中，这个类是同一个包中的**:**

```java
public void run(String direction) {

}
```

如果我们想要引用驻留在另一个包中的方法，我们有两个选择。一种方法是**直接指定包以及类名**:

```java
/**
 * Also consider checking {@link com.baeldung.sealed.classes.Vehicle#Vehicle() Vehicle} 
 * constructor to initialize vehicle object.
 */
public void goToWork() {

} 
```

在这种情况下，为了引用`Vehicle()`方法，已经用完整的包名提到了`Vehicle`类。

另外，我们可以**导入包并单独提到类名**:

```java
import com.baeldung.sealed.records.Car;

/**
 * Have a look at {@link Car#getNumberOfSeats() SeatsAvailability} 
 * method for checking the available seats needed for driving.
 */
public void drive() {

}
```

这里，位于另一个包中的`Car`类已经被导入。所以，`@link`只需要包含类名和方法。

我们可以选择两种方式中的任何一种来引用不同包中的方法。如果有一次性使用的包，那么我们可以用第一种方式，否则，如果有多个依赖项，我们应该选择第二种方式。

## 5.`@linkplain`标签

我们已经在注释中看到了用于引用方法的`@link` Javadoc 标签。Javadoc 提供了另一个名为`@linkplain `的标签，用于在注释中引用方法，它类似于`@link`标签。主要区别在于，在生成文档时， **`@link`以等宽格式文本生成标签值，而`@linkplain`以标准格式生成，如纯文本**。

## 6.结论

在本文中，我们讨论了如何在 Javadoc 注释中引用方法，还探讨了在其他类和包中引用方法。最后，我们检查了`@link`和`@linkplain`标签之间的区别。

一如既往，本文的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220821134523/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-4)