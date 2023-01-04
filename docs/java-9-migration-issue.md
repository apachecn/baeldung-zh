# Java 9 迁移问题和解决方案

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-migration-issue>

## 1.概观

Java 平台曾经有一个单一的架构，将所有的包捆绑成一个单元。

在 Java 9 中，随着 [Java 平台模块系统(JPMS)](/web/20221205121243/https://www.baeldung.com/java-9-modularity) ，或简称为`Modules`的引入，这一点得到了简化。相关的包被分组到模块下，**模块取代包成为复用的基本单元**。

在这个快速教程中，我们将讨论一些与模块相关的问题，当**将现有的应用程序迁移到 Java 9** 时，我们可能会遇到这些问题。

## 2.简单的例子

让我们来看一个简单的 Java 8 应用程序，它包含四个方法，这些方法在 Java 8 下是有效的，但在未来的版本中很有挑战性。我们将使用这些方法来理解迁移到 Java 9 的影响。

第一个方法**获取应用程序中引用的 JCE 提供者**的名称:

```java
private static void getCrytpographyProviderName() {
    LOGGER.info("1\. JCE Provider Name: {}\n", new SunJCE().getName());
}
```

第二种方法在堆栈跟踪中列出类的**名:**

```java
private static void getCallStackClassNames() {
    StringBuffer sbStack = new StringBuffer();
    int i = 0;
    Class<?> caller = Reflection.getCallerClass(i++);
    do {
        sbStack.append(i + ".").append(caller.getName())
            .append("\n");
        caller = Reflection.getCallerClass(i++);
    } while (caller != null);
    LOGGER.info("2\. Call Stack:\n{}", sbStack);
}
```

第三种方法**将 Java 对象转换成 XML** :

```java
private static void getXmlFromObject(Book book) throws JAXBException {
    Marshaller marshallerObj = JAXBContext.newInstance(Book.class).createMarshaller();
    marshallerObj.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);

    StringWriter sw = new StringWriter();
    marshallerObj.marshal(book, sw);
    LOGGER.info("3\. Xml for Book object:\n{}", sw);
}
```

最后一个方法**使用`sun.misc.BASE64Encoder`将一个字符串编码为 64 进制，来自 JDK 内部库**:

```java
private static void getBase64EncodedString(String inputString) {
    String encodedString = new BASE64Encoder().encode(inputString.getBytes());
    LOGGER.info("4\. Base Encoded String: {}", encodedString);
}
```

让我们调用 main 方法中的所有方法:

```java
public static void main(String[] args) throws Exception {
    getCrytpographyProviderName();
    getCallStackClassNames();
    getXmlFromObject(new Book(100, "Java Modules Architecture"));
    getBase64EncodedString("Java");
}
```

当我们在 Java 8 中运行这个应用程序时，我们得到如下结果:

```java
> java -jar target\pre-jpms.jar
[INFO] 1\. JCE Provider Name: SunJCE

[INFO] 2\. Call Stack:
1.sun.reflect.Reflection
2.com.baeldung.prejpms.App
3.com.baeldung.prejpms.App

[INFO] 3\. Xml for Book object:
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<book id="100">
    <title>Java Modules Architecture</title>
</book>

[INFO] 4\. Base Encoded String: SmF2YQ==
```

通常，Java 版本保证向后兼容，但是`JPMS`改变了这一点。

## 3.Java 9 中的执行

现在，让我们用 Java 9 运行这个应用程序:

```java
>java -jar target\pre-jpms.jar
[INFO] 1\. JCE Provider Name: SunJCE

[INFO] 2\. Call Stack:
1.sun.reflect.Reflection
2.com.baeldung.prejpms.App
3.com.baeldung.prejpms.App

[ERROR] java.lang.NoClassDefFoundError: javax/xml/bind/JAXBContext
[ERROR] java.lang.NoClassDefFoundError: sun/misc/BASE64Encoder
```

我们可以看到，前两种方法运行良好，而后两种方法失败了。**让我们通过分析我们的应用**的依赖性来调查失败的原因。我们将使用 Java 9 附带的`jdeps`工具:

```java
>jdeps target\pre-jpms.jar
   com.baeldung.prejpms            -> com.sun.crypto.provider               JDK internal API (java.base)
   com.baeldung.prejpms            -> java.io                               java.base
   com.baeldung.prejpms            -> java.lang                             java.base
   com.baeldung.prejpms            -> javax.xml.bind                        java.xml.bind
   com.baeldung.prejpms            -> javax.xml.bind.annotation             java.xml.bind
   com.baeldung.prejpms            -> org.slf4j                             not found
   com.baeldung.prejpms            -> sun.misc                              JDK internal API (JDK removed internal API)
   com.baeldung.prejpms            -> sun.reflect                           JDK internal API (jdk.unsupported)
```

该命令的输出给出了:

*   第一列是我们的应用程序中所有包的列表
*   第二列是应用程序中所有依赖项的列表
*   Java 9 平台中依赖项的位置–**这可以是模块名，或者是内部 JDK API，**或者是第三方库的无

## 4.不推荐使用的模块

现在让我们试着解决第一个错误 `java.lang.NoClassDefFoundError: javax/xml/bind/JAXBContext.`

根据依赖列表，我们知道`java.xml.bind`包属于似乎是有效模块的`java.xml.bind module `。所以，让我们来看看这个模块的[官方文档。](https://web.archive.org/web/20221205121243/https://docs.oracle.com/javase/9/docs/api/java.xml.bind-summary.html)

官方文档称`java.xml.bind`模块不赞成在未来版本中删除。因此，默认情况下，该模块不会加载到类路径中。

然而， **Java 通过使用`–add-modules`选项提供了按需加载模块**的方法。所以，让我们来试试吧:

```java
>java --add-modules java.xml.bind -jar target\pre-jpms.jar
...
INFO 3\. Xml for Book object:
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<book id="100">
    <title>Java Modules Architecture</title>
</book>
...
```

我们可以看到执行是成功的。这种解决方案快速简单，但不是最佳解决方案。

作为长期解决方案，我们应该使用 Maven 添加[依赖项](https://web.archive.org/web/20221205121243/https://search.maven.org/search?q=g:javax.xml.bind%20AND%20a:jaxb-api&core=gav)作为第三方库:

```java
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.1</version>
</dependency>
```

## 5.JDK 内部 API

现在让我们来看看第二个错误`java.lang.NoClassDefFoundError: sun/misc/BASE64Encoder.`

从依赖列表中，我们可以看到`sun.misc`包是一个`JDK internal` API。

顾名思义，内部 API 是私有代码，在 JDK 内部使用。

在我们的示例中，**内部 API 似乎已经从 JDK** `.`中移除。让我们通过使用`–jdk-internals`选项来检查替代 API 是什么:

```java
>jdeps --jdk-internals target\pre-jpms.jar
...
JDK Internal API                         Suggested Replacement
----------------                         ---------------------
com.sun.crypto.provider.SunJCE           Use java.security.Security.getProvider(provider-name) @since 1.3
sun.misc.BASE64Encoder                   Use java.util.Base64 @since 1.8
sun.reflect.Reflection                   Use java.lang.StackWalker @since 9
```

我们可以看到，Java 9 推荐使用`java.util.Base64`而不是`sun.misc.Base64Encoder.`,因此，为了让我们的应用程序在 Java 9 中运行，代码更改是强制性的。

请注意，我们在应用程序中使用了另外两个内部 API，Java 平台已经建议替换它们，但是我们没有收到任何错误:

*   一些像`sun.reflect.Reflection` **这样的内部 API 被认为对平台至关重要，因此被添加到 JDK 特有的** `jdk.unsupported`模块中。默认情况下，这个模块在 Java 9 的类路径中是可用的。
*   **像`com.sun.crypto.provider.SunJCE`这样的内部 API 只在某些 Java 实现上提供。**只要使用它们的代码运行在同一个实现上，它就不会抛出任何错误。

在本例中的所有情况下**，我们使用内部 API，这不是推荐的做法**。因此，长期的解决方案是用平台提供的合适的公共 API 替换它们。

## 6.结论

在本文中，我们看到了在 **Java 9 中引入的模块系统如何给一些使用废弃或内部 API**的旧应用程序带来迁移问题。

我们还看到了如何对这些错误进行短期和长期修复。

和往常一样，GitHub 上的[提供了这篇文章中的例子。](https://web.archive.org/web/20221205121243/https://github.com/eugenp/tutorials/tree/master/core-java-modules/pre-jpms)