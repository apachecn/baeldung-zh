# JVM 语言概述

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-languages>

## 1。简介

除了 Java，其他语言也可以在 Java 虚拟机上运行，比如 Scala、Kotlin、Groovy、Clojure。

在接下来的几节中，我们将从高层次上看一下最流行的 JVM 语言。

当然，我们将从 JVM 语言的先驱——Java 开始。

## 2。Java

### 2.1。概述

Java 是一种通用编程语言，采用了面向对象的范例。

该语言的一个核心特点是跨平台的可移植性，这意味着在一个平台上编写的程序可以在任何软件和硬件的组合上执行，并有足够的运行时支持。这是通过首先将代码编译成字节码来实现的，而不是直接编译成特定于平台的机器码。

Java 字节码指令类似于机器码，但是它们由特定于主机操作系统和硬件组合的 Java 虚拟机(JVM)来解释。

虽然 Java 最初是面向对象的语言，但它已经开始采用来自其他编程范例的概念，如[函数式编程](/web/20221123135200/https://www.baeldung.com/cs/functional-programming)。

让我们快速看一下 Java 的一些主要特性:

*   面向对象
*   强[静态类型](/web/20221123135200/https://www.baeldung.com/cs/statically-vs-dynamically-typed-languages)
*   独立于平台
*   垃圾收集的
*   多线程

### 2.2。示例

让我们看看一句简单的“你好，世界！”示例如下:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

在这个例子中，我们创建了一个名为`HelloWorld`的类，并定义了在控制台上打印消息的 main 方法。

接下来，**我们将使用`javac`命令生成可以在 JVM 上执行的字节码**:

```java
javac HelloWorld.java
```

最后，**`java`命令在 JVM 上执行生成的字节码:**

```java
java HelloWorld
```

更多 Java 示例，请查看我们的教程列表。

## 3。Scala

### 3.1。概述

[Scala](https://web.archive.org/web/20221123135200/https://www.scala-lang.org/) 代表“可扩展语言”。 **Scala 是一种静态类型语言，它结合了两种重要的编程范式，即面向对象和函数式编程。**

这种语言起源于 2004 年，但近年来变得更加流行。

Scala 是一种纯面向对象的语言，因为它不支持原语。Scala 提供了定义类、对象、方法以及函数式编程特性的能力，比如特征、代数数据类型或类型类。

Scala 的几个重要特性是:

*   功能性、面向对象
*   强静态类型
*   代数数据类型
*   模式匹配
*   增强的不变性支持
*   懒惰计算
*   多线程

### 3.2。示例

首先，我们来看看同样的“你好，世界！”和前面一样，这次是在 Scala 中:

```java
object HelloWorld {
    def main(args: Array[String]): Unit = println("Hello, world!")
}
```

在这个例子中，我们创建了一个名为`HelloWorld`的单例对象和`main`方法。

接下来，为了编译它，我们可以使用`scalac`:

```java
scalac HelloWorld.scala
```

`scala`命令在 JVM 上执行生成的字节码:

```java
scala HelloWorld
```

## 4.**锅炉**

### 4.1。概述

**[Kotlin](https://web.archive.org/web/20221123135200/https://kotlinlang.org/) 是一种静态类型的通用开源语言，由 [JetBrains](https://web.archive.org/web/20221123135200/https://en.wikipedia.org/wiki/JetBrains) 团队**开发，它将面向对象和函数式范例结合在一起。

开发 Kotlin 的主要焦点是 Java 互操作性、安全性(异常处理)、简洁性和更好的工具支持。

自从 Android Studio 3.0 发布以来，Kotlin 就是 Google 在 Android 平台上全面支持的编程语言。它也包含在 Android Studio IDE 包中，作为标准 Java 编译器的替代。

Kotlin 的一些重要特性:

*   面向对象+函数式
*   强静态类型
*   简明的
*   与 Java 互操作

我们对 Kotlin 的[介绍也包含了更多关于特性的细节。](/web/20221123135200/https://www.baeldung.com/kotlin)

### 4.2。示例

让我们看看“你好，世界！”科特林的例子:

```java
fun main(args: Array<String>) { println("Hello, World!") }
```

我们可以将上面的代码写在一个名为`helloWorld.kt.`的新文件中

然后，**我们将使用`kotlinc`命令来编译这个**，并生成可以在 JVM 上执行的字节码:

```java
kotlinc helloWorld.kt -include-runtime -d helloWorld.jar
```

`-d`选项用于指示输出文件为`class`文件或一个`.jar`文件名。**`-include-runtime`选项通过在生成的`.jar`文件中包含 Kotlin 运行时库，使其自包含并可运行。**

然后，`java`命令在 JVM 上执行生成的字节码:

```java
java -jar helloWorld.jar
```

让我们看看另一个使用`for`循环打印项目列表的例子:

```java
fun main(args: Array<String>) {
    val items = listOf(1, 2, 3, 4)
    for (i in items) println(i)
}
```

## 5。Groovy

### 5.1。概述

[**Groovy**](https://web.archive.org/web/20221123135200/http://groovy-lang.org/) **是一种面向对象的、可选类型的动态领域特定语言(DSL)** ，支持静态类型和静态编译功能。它旨在提高开发人员的生产力，语法简单易学。

Groovy 可以轻松地与任何 Java 程序集成，并立即添加强大的功能，如脚本功能、运行时和编译时元编程以及函数式编程功能。

让我们强调几个重要的特性:

*   面向对象，具有高阶函数、currying、闭包等功能特性
*   打字–动态、静态、强、鸭
*   领域特定语言
*   与 Java 的互操作性
*   简洁带来的生产力
*   运算符重载

### 5.2。示例

首先，让我们看看我们的“你好，世界！”Groovy 中的示例:

```java
println("Hello world")
```

我们将上述代码写在一个名为`HelloWorld.groovy`的新文件中。现在**我们可以用两种方式运行这段代码:编译然后执行，或者只运行未编译的代码。**

我们可以使用`groovyc` 命令编译一个`.groovy`文件，如下所示:

```java
groovyc HelloWorld.groovy
```

然后，我们将使用`java`命令来执行 groovy 代码:

```java
java -cp <GROOVY_HOME>\embeddable\groovy-all-<VERSION>.jar;. HelloWorld
```

例如，上面的命令可能类似于:

```java
java -cp C:\utils\groovy-1.8.1\embeddable\groovy-all-1.8.1.jar;. HelloWorld
```

让我们看看如何使用`groovy`命令执行`.groovy`文件而不编译:

```java
groovy HelloWorld.groovy
```

最后，这里是打印带有索引的项目列表的另一个示例:

```java
list = [1, 2, 3, 4, 5, 6, 7, 8, 9]
list.eachWithIndex { it, i -> println "$i: $it"}
```

在我们的[介绍文章](/web/20221123135200/https://www.baeldung.com/groovy-language)中看看更多的 Groovy 例子。

## 6。Clojure

### 6.1。概述

[**Clojure**](https://web.archive.org/web/20221123135200/https://clojure.org/) **是一种通用函数式编程语言。这种语言运行在 JVM 和微软的公共语言运行时上。Clojure 仍然是一种编译语言，它仍然是动态的，因为它的特性在运行时得到支持。**

Clojure 的设计者想要设计现代的可以在 JVM 上运行的 Lisp。这就是为什么它也被称为 Lisp 编程语言的一种方言。**与 Lisps 类似，Clojure 将代码视为数据，也有一个宏系统。**

一些重要的 Clojure 特性:

*   功能的
*   打字-动态，强大，最近开始支持[渐进打字](https://web.archive.org/web/20221123135200/https://en.wikipedia.org/wiki/Gradual_typing)
*   专为并发性设计
*   运行时多态性

### 6.2。示例

与其他 JVM 语言不同，创建简单的“Hello，World！”Clojure 中的程序。

我们将使用 [Leiningen](https://web.archive.org/web/20221123135200/https://leiningen.org/) 工具来运行我们的示例。

首先，我们将使用以下命令创建一个带有默认模板的简单项目:

```java
lein new hello-world
```

将使用以下文件结构创建项目:

```java
./project.clj
./src
./src/hello-world
./src/hello-world/core.clj
```

现在我们需要用以下内容更新`./project.ctj`文件来设置主源文件:

```java
(defproject hello-world "0.1.0-SNAPSHOT"
  :main hello-world.core
  :dependencies [[org.clojure/clojure "1.5.1"]])
```

现在，我们将更新代码，打印“Hello，World！”在`./src/hello-world/core.` clj 文件中:

```java
(ns hello-world.core)

(defn -main [& args]
    (println "Hello, World!"))
```

最后，在移动到项目的根目录后，我们将使用`lein`命令来执行上面的代码:

```java
cd hello-world
lein run
```

## 7.其他 JVM 语言

### 7.1。Jython

[**Jython**](https://web.archive.org/web/20221123135200/http://www.jython.org/) **是运行在 JVM 上的 [Python](https://web.archive.org/web/20221123135200/https://www.python.org/)** 的 Java 平台实现。

这种语言最初被设计用来编写高性能的应用程序而不牺牲交互性。Jython 是面向对象的、多线程的，并使用 Java 的垃圾收集器来有效地清理内存。

Jython 包含了 Python 语言的大部分模块。它还可以导入和使用 Java 库中的任何类。

让我们看一个快速的“你好，世界！”示例:

```java
print "Hello, world!"
```

### 7.2。JRuby

**[JRuby](https://web.archive.org/web/20221123135200/http://www.jruby.org/) 是运行在 Java 虚拟机上的 [Ruby](https://web.archive.org/web/20221123135200/https://www.ruby-lang.org/en/) 编程语言的实现。**

JRuby 语言是高性能和多线程的，有大量来自 Java 和 Ruby 的可用库。此外，它结合了两种语言的特性，如面向对象编程和鸭子打字。

让我们印上“你好，世界！”在 JRuby 中:

```java
require "java"

stringHello= "Hello World"
puts "#{stringHello.to_s}"
```

## 8。结论

在本文中，我们研究了许多流行的 JVM 语言和基本代码示例。这些语言实现了各种编程范例，如面向对象、函数式、静态类型、动态类型。

到目前为止，它表明即使 JVM 可以追溯到 1995 年，它仍然是现代编程语言的高度相关和引人注目的平台。