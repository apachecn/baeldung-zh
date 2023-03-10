# JVM 中调用动态的介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-invoke-dynamic>

## 1.概观

Invoke Dynamic(也称为 Indy)是 JSR 292 T2 的一部分，旨在增强 JVM 对动态类型语言的支持。在 Java 7 中首次发布之后， `invokedynamic`操作码被基于 JVM 的动态语言(如 JRuby)甚至静态类型语言(如 Java)广泛使用。

在本教程中，我们将揭开`invokedynamic`的神秘面纱，看看它如何帮助库和语言设计者实现多种形式的动态性。

## 2.会议调用动态

让我们从一个简单的[流 API](/web/20220524114233/https://www.baeldung.com/java-streams) 调用链开始:

```java
public class Main { 

    public static void main(String[] args) {
        long lengthyColors = List.of("Red", "Green", "Blue")
          .stream().filter(c -> c.length() > 3).count();
    }
}
```

**起初，我们可能认为 Java 创建了一个从`Predicate `派生的匿名内部类，然后将该实例传递给`filter `方法。**但是，我们错了。

### 2.1.字节码

为了验证这个假设，我们可以看一下生成的字节码:

```java
javap -c -p Main
// truncated
// class names are simplified for the sake of brevity 
// for instance, Stream is actually java/util/stream/Stream
0: ldc               #7             // String Red
2: ldc               #9             // String Green
4: ldc               #11            // String Blue
6: invokestatic      #13            // InterfaceMethod List.of:(LObject;LObject;)LList;
9: invokeinterface   #19,  1        // InterfaceMethod List.stream:()LStream;
14: invokedynamic    #23,  0        // InvokeDynamic #0:test:()LPredicate;
19: invokeinterface  #27,  2        // InterfaceMethod Stream.filter:(LPredicate;)LStream;
24: invokeinterface  #33,  1        // InterfaceMethod Stream.count:()J
29: lstore_1
30: return
```

不管我们怎么想，**没有匿名的内部类**，当然，没有人会将这样一个类的实例传递给`filter `方法`. `

令人惊讶的是，`invokedynamic`指令负责创建`Predicate `实例。

### 2.2.Lambda 特定方法

此外，Java 编译器还生成了以下有趣的静态方法:

```java
private static boolean lambda$main$0(java.lang.String);
    Code:
       0: aload_0
       1: invokevirtual #37                 // Method java/lang/String.length:()I
       4: iconst_3
       5: if_icmple     12
       8: iconst_1
       9: goto          13
      12: iconst_0
      13: ireturn
```

该方法将一个`String `作为输入，然后执行以下步骤:

*   在`length`上计算输入长度`(invokevirtual`
*   将长度与常数 3 ( `if_icmple `和`iconst_3`)进行比较
*   如果长度小于或等于 3，则返回`false `

有趣的是，这实际上相当于我们传递给`filter `方法的 lambda:

```java
c -> c.length() > 3
```

**因此，Java 创建了一个特殊的静态方法，并通过`invokedynamic. `** 调用该方法，而不是匿名的内部类

在本文的整个过程中，我们将看到这个调用是如何在内部工作的。但是，首先，让我们定义一下`invokedynamic `试图解决的问题。

### 2.3.问题是

在 Java 7 之前，JVM 只有四种方法调用类型:`invokevirtual `调用普通类方法，`invokestatic `调用静态方法，`invokeinterface `调用接口方法，`invokespecial `调用构造函数或私有方法。

尽管它们有所不同，但所有这些调用都有一个简单的特点:它们有一些预定义的步骤来完成每个方法调用，我们不能用我们的自定义行为来丰富这些步骤。

这种限制有两种主要的解决方法:一种是在编译时，另一种是在运行时。前者通常被像 Scala T1 或 T2 Koltin T3 这样的语言使用，后者是像 JRuby 这样的基于 JVM 的动态语言的解决方案。

运行时方法通常是基于反射的，因此效率很低。

另一方面，编译时解决方案通常依赖于编译时的代码生成。这种方法在运行时效率更高。然而，它有些脆弱，而且可能会导致启动时间变慢，因为有更多的字节码需要处理。

现在我们对问题有了更好的理解，让我们看看解决方案在内部是如何工作的。

## 3.在后台

**`invokedynamic` 让我们以任何我们想要的方式引导方法调用过程**。也就是说，当 JVM 第一次看到一个`invokedynamic `操作码时，它调用一个称为 bootstrap 方法的特殊方法来初始化调用过程:

[![](img/35cc3444901ba4acbaa9e89038f7f308.png)](/web/20220524114233/https://www.baeldung.com/wp-content/uploads/2020/05/Untitled-Diagram.svg)

bootstrap 方法是我们编写的一段普通的 Java 代码，用来设置调用过程。因此，它可以包含任何逻辑。

一旦 bootstrap 方法正常完成，它应该返回一个`[CallSite](https://web.archive.org/web/20220524114233/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/CallSite.html). `的实例。这个`CallSite `封装了以下信息:

*   指向 JVM 应该执行的实际逻辑的指针。这应该表示为`[MethodHandle](/web/20220524114233/https://www.baeldung.com/java-method-handles). `
*   表示返回的`CallSite.`的有效性的条件

**从现在开始，每次 JVM 再次看到这个特定的操作码，都会跳过慢路径，直接调用底层可执行文件**。此外，JVM 将继续跳过慢速路径，直到`CallSite `中的条件改变。

与反射 API 相反，JVM 可以完全看穿`MethodHandle`并试图优化它们，从而获得更好的性能。

### 3.1.引导方法表

让我们再来看看生成的`invokedynamic `字节码:

```java
14: invokedynamic #23,  0  // InvokeDynamic #0:test:()Ljava/util/function/Predicate;
```

这意味着这个特殊的指令应该从引导方法表中调用第一个引导方法(#0 部分)。此外，它提到了传递给 bootstrap 方法的一些参数:

*   `test `是`Predicate`中唯一的抽象方法
*   `()Ljava/util/function/Predicate `表示 JVM 中的方法签名——该方法不接受任何输入，并返回一个`Predicate `接口的实例

为了查看 lambda 示例的引导方法表，我们应该将`-v `选项传递给`javap:`

```java
javap -c -p -v Main
// truncated
// added new lines for brevity
BootstrapMethods:
  0: #55 REF_invokeStatic java/lang/invoke/LambdaMetafactory.metafactory:
    (Ljava/lang/invoke/MethodHandles$Lookup;
     Ljava/lang/String;
     Ljava/lang/invoke/MethodType;
     Ljava/lang/invoke/MethodType;
     Ljava/lang/invoke/MethodHandle;
     Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #62 (Ljava/lang/Object;)Z
      #64 REF_invokeStatic Main.lambda$main$0:(Ljava/lang/String;)Z
      #67 (Ljava/lang/String;)Z
```

所有 lambdas 的 bootstrap 方法是`[LambdaMetafactory](https://web.archive.org/web/20220524114233/https://github.com/openjdk/jdk/blob/a445b66e58a30577dee29cacb636d4c14f0574a2/src/java.base/share/classes/java/lang/invoke/LambdaMetafactory.java#L227) `类中的`[metafactory](https://web.archive.org/web/20220524114233/https://github.com/openjdk/jdk/blob/a445b66e58a30577dee29cacb636d4c14f0574a2/src/java.base/share/classes/java/lang/invoke/LambdaMetafactory.java#L315) `静态方法。

**与所有其他引导方法类似，这种方法至少需要三个参数，如下所示**:

*   `Ljava/lang/invoke/MethodHandles$Lookup`参数表示`invokedynamic`的查找上下文
*   `Ljava/lang/String `表示调用点中的方法名——在本例中，方法名是`test`
*   `Ljava/lang/invoke/MethodType `是调用点的动态方法签名——在本例中，它是`()Ljava/util/function/Predicate`

除了这三个参数，bootstrap 方法还可以选择性地接受一个或多个额外的参数。在本例中，这些是额外的:

*   `(Ljava/lang/Object;)Z `是一个被[擦除的](/web/20220524114233/https://www.baeldung.com/java-type-erasure)方法签名，它接受一个`Object `的实例并返回一个`boolean.`
*   `REF_invokeStatic Main.lambda$main$0:(Ljava/lang/String;)Z `是指向实际 lambda 逻辑的`MethodHandle `。
*   `(Ljava/lang/String;)Z `是一个非擦除的方法签名，接受一个`String `并返回一个`boolean.`

**简单地说，JVM 将把所有需要的信息传递给 bootstrap 方法。Bootstrap 方法将依次使用该信息创建一个适当的`Predicate. `** 实例，然后，JVM 将把该实例传递给`filter `方法。

### 3.2.不同类型的

一旦 JVM 第一次看到这个例子中的`invokedynamic `，它就调用 bootstrap 方法。在撰写本文时，**lambda bootstrap 方法将使用** **`[InnerClassLambdaMetafactory](https://web.archive.org/web/20220524114233/https://github.com/openjdk/jdk/blob/a445b66e58a30577dee29cacb636d4c14f0574a2/src/java.base/share/classes/java/lang/invoke/InnerClassLambdaMetafactory.java#L194)`** **在运行时为 lambda 生成一个内部类。**

然后 bootstrap 方法将生成的内部类封装在一个特殊类型的`CallSite `中，称为`[ConstantCallSite](https://web.archive.org/web/20220524114233/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/ConstantCallSite.html). `。这种类型的`CallSite `在设置后永远不会改变。**因此，在每个 lambda 的第一次设置之后，JVM 将总是使用快速路径来直接调用 lambda 逻辑。**

尽管这是最有效的方式，但肯定不是唯一的选择。事实上，Java 提供了`[MutableCallSite](https://web.archive.org/web/20220524114233/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/MutableCallSite.html) `和`[VolatileCallSite](https://web.archive.org/web/20220524114233/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/VolatileCallSite.html) `来适应更动态的需求。

### 3.3.优势

因此，为了实现 lambda 表达式，Java 不是在编译时创建匿名内部类，而是通过`invokedynamic.`在运行时创建它们

有人可能反对将内部类的生成推迟到运行时。然而，`invokedynamic `方法比简单的编译时解决方案有一些优势。

首先，JVM 直到第一次使用 lambda 时才生成内部类。因此，**在第一次 lambda 执行**之前，我们不会为与内部类相关的额外内存占用付费。

此外，许多链接逻辑从字节码移到了引导方法。因此，**`invokedynamic `字节码通常比备选方案**小得多。较小的字节码可以提高启动速度。

假设一个新版本的 Java 附带了一个更有效的引导方法实现。**那么我们的`invokedynamic `字节码就可以利用这种改进，而不需要重新编译**。这样我们可以实现某种转发二进制兼容性。基本上，我们可以在不同的策略之间切换，而无需重新编译。

最后，用 Java 编写引导和链接逻辑通常比遍历 AST 来生成复杂的字节码更容易。所以，`invokedynamic `可以(主观上)不那么脆弱。

## 4.更多示例

Lambda 表达式不是唯一的特性，Java 也不是唯一使用`invokedynamic. `的语言。在这一节中，我们将熟悉一些其他的动态调用的例子。

### 4.1.Java 14:记录

[记录](/web/20220524114233/https://www.baeldung.com/java-record-keyword)是 [Java 14](https://web.archive.org/web/20220524114233/https://openjdk.java.net/projects/jdk/14/) 中的一个新的预览特性，它提供了一个简洁的语法来声明被认为是哑数据持有者的类。

下面是一个简单的记录示例:

```java
public record Color(String name, int code) {}
```

给定这个简单的一行程序，Java 编译器为访问器方法`toString, equals, `和`hashcode. `生成适当的实现

**为了实现`toString, equals, `或`hashcode, `，Java 正在使用** `**invokedynamic**. `举例，`equals `的字节码如下:

```java
public final boolean equals(java.lang.Object);
    Code:
       0: aload_0
       1: aload_1
       2: invokedynamic #27,  0  // InvokeDynamic #0:equals:(LColor;Ljava/lang/Object;)Z
       7: ireturn
```

另一个解决方案是找到所有记录字段，并在编译时基于这些字段生成`equals `逻辑。字段越多，字节码就越长。

相反，Java 在运行时调用 bootstrap 方法来链接适当的实现。因此，不管字段的数量是多少，字节码长度都将保持不变。

仔细观察字节码可以发现，自举方法是 [`ObjectMethods#bootstrap`](https://web.archive.org/web/20220524114233/https://github.com/openjdk/jdk/blob/827e5e32264666639d36990edd5e7d0b7e7c78a9/src/java.base/share/classes/java/lang/runtime/ObjectMethods.java#L338) :

```java
BootstrapMethods:
  0: #42 REF_invokeStatic java/lang/runtime/ObjectMethods.bootstrap:
    (Ljava/lang/invoke/MethodHandles$Lookup;
     Ljava/lang/String;
     Ljava/lang/invoke/TypeDescriptor;
     Ljava/lang/Class;
     Ljava/lang/String;
     [Ljava/lang/invoke/MethodHandle;)Ljava/lang/Object;
    Method arguments:
      #8 Color
      #49 name;code
      #51 REF_getField Color.name:Ljava/lang/String;
      #52 REF_getField Color.code:I
```

### 4.2.Java 9:字符串连接

在 Java 9 之前，使用`StringBuilder. `作为 JEP [280](https://web.archive.org/web/20220524114233/https://openjdk.java.net/jeps/280) 的一部分来实现重要的字符串连接，现在字符串连接使用`invokedynamic. `例如，让我们将一个常量字符串与一个随机变量连接起来:

```java
"random-" + ThreadLocalRandom.current().nextInt();
```

下面是这个例子中字节码的样子:

```java
0: invokestatic  #7          // Method ThreadLocalRandom.current:()LThreadLocalRandom;
3: invokevirtual #13         // Method ThreadLocalRandom.nextInt:()I
6: invokedynamic #17,  0     // InvokeDynamic #0:makeConcatWithConstants:(I)LString;
```

此外，字符串连接的引导方法位于 [`StringConcatFactory`](https://web.archive.org/web/20220524114233/https://github.com/openjdk/jdk/blob/827e5e32264666639d36990edd5e7d0b7e7c78a9/src/java.base/share/classes/java/lang/invoke/StringConcatFactory.java#L593) 类中:

```java
BootstrapMethods:
  0: #30 REF_invokeStatic java/lang/invoke/StringConcatFactory.makeConcatWithConstants:
    (Ljava/lang/invoke/MethodHandles$Lookup;
     Ljava/lang/String;
     Ljava/lang/invoke/MethodType;
     Ljava/lang/String;
     [Ljava/lang/Object;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #36 random-\u0001
```

## 5.结论

在本文中，首先，我们熟悉了 indy 试图解决的问题。

然后，通过一个简单的 lambda 表达式示例，我们看到了`invokedynamic `在内部是如何工作的。

最后，我们列举了最近版本的 Java 中其他几个 indy 的例子。