# Java 9——探索 REPL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-repl>

## 1。简介

这篇文章是关于`jshell`，一个交互式`REPL`(读取-评估-打印-循环)控制台，它与即将发布的 Java 9 版本的 JDK 捆绑在一起。对于那些不熟悉这个概念的人来说，REPL 允许交互式地运行任意代码片段并评估它们的结果。

REPL 可以用于快速检查一个想法的可行性，或者计算出`String`或`SimpleDateFormat`的格式化字符串。

## 2。正在运行

首先，我们需要运行 REPL，这是通过调用:

```
$JAVA_HOME/bin/jshell
```

如果需要来自 shell 的更详细的消息，可以使用一个`-v`标志:

```
$JAVA_HOME/bin/jshell -v
```

一旦它准备好了，我们将会收到一条友好的消息，并在底部看到一个熟悉的 Unix 风格的提示。

## 3。定义和调用方法

可以通过键入方法的签名和主体来添加方法:

```
jshell> void helloWorld() { System.out.println("Hello world");}
|  created method helloWorld()
```

这里我们定义了无处不在的“hello world”方法。可以使用普通的 Java 语法调用它:

```
jshell> helloWorld()
Hello world
```

## 4。变量

变量可以用普通的 Java 声明语法来定义:

```
jshell> int i = 0;
i ==> 0
|  created variable i : int

jshell> String company = "Baeldung"
company ==> "Baeldung"
|  created variable company : String

jshell> Date date = new Date()
date ==> Sun Feb 26 06:30:16 EST 2017
|  created variable date : Date
```

注意分号是可选的。变量也可以在没有初始化的情况下声明:

```
jshell> File file
file ==> null
|  created variable file : File
```

## 5。表情

任何有效的 Java 表达式都被接受，求值结果将被显示。如果没有提供结果的显式接收者，将创建“临时”变量:

```
jshell> String.format("%d of bottles of beer", 100)
$6 ==> "100 of bottles of beer"
|  created scratch variable $6 : String
```

REPL 在这里很有帮助，它告诉我们它创建了一个名为`$6`的临时变量，其值为“墙上的 100 瓶啤酒”，类型为`String`。

多行表达式也是可能的。 **`Jshell`足够聪明，知道表达式何时不完整，并会提示用户在新的一行继续:**

```
jshell> int i =
   ...> 5;
i ==> 5
|  modified variable i : int
|    update overwrote variable i : int
```

注意提示是如何变成缩进的`…>`来表示表达式的继续。

## 6。命令

Jshell 提供了很多与评估 Java 语句无关的元命令。它们都以正斜杠(/)开头，以区别于正常操作。例如，我们可以通过发出`/help`或/，请求所有可用命令的列表。。

让我们来看看其中的一些。

### 6.1。进口

要列出当前会话中所有活动的导入，我们可以使用`/import`命令:

```
jshell> /import
|    import java.io.*
|    import java.math.*
|    import java.net.*
|    import java.nio.file.*
|    import java.util.*
|    import java.util.concurrent.*
|    import java.util.function.*
|    import java.util.prefs.*
|    import java.util.regex.*
|    import java.util.stream.*
```

正如我们所看到的，shell 开始时已经添加了相当多有用的导入。

### 6.2。列表

在 REPL 中工作远不如在我们的指尖拥有一个全功能的 IDE 那么容易:很容易忘记什么变量有什么值，定义了什么方法等等。要检查外壳的状态，我们可以使用`/var`、`/methods`、`/list`或`/history:`

```
jshell> /var
| int i = 0
| String company = "Baeldung"
| Date date = Sun Feb 26 06:30:16 EST 2017
| File file = null
| String $6 = "100 of bottles of beer on the wall"

jshell> /methods
| void helloWorld()

jshell> /list

 1 : void helloWorld() { System.out.println("Hello world");}
 2 : int i = 0;
 3 : String company = "Baeldung";
 4 : Date date = new Date();
 5 : File file;
 6 : String.format("%d of bottles of beer on the wall", 100)

jshell> /history

void helloWorld() { System.out.println("Hello world");}
int i = 0;
String company = "Baeldung"
Date date = new Date()
File file
String.format("%d of bottles of beer on the wall", 100)
/var
/methods
/list
/history 
```

`/list`和`/history`的区别在于，后者除了表达式还显示命令。

### 6.3。保存

要保存表达式历史，可使用`/save`命令:

```
jshell> /save repl.java 
```

这将我们的表达式历史保存到运行`jshell`命令的同一个目录下的`repl.java`中。

### 6.4。正在加载

要加载先前保存的文件，我们可以使用`/open`命令:

```
jshell> /open repl.java 
```

然后，可以通过发出`/var`、`/method`或`/list`来验证加载的会话。

### 6.5。退出

当我们完成这些工作后，`/exit`命令可以终止 shell:

```
jshell> /exit
|  Goodbye
```

再见`jshell`。

## 7。结论

在本文中，我们看了一下 Java 9 REPL。由于 Java 已经存在了 20 多年，也许它来得有点晚。然而，它应该被证明是我们 Java 工具箱中的另一个有价值的工具。