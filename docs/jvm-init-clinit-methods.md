# JVM 中的`<init>`和`<clinit>`方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-init-clinit-methods>

## 1.概观

JVM 使用两种不同的方法来初始化对象实例和类。

在这篇简短的文章中，我们将看到编译器和运行时如何使用`<init> `和`<clinit> `方法进行初始化。

## 2.实例初始化方法

让我们从简单的对象分配和赋值开始:

```java
Object obj = new Object();
```

如果我们编译这个代码片段，并通过`javap -c`查看它的字节码，我们会看到类似这样的内容:

```java
0: new           #2      // class java/lang/Object
3: dup
4: invokespecial #1      // Method java/lang/Object."<init>":()V
7: astore_1
```

为了初始化对象`,`，JVM 调用一个特殊的方法，用 JVM 的行话来说叫做`<init>.` **，这个方法是一个[实例初始化方法](https://web.archive.org/web/20220625225859/https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html#jvms-2.9.1)** 。当且仅当以下条件成立时，方法才是实例初始化:

*   它是在类中定义的
*   它的名字叫`<` `init>`
*   它返回`void`

每个类可以有零个或多个实例初始化方法。这些方法通常对应于基于 JVM 的编程语言(如 Java 或 Kotlin)中的构造函数。

### 2.1.构造函数和实例初始化程序块

为了更好地理解 Java 编译器如何将构造函数翻译成`<init>`，让我们考虑另一个例子:

```java
public class Person {

    private String firstName = "Foo"; // <init>
    private String lastName = "Bar"; // <init>

    // <init>
    {
        System.out.println("Initializing...");
    }

    // <init>
    public Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    // <init>
    public Person() {
    }
}
```

这是这个类的字节码:

```java
public Person(java.lang.String, java.lang.String);
  Code:
     0: aload_0
     1: invokespecial #1       // Method java/lang/Object."<init>":()V
     4: aload_0
     5: ldc           #7       // String Foo
     7: putfield      #9       // Field firstName:Ljava/lang/String;
    10: aload_0
    11: ldc           #15      // String Bar
    13: putfield      #17      // Field lastName:Ljava/lang/String;
    16: getstatic     #20      // Field java/lang/System.out:Ljava/io/PrintStream;
    19: ldc           #26      // String Initializing...
    21: invokevirtual #28      // Method java/io/PrintStream.println:(Ljava/lang/String;)V
    24: aload_0
    25: aload_1
    26: putfield      #9       // Field firstName:Ljava/lang/String;
    29: aload_0
    30: aload_2
    31: putfield      #17      // Field lastName:Ljava/lang/String;
    34: return
```

**尽管在 Java 中构造器和初始化器块是分开的，但是它们在字节码级别上是在同一个实例初始化方法中。**事实上，这个`<init> `方法:

*   首先，初始化`firstName `和`lastName `字段(索引 0 到 13)
*   然后，它将一些内容打印到控制台，作为实例初始化块的一部分(索引 16 到 21)
*   最后，它用构造函数参数更新实例变量

如果我们如下创建一个`Person` :

```java
Person person = new Person("Brian", "Goetz");
```

然后，这将转化为以下字节码:

```java
0: new           #7        // class Person
3: dup
4: ldc           #9        // String Brian
6: ldc           #11       // String Goetz
8: invokespecial #13       // Method Person."<init>":(Ljava/lang/String;Ljava/lang/String;)V
11: astore_1
```

这一次 JVM 调用另一个`<init> `方法，其签名对应于 Java 构造函数。

**这里的关键要点是，构造函数和其他实例初始化器相当于 JVM 世界中的`<init> `方法。**

## 3.类初始化方法

在 Java 中，[静态初始化器块](/web/20220625225859/https://www.baeldung.com/java-static#a-static-block)在我们要在类级别初始化某些东西时很有用:

```java
public class Person {

    private static final Logger LOGGER = LoggerFactory.getLogger(Person.class); // <clinit>

    // <clinit>
    static {
        System.out.println("Static Initializing...");
    }

    // omitted
}
```

当我们编译前面的代码时，编译器将静态块翻译成字节码级别的[类初始化方法](https://web.archive.org/web/20220625225859/https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html#jvms-2.9.2)。

简单地说，一个方法是类初始化方法，当且仅当:

*   它的名字叫`<clinit>`
*   它返回`void`

**因此，在 Java 中生成`<clinit>`方法的唯一方法就是使用静态字段和静态块初始化器。**

JVM 在我们第一次使用相应的类时调用`<clinit>`。因此，`<clinit> `调用发生在运行时，我们看不到字节码级别的调用。

## 4.结论

在这篇简短的文章中，我们看到了 JVM 中的`<init> `和`<clinit> `方法之间的区别。`<init> `方法用于初始化对象实例。此外，JVM 调用`<clinit> `方法在必要时初始化一个类。

为了更好地理解初始化在 JVM 中是如何工作的，强烈建议阅读一下 [JVM 规范](https://web.archive.org/web/20220625225859/https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-5.html#jvms-5.5)。