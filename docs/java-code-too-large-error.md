# Java 中的“代码太大”编译错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-code-too-large-error>

## 1。概述

**当一个 J** **ava 方法超过 65535 字节时，我们得到编译错误，“代码太大”**。在本文中，我们将讨论为什么会出现这个错误以及如何修复它。

## 2。JVM 约束

[`Code_attribute`](https://web.archive.org/web/20220626074731/https://docs.oracle.com/javase/specs/jvms/se16/html/jvms-4.html#jvms-4.7.3) 是 JVM 规范的`method_info`结构中一个可变长度的表。该结构包含方法的 JVM 指令，该方法可以是实例、类或接口的常规方法或初始化方法:

```java
Code_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 max_stack;
    u2 max_locals;
    u4 code_length;
    u1 code[code_length];
    u2 exception_table_length;
    {   
        u2 start_pc;
        u2 end_pc;
        u2 handler_pc;
        u2 catch_type;
    }
    exception_table[exception_table_length];
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```

属性`code_length`指定了方法中代码的长度:

```java
code_length
The value of the code_length item gives the number of bytes in the code array for this method.
The value of code_length must be greater than zero (as the code array must not be empty) and less than 65536. 
```

从上面可以看出， **JVM 规范声明** t **一个方法的代码长度必须小于 65536 字节，所以这意味着一个方法的大小不能超过 65535 字节**。

## 3。为什么会出现问题

现在我们已经知道了方法的大小限制，让我们来看看会产生如此大的方法的情况:

*   代码生成器:大多数大型方法都是使用代码生成器的结果，比如 [ANTLR parser](/web/20220626074731/https://www.baeldung.com/java-antlr)
*   初始化方法:GUI 初始化可以添加很多细节，比如布局、事件监听器等等，所有这些都在一个方法中完成
*   JSP 页面:在该类的一个方法中包含所有代码
*   [代码检测](/web/20220626074731/https://www.baeldung.com/java-instrumentation):在运行时将字节码添加到编译后的类中
*   数组初始化器:初始化非常大的数组的方法，如下所示:

```java
String[][] largeStringArray = new String[][] {
    { "java", "code", "exceeded", "65355", "bytes" },
    { "alpha", "beta", "gamma", "delta", "epsilon" },
    { "one", "two", "three", "four", "five" }, 
    { "uno", "dos", "tres", "cuatro", "cinco" }, 

    //More values
};
```

## 4。如何修复错误

正如我们注意到的，错误的根本原因是方法超过了 65535 字节的阈值。因此，**将出错的方法重构为几个更小的方法**将为我们解决这个问题。

在数组初始化的情况下，我们可以拆分数组或者从文件中加载。我们也可以使用[静态初始化器](/web/20220626074731/https://www.baeldung.com/java-initialization#2-static-initialization-block)。即使我们使用代码生成器，我们仍然可以重构代码。在一个大的 JSP 文件的情况下，我们可以使用 j *sp:include* 指令并把它分成更小的单元。

上面的问题相对容易处理，但是当我们在代码中添加了检测工具之后，出现“代码太大”的错误时，事情就变得复杂了。如果我们拥有代码，我们仍然可以重构方法。但是当我们从第三方库得到这个错误时，我们就陷入了困境。通过降低仪器等级，我们也许能够解决这个问题。

## 5。结论

在本文中，我们讨论了“代码太大”错误的原因和可能的解决方案。我们总是可以参考 JVM 规范的 [Code_Attributes 部分来找到关于这个约束的更多细节。](https://web.archive.org/web/20220626074731/https://docs.oracle.com/javase/specs/jvms/se16/html/jvms-4.html#jvms-4.7.3)