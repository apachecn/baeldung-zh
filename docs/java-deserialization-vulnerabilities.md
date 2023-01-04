# Java 中的反序列化漏洞

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-deserialization-vulnerabilities>

## 1.概观

在本教程中，我们将探讨攻击者如何使用 Java 代码中的反序列化来利用系统。

我们首先来看看攻击者可能利用系统的一些不同方法。然后，我们将看看成功攻击的含义。最后，我们将了解一些有助于避免此类攻击的最佳实践。

## 2.反序列化漏洞

Java 广泛使用反序列化从输入源创建对象。

这些输入源是字节流，有多种格式(一些标准格式包括 JSON 和 XML)。 **L** **使用反序列化来评估系统功能或跨网络与可信来源的通信。**然而，不可信或恶意的字节流可以利用易受攻击的反序列化代码。

我们上一篇关于 [Java 序列化](/web/20220810175722/https://www.baeldung.com/java-serialization)的文章更深入地介绍了序列化和反序列化是如何工作的。

### 2.1.攻击媒介

让我们讨论一下攻击者可能如何使用反序列化来利用系统。

对于一个可序列化的类，它必须符合`[Serializable](https://web.archive.org/web/20220810175722/https://docs.oracle.com/en/java/javase/13/docs/api/java.base/java/io/Serializable.html) `接口。实现`Serializable` 的类使用方法`readObject` 和 `writeObject.`，这些方法分别反序列化和序列化类的对象实例。

这种情况的典型实现可能如下所示:

```java
public class Thing implements Serializable {
    private static final long serialVersionUID = 0L;

    // Class fields

    private void readObject(ObjectInputStream ois) throws ClassNotFoundException, IOException {
        ois.defaultReadObject();
        // Custom attribute setting
    }

    private void writeObject(ObjectOutputStream oos) throws IOException {
        oos.defaultWriteObject(); 
        // Custom attribute getting
    }
}
```

当类具有通用的或松散定义的字段并使用反射来设置这些字段的属性时，类变得易受攻击:

```java
public class BadThing implements Serializable {
    private static final long serialVersionUID = 0L;

    Object looselyDefinedThing;
    String methodName;

    private void readObject(ObjectInputStream ois) throws ClassNotFoundException, IOException {
        ois.defaultReadObject();
        try {
            Method method = looselyDefinedThing.getClass().getMethod(methodName);
            method.invoke(looselyDefinedThing);
        } catch (Exception e) {
            // handle error...
        }
    }

    // ...
}
```

让我们分解以上内容，看看发生了什么。

首先，我们的类`BadThing `有一个类型为`Object.` **的字段`looselyDefinedThing`，这是模糊的，允许攻击者将该字段设置为类路径中可用的任何类型。**

接下来，使这个类易受攻击的是，`readObject`方法包含调用`looselyDefinedThing`上的方法的定制代码。**我们想要调用的方法通过反射使用字段`methodName`(也可以被攻击者控制)。**

如果类`MyCustomAttackObject` 在系统的类路径中，上面的代码在执行时等效于下面的代码:

```java
BadThing badThing = new BadThing();
badThing.looselyDefinedThing = new MyCustomAttackObject();
badThing.methodName = "methodThatTriggersAttack";

Method method = looselyDefinedThing.getClass().getMethod(methodName);
method.invoke(methodName);
```

```java
public class MyCustomAttackObject implements Serializable {
    public static void methodThatTriggersAttack() {
        try {
            Runtime.getRuntime().exec("echo \"Oh, no! I've been hacked\"");
        } catch (IOException e) {
            // handle error...
        }
    }
}
```

通过使用`MyCustomAttackObject`类，攻击者能够在主机上执行命令。

这个特殊的命令是无害的。然而，如果这种方法能够接受自定义命令，那么攻击者能够实现的可能性是无限的。

仍然存在的问题是，“为什么首先会有人在类路径中有这样一个类？”。

允许攻击者执行恶意代码的类广泛存在于许多框架和软件使用的开源和第三方库中。它们通常不像上面的例子那么简单，而是涉及到使用多个类和反射来执行类似的命令。

以这种方式使用多个类通常被称为小工具链。开源工具 [ysoserial](https://web.archive.org/web/20220810175722/https://github.com/frohoff/ysoserial) 维护了一个可用于攻击的小工具链的活动列表。

### 2.2.含义

现在我们知道了攻击者是如何获得远程命令执行权限的，让我们来讨论一下攻击者能够在我们的系统上实现什么的一些含义。

根据运行 JVM 的用户所拥有的访问级别，攻击者可能已经在机器上拥有更高的权限，这将允许他们访问系统中的大多数文件并窃取信息。

一些反序列化漏洞允许攻击者执行自定义 Java 代码，这可能导致拒绝服务攻击、窃取用户会话或未经授权访问资源。

由于每个反序列化漏洞不同，每个系统的设置也不同，因此攻击者能够实现的目标也千差万别。因此，**漏洞数据库认为反序列化漏洞风险高。**

## 3.预防的最佳做法

既然我们已经介绍了我们的系统可能被利用的方式，我们将触及一些可以遵循的最佳实践，以帮助防止这种类型的攻击并限制潜在利用的范围。

**注意，在漏洞利用预防中没有灵丹妙药，本节也不是所有预防措施的详尽列表:**

*   我们应该不断更新开源库。当库的最新版本可用时，优先更新到最新版本。
*   主动检查漏洞数据库，例如国家漏洞数据库(T0)或 T2 CVE 密特漏洞数据库(T3)中新公布的漏洞，确保我们不会暴露
*   验证反序列化的输入字节流的来源(使用安全连接并验证用户等)。)
*   如果输入来自用户输入字段，请确保在反序列化之前验证这些字段并授权用户
*   创建自定义反序列化代码时，请遵循反序列化的 [owasp 备忘单](https://web.archive.org/web/20220810175722/https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html)
*   限制 JVM 在主机上可以访问的内容，以缩小攻击者能够利用我们的系统的范围

## 4.结论

在本文中，我们讨论了攻击者如何利用反序列化来利用易受攻击的系统。此外，我们还介绍了一些在 Java 系统中维护良好安全卫生的实践。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220810175722/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-serialization)