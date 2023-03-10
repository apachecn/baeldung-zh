# nudge4j 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/nudge4j>

## 1。概述

`[nudge4j](https://web.archive.org/web/20220628085226/https://lorenzoongithub.github.io/nudge4j/)`允许开发人员直接看到任何操作的影响，并提供一个环境，让他们可以探索、学习并最终花更少的时间调试和重新部署他们的应用程序。

在本文中，我们将探索什么是`nudge4j`,它是如何工作的，以及开发中的任何 Java 应用程序如何从中受益。

## 2。如何运作

### 2.1。伪装的 REPL

`nudge4j`本质上是一个读取-评估-打印循环(REPL ),其中**通过一个仅包含两个元素的简单页面在浏览器窗口**中与您的 Java 应用程序对话:

*   一个编辑
*   `Execute on JVM`按钮

[![nudge4j.in_.action](img/92d3bbacaddb10254d91a085a40faac1.png)](/web/20220628085226/https://www.baeldung.com/wp-content/uploads/2017/02/nudge4j.in_.action.499x439.png)

您可以在典型的 REPL 循环中与您的 JVM 对话:

*   在编辑器中键入任意代码，然后按`Execute on JVM`
*   浏览器将代码发送到 JVM，然后 JVM 运行代码
*   结果被返回(作为字符串)并显示在按钮下方

有几个例子可以直接尝试，比如查询 JVM 已经运行了多长时间，以及当前有多少可用内存。我建议您在编写自己的代码之前从这些开始。

### 2.2。JavaScript 引擎

浏览器发送给 JVM 的代码是操纵 Java 对象的 JavaScript(不要与浏览器上运行的任何 JavaScript 混淆)。JavaScript 由内置的 JavaScript 引擎 [`Nashorn`](https://web.archive.org/web/20220628085226/https://openjdk.java.net/projects/nashorn/) 执行。

如果你不知道(或不喜欢)JavaScript，也不要担心——对于你的`nudge4j`需求，你可以把它当成一种非类型化的 Java 方言。

注意，我意识到说`“JavaScript is untyped Java”`是一个巨大的简化。但是我希望 Java 开发者(他们可能对 JavaScript 有偏见)给`nudge4j`一个公平的机会。

### 2.3。JVM 交互的范围

`nudge4j`让你**访问任何可以从你的 JVM** 访问的 Java 类，允许你调用方法，创建对象等等。这非常强大，但是在使用您的应用程序时可能还不够。

在某些情况下，您可能希望访问一个或多个特定于您的应用程序的对象，以便您可以操作它们。考虑到这一点。任何需要公开的对象都可以在实例化时作为参数传递。

### 2.4。异常处理

`nudge4j`的设计认识到工具用户可能会在 JVM 上犯错误或导致错误。在这两种情况下，该工具都被设计为报告完整的堆栈跟踪，以便指导用户纠正错误。

让我们来看一个屏幕截图，其中一段已经执行的代码导致了一个异常被抛出:

## [![nudge4j.exception](img/4f3bde9db0cac1bb3edb4cc9469e425a.png)](/web/20220628085226/https://www.baeldung.com/wp-content/uploads/2017/02/nudge4j.exception.488x711.png)

## 3。将`nudge4j`添加到您的应用程序中

### 3.1。只需复制粘贴

与`nudge4j`的集成有点不寻常，因为没有`jar`文件要添加到您的类路径中，也没有依赖项要添加到 Maven 或 Gradle 构建中。

取而代之的是，在运行之前，你需要简单地复制并粘贴一小段 Java 代码(大约 100 行)到你自己的代码中。

你可以在 [`nudge4j`主页](https://web.archive.org/web/20220628085226/https://lorenzoongithub.github.io/nudge4j/)上找到这个片段——页面上甚至有一个按钮，你可以点击它将片段复制到你的剪贴板上。

这段代码一开始可能看起来很深奥。这有几个原因:

*   可以将`nudge4j`片段放到任何类中；因此，它不能对`import` s 做任何假设，它包含的任何类都必须是完全限定的
*   为了避免与已经定义的变量发生潜在冲突，代码被包装在一个函数中
*   对内置 JDK HttpServer 的访问是通过自省完成的，以避免一些 ide(例如 Eclipse)对以`“com.sun.*”`开头的包的限制

因此，即使 Java 已经是一种冗长的语言，也必须让它变得更加冗长，以提供无缝集成。

### 3.2。示例应用程序

让我们从一个标准的 JVM 应用程序开始，我们假设一个简单的`java.util.HashMap`保存了我们想要使用的大部分信息:

```java
public class MyApp {
    public static void main(String args[]) {
        Map map = new HashMap();
        map.put("health", 60);
        map.put("strength", 4);
        map.put("tools", Arrays.asList("hammer"));
        map.put("places", Arrays.asList("savannah","tundra"));
        map.put("location-x", -42 );
        map.put("location-y", 32);

        // paste original code from nudge4j below
        (new java.util.function.Consumer<Object[]>() {
            public void accept(Object args[]) {
                ...
                ...
            }
        }).accept(new Object[] { 
            5050,  // <-- the port
            map    // <-- the map is passed as a parameter.
        });
    }
}
```

从这个例子中可以看出，您只需将`nudge4j`片段粘贴到您自己代码的末尾。此处示例中的第 12-20 行充当代码片段的缩写版本的占位符。

现在，让我们将浏览器指向`http://localhost:5050/.`现在，只需输入以下命令，就可以在浏览器中以`args[1]` 的形式访问地图:

```java
args[1];
```

这将提供我们的`Map`的摘要(在这种情况下依赖于`Map`的`toString()`方法及其键和值)。

假设我们想要检查并修改键值为`“tools”`的`Map`条目。

要获得 `Map`中所有可用 `tools` 的列表，您应该编写:

```java
map = args[1];
map.get("tools");
```

为了给`Map`添加一个新的`tool`，您应该写:

```java
map = args[1];
map.get("tools").add("axe");
```

一般来说，几行代码就足以探测任何 Java 应用程序。

## 4。结论

通过在 JDK 中组合两个简单的 API(`Nashorn`和`Http server` ) `nudge4j`让您能够探测任何 Java 8 应用程序。

从某种程度上来说，`nudge4j`只是一个旧想法的现代翻版:让开发人员通过脚本语言访问现有系统的设施——这个想法可以对 Java 开发人员如何度过他们的编码日产生影响。