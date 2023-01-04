# Java 中的构造函数返回类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-constructor-return-type>

## 1.概观

在这个快速教程中，我们将关注 Java 中构造函数的返回类型。

首先，我们将熟悉 Java 和 JVM 中对象初始化的工作方式。然后，我们将深入了解对象初始化和赋值是如何工作的。

## 2.实例初始化

让我们从一个空类开始:

```java
public class Color {}
```

这里，我们将从该类创建一个实例，并将其赋给某个变量:

```java
Color color = new Color();
```

编译完这个简单的 Java 片段后，让我们通过`javap -c`命令来看看它的字节码:

```java
0: new           #7                  // class Color
3: dup
4: invokespecial #9                  // Method Color."<init>":()V
7: astore_1
```

当我们用 Java 实例化一个对象时，JVM 执行以下操作:

1.  首先，它[在其进程空间中为新对象找到一个位置](https://web.archive.org/web/20220926182252/https://alidg.me/blog/2019/6/21/tlab-jvm)。
2.  然后，JVM 执行系统初始化过程。在这一步中，它以默认状态创建对象。字节码中的`new `操作码实际上负责这一步。
3.  最后，它用构造函数和其他初始化器块初始化对象。在这种情况下，`invokespecial `操作码调用构造函数。

如上所示，默认构造函数的方法签名是:

```java
Method Color."<init>":()V
```

**`<init> `是 JVM** 中[实例初始化方法](https://web.archive.org/web/20220926182252/https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html#jvms-2.9)的名称。在这种情况下，`<init> `是一个函数:

*   不接受任何东西作为输入(方法名后的空括号)
*   return nothing`(V`代表`void`

**所以 Java 和 JVM 中构造函数的返回类型是`void.`**

再看一下我们的简单任务:

```java
Color color = new Color();
```

现在我们知道构造函数返回`void`，让我们看看赋值是如何工作的。

## 3.任务如何工作

JVM 是基于堆栈的虚拟机。每个堆栈由[个堆栈帧](https://web.archive.org/web/20220926182252/https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html#jvms-2.6)组成。简单地说，每个堆栈帧对应一个方法调用。事实上，JVM 用一个新的方法调用创建帧，并在它们完成任务时销毁它们:

[![](img/106aa184ee09572860eb790120cab57f.png)](/web/20220926182252/https://www.baeldung.com/wp-content/uploads/2020/06/simple-ol.svg)

**每个堆栈帧使用一个数组存储局部变量，使用一个操作数堆栈存储部分结果**。考虑到这一点，让我们再来看看字节码:

```java
0: new           #7                // class Color
3: dup
4: invokespecial #9               // Method Color."<init>":()V
7: astore_1
```

任务是这样进行的:

*   `new `指令创建`Color `的实例，并将其引用推送到操作数堆栈上
*   `dup `操作码复制了操作数堆栈上的最后一项
*   `invokespecial `获取复制的引用并将其用于初始化。此后，只有原始引用保留在操作数堆栈中
*   `astore_1 `存储对局部变量数组的索引 1 的原始引用。前缀“a”意味着要存储的项是对象引用，“1”是数组索引

从现在开始，**局部变量数组中的第二项(索引 1)是对新创建的对象**的引用。因此，我们没有丢失引用，赋值实际上是有效的——即使构造函数没有返回任何东西！

## 4.结论

在这个快速教程中，我们学习了 JVM 如何创建和初始化我们的类实例。此外，我们看到了实例初始化是如何工作的。

为了更详细地理解 JVM，查看它的[规范](https://web.archive.org/web/20220926182252/https://docs.oracle.com/javase/specs/jvms/se14/html/index.html)总是一个好主意。