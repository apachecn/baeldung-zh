# JVM 中常量池的介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-constant-pool>

## 1.介绍

当我们编译一个`.java`文件时，我们得到一个单独的类文件，扩展名为`.class`。`.class` 文件由几个部分组成，常量池是其中之一。

在这个快速教程中，我们将探索常量池的细节。此外，我们将了解它支持哪些类型以及它如何格式化信息。

## 2.Java 中的常量池

简单地说，常量池包含运行特定类的代码所需的常量。基本上，它是一个类似于符号表的运行时数据结构。它是 Java 类文件中每个类或每个接口的运行时表示。

常量池的内容由编译器生成的符号引用组成。这些引用是从代码中引用的变量、方法、接口和类的名称。JVM 使用它们将代码与它所依赖的其他类链接起来。

让我们使用一个简单的 Java 类来理解常量池的结构:

```java
public class ConstantPool {

    public void sayHello() {
        System.out.println("Hello World");
    }
}
```

要查看常量池的内容，我们需要首先编译文件，然后运行命令:

```java
javap -v name.class
```

上述命令将产生:

```java
 #1 = Methodref          #6.#14         // java/lang/Object."<init>":()V
   #2 = Fieldref           #15.#16        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #17            // Hello World
   #4 = Methodref          #18.#19        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #20            // com/baeldung/jvm/ConstantPool
   #6 = Class              #21            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               sayHello
  #12 = Utf8               SourceFile
  #13 = Utf8               ConstantPool.java
  #14 = NameAndType        #7:#8          // "<init>":()V
  #15 = Class              #22            // java/lang/System
  #16 = NameAndType        #23:#24        // out:Ljava/io/PrintStream;
  #17 = Utf8               Hello World
  #18 = Class              #25            // java/io/PrintStream
  #19 = NameAndType        #26:#27        // println:(Ljava/lang/String;)V
  #20 = Utf8               com/baeldung/jvm/ConstantPool
  #21 = Utf8               java/lang/Object
  #22 = Utf8               java/lang/System
  #23 = Utf8               out
  #24 = Utf8               Ljava/io/PrintStream;
  #25 = Utf8               java/io/PrintStream
  #26 = Utf8               println
  #27 = Utf8               (Ljava/lang/String;)V 
```

**`#n`表示对常量池的引用。** #17 是对`“Hello World” String`的符号引用，#18 是`System.out`，#19 是`println.` 类似的`,` ，#8 强调方法的返回类型是`void`，#20 是完全限定的类名。

需要注意的是，常量池表从索引 1 开始。索引值 0 被视为无效索引。

### 2.1.类型

常量池支持几种类型:

*   `Integer`、`Float`:32 位常量
*   `Double`、`Long`:64 位常量
*   `String`:16 位字符串常量，指向包含实际字节的池中的另一个条目
*   `Class`:包含全限定类名
*   `Utf8`:字节流
*   `NameAndType`:冒号分隔的一对值，第一项表示名称，第二项表示类型
*   `Fieldref`、`Methodref`、`InterfaceMethodref`:一对用点分隔的值，第一个值指向`Class`条目，而第二个值指向`NameAndType`条目

其他类型如`boolean`、`short`和`byte`呢？这些类型在池中被表示为`Integer`常量。

### 2.2.格式

表中的每个条目都遵循一种通用格式:

```java
cp_info {
    u1 tag;
    u1 info[];
}
```

最初的 1 字节标签表示常量的种类。一旦 JVM 抓取并拦截了标签，它就知道标签后面是什么。通常，标签后面是两个或更多的字节来携带关于那个常数的信息。

让我们看看一些类型及其标记索引:

*   `Utf8` : 1
*   `Integer` : 3
*   `Float` : 4
*   `Long` : 5
*   `Double` : 6
*   类别参考:7
*   参考文献:8

任何类或接口的常量池只有在 JVM 完成加载时才被创建。

## 3.结论

在这篇简短的文章中，我们了解了 JVM 中的常量池。我们已经看到它包含了用于定位实际对象的符号引用。此外，我们看一下池如何格式化关于常量及其类型的信息。

和往常一样，代码片段可以在 Github 上找到[。](https://web.archive.org/web/20221202143840/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm-2)