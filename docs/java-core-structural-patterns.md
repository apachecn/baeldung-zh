# 核心 Java 中的结构模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-core-structural-patterns>

## 1.概观

**结构[设计模式](/web/20220524131130/https://www.baeldung.com/design-patterns-series)是那些通过识别大型对象结构**之间的关系来简化它们的设计的模式。它们描述了组成类和对象的常见方法，以便它们可以作为解决方案重复使用。

四人组描述了七种这样的结构方式或模式。在这个快速教程中，**我们将会看到一些 Java 核心库是如何采用它们的例子**。

## 2.[适配器](/web/20220524131130/https://www.baeldung.com/java-adapter-pattern)

顾名思义，**适配器充当中介，将原本不兼容的接口转换成客户机期望的接口**。

这在我们想要获取一个其源代码不能被修改的现有类并使它与另一个类一起工作的情况下是有用的。

JDK 的集合框架提供了许多适配器模式的例子:

```java
List<String> musketeers = Arrays.asList("Athos", "Aramis", "Porthos");
```

在这里， **[`Arrays#asList`](/web/20220524131130/https://www.baeldung.com/java-arrays-aslist-vs-new-arraylist) 是在帮我们把一个`Array`改编成一个`List`** 。

I/O 框架也大量使用了这种模式。作为一个例子，让我们考虑这个片段，它将一个 [`InputStream`映射到一个`Reader`](/web/20220524131130/https://www.baeldung.com/java-convert-inputstream-to-reader) 对象:

```java
InputStreamReader input = new InputStreamReader(new FileInputStream("input.txt"));
```

## 3.[桥](/web/20220524131130/https://www.baeldung.com/java-structural-design-patterns#bridge)

桥接模式**允许抽象和实现之间的分离，这样它们可以彼此独立地开发，但仍然有一种方式或桥梁共存和交互**。

Java 中的一个例子是 [JDBC API](/web/20220524131130/https://www.baeldung.com/java-jdbc) 。它充当 Oracle、MySQL 和 PostgreSQL 等数据库与其特定实现之间的链接。

JDBC API 是一组标准接口，例如`Driver`、`Connection`和`ResultSet,`等等。这使得不同的数据库供应商可以有他们各自的实现。

例如，要创建到数据库的连接，我们可以说:

```java
Connection connection = DriverManager.getConnection(url);
```

这里，`url`是一个可以代表任何数据库厂商的字符串。

例如，对于 PostgreSQL，我们可能有:

```java
String url = "jdbc:postgresql://localhost/demo";
```

对于 MySQL:

```java
String url = "jdbc:mysql://localhost/demo";
```

## 4.[复合](/web/20220524131130/https://www.baeldung.com/java-composite-pattern)

这种模式处理对象的树状结构。在这个树中，单个对象，甚至整个层次，都以同样的方式处理。更简单地说，**这种模式以分层的方式安排对象，这样客户就可以无缝地处理整体的一部分**。

AWT/Swing 中的嵌套容器是在核心 Java 中使用复合模式的很好的例子。`java.awt.Container`对象基本上是一个根组件，可以包含其他组件，形成嵌套组件的树形结构。

考虑以下代码片段:

```java
JTabbedPane pane = new JTabbedPane();
pane.addTab("1", new Container());
pane.addTab("2", new JButton());
pane.addTab("3", new JCheckBox());
```

这里使用的所有类，即`JTabbedPane`、`JButton`、`JCheckBox`和`JFrame`，都是`Container`的后代。正如我们所见，**这个代码片段处理第二行中的树根或`Container`，就像处理其子节点**一样。

## 5.[装饰者](/web/20220524131130/https://www.baeldung.com/java-decorator-pattern)

当我们想要增强一个对象的行为而不修改原始对象本身时，这种模式就发挥作用了。这是通过向对象添加相同类型的包装来实现的，以便为其附加额外的责任。

这种模式最普遍的用法之一可以在 [`java.io`](/web/20220524131130/https://www.baeldung.com/java-download-file#using-java-io) 包中找到:

```java
BufferedInputStream bis = new BufferedInputStream(new FileInputStream(new File("test.txt")));
while (bis.available() > 0) {
    char c = (char) bis.read();
    System.out.println("Char: " + c);
}
```

这里， **`BufferedInputStream`对`FileInputStream`进行了修饰，增加了缓冲输入**的能力。值得注意的是，这两个类都有一个共同的祖先`InputStream` 。这意味着装饰的对象和被装饰的对象是同一类型的。这是装饰模式的一个明显标志。

## 6.[立面](/web/20220524131130/https://www.baeldung.com/java-facade-pattern)

根据定义，facade 这个词的意思是一个物体的人造的或虚假的外观。应用于编程，它同样意味着**为一组复杂的对象**提供另一个面——或者说，接口。

当我们想要简化或隐藏子系统或框架的复杂性时，这种模式就发挥作用了。

Faces API 的`ExternalContext`是 facade 模式的一个很好的例子。它在内部使用诸如`HttpServletRequest`、`HttpServletResponse`和`HttpSession`之类的类。基本上，它是一个允许 Faces API 不知道其底层应用程序环境的类。

让我们看看 [`**Primefaces**`](https://web.archive.org/web/20220524131130/https://www.primefaces.org/docs/api/5.3/org/primefaces/component/export/PDFExporter.html#writePDFToResponse(javax.faces.context.ExternalContext,%20java.io.ByteArrayOutputStream,%20java.lang.String)) **如何用它来写一个`HttpResponse`，而实际上并不知道它**:

```java
protected void writePDFToResponse(ExternalContext externalContext, ByteArrayOutputStream baos, String fileName)
  throws IOException, DocumentException {
    externalContext.setResponseContentType("application/pdf");
    externalContext.setResponseHeader("Expires", "0");
    // set more relevant headers
    externalContext.setResponseContentLength(baos.size());
    externalContext.addResponseCookie(
      Constants.DOWNLOAD_COOKIE, "true", Collections.<String, Object>emptyMap());
    OutputStream out = externalContext.getResponseOutputStream();
    baos.writeTo(out);
    // do cleanup
}
```

正如我们在这里看到的，我们直接使用`ExternalContext`作为外观来设置响应头、实际响应和 cookie。**图中的`HTTPResponse`不是**。

## 7.[飞锤](/web/20220524131130/https://www.baeldung.com/java-flyweight)

**flyweight 模式通过回收我们的对象来减轻它们的重量，或者内存占用**。换句话说，如果我们有可以共享状态的不可变对象，按照这种模式，我们可以缓存它们以提高系统性能。

在 Java 的所有 [`Number`](/web/20220524131130/https://www.baeldung.com/java-number-class) 类中都可以看到 Flyweight。

用于创建任何数据类型的包装类的对象的`valueOf`方法被设计用来缓存值并在需要时返回它们。

例如， [`Integer`](https://web.archive.org/web/20220524131130/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html) 有一个静态类，`IntegerCache,`帮助它的 [`valueOf`](/web/20220524131130/https://www.baeldung.com/java-convert-string-to-int-or-integer#integervalueof) 方法总是缓存-128 到 127 范围内的值:

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high) {
        return IntegerCache.cache[i + (-IntegerCache.low)];
    }
    return new Integer(i);
}
```

## 8.[代理](/web/20220524131130/https://www.baeldung.com/java-proxy-pattern)

这个模式为另一个复杂的对象提供了一个代理或者替代品。虽然听起来类似于 facade，但实际上是不同的，因为 facade 为客户端提供了不同的接口来进行交互。在代理的情况下，接口与它隐藏的对象的接口相同。

使用这种模式，在原始对象创建之前或之后对其执行任何操作都变得很容易。

JDK 为代理实现提供了一个现成的 [`java.lang.reflect.Proxy`](/web/20220524131130/https://www.baeldung.com/java-dynamic-proxies) 类:

```java
Foo proxyFoo = (Foo) Proxy.newProxyInstance(Foo.class.getClassLoader(),
  new Class<?>[] { Foo.class }, handler);
```

上面的代码片段为接口`Foo`创建了一个代理`proxyFoo`。

## 9.结论

在这个简短的教程中，**我们看到了在核心 Java** 中实现的结构设计模式的实际应用。

总而言之，我们简要地定义了七种模式的含义，然后通过代码片段逐一理解它们。