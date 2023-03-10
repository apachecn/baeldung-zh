# 基于 Java 11 嵌套的访问控制

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nest-based-access-control>

## 1。简介

在本教程中，我们将探索 Java 11 中引入的新的访问控制上下文`nests`。

## 2。Java 11 之前

### 2.1.嵌套类型

Java 允许类和接口[互相嵌套](/web/20221205232039/https://www.baeldung.com/java-nested-classes)。这些**嵌套类型可以无限制地相互访问**，包括私有字段、方法和构造函数。

考虑下面的嵌套类示例:

```java
public class Outer {

    public void outerPublic() {
    }

    private void outerPrivate() {
    }

    class Inner {

        public void innerPublic() {
            outerPrivate();
        }
    }
}
```

这里，虽然方法`outerPrivate()`是`private`，但是它可以从方法`innerPublic()`中访问。

我们可以将一个顶级类型，加上嵌套在其中的所有类型，描述为形成一个嵌套。一个巢的两个成员被称为巢友。

因此，在上面的例子中，`Outer`和`Inner`一起形成一个巢，并且是彼此的巢友。

### 2.2.桥路法

JVM 访问规则不允许嵌套成员之间的私有访问。理想情况下，对于上面的例子，我们应该得到一个编译错误。然而，Java 源代码编译器通过引入间接层来允许这种访问。

例如，**对私有成员的调用被编译成对编译器生成的、包私有的、目标类中的桥接方法的调用，该方法又调用预期的私有方法**。

这发生在幕后。这种桥接方法稍微增加了部署的应用程序的大小，并且会使用户和工具感到困惑。

### 2.3.使用反射

进一步的结果是核心反射也拒绝访问。这是令人惊讶的，因为反射调用的行为应该与源代码级调用相同。

例如，如果我们试图从`Inner`类反射性地调用`outerPrivate()`:

```java
public void innerPublicReflection(Outer ob) throws Exception {
    Method method = ob.getClass().getDeclaredMethod("outerPrivate");
    method.invoke(ob);
}
```

我们会得到一个异常:

```java
java.lang.IllegalAccessException: 
Class com.baeldung.Outer$Inner can not access a member of class com.baeldung.Outer with modifiers "private"
```

Java 11 试图解决这些问题。

## 3。基于嵌套的访问控制

Java 11 在 JVM 中引入了嵌套的概念和相关的访问规则。这简化了 Java 源代码编译器的工作。

为此，类文件格式现在包含两个新属性:

1.  一个嵌套成员(通常是顶级类)被指定为嵌套宿主。它包含一个属性(NestMembers)来标识其他静态已知的嵌套成员。
2.  每个其他嵌套成员都有一个属性(NestHost)来标识其嵌套主机。

因此，为了使类型`C`和`D`成为巢友，它们必须有相同的巢宿主。如果类型`C`在其 NestHost 属性中列出了`D`，那么它就声称是由`D`托管的嵌套的成员。如果`D`在其 NestMembers 属性中也列出了`C`,则成员资格有效。此外，类型`D`是它所承载的嵌套的隐式成员。

现在**不需要编译器生成桥方法**。

最后，基于嵌套的访问控制从核心反射中消除了令人惊讶的行为。因此，上一节中显示的方法`innerPublicReflection()`将无任何异常地执行。

## 4。嵌套反射 API

**Java 11 提供了使用核心反射**查询新类文件属性的方法。类`java.lang.Class`包含以下三个新方法。

### 4.1.`getNestHost()`

这将返回该`Class`对象所属嵌套的嵌套主机:

```java
@Test
public void whenGetNestHostFromOuter_thenGetNestHost() {
    is(Outer.class.getNestHost().getName()).equals("com.baeldung.Outer");
}

@Test
public void whenGetNestHostFromInner_thenGetNestHost() {
    is(Outer.Inner.class.getNestHost().getName()).equals("com.baeldung.Outer");
}
```

`Outer`和`Inner`类都属于嵌套宿主`com.baeldung.Outer`。

### 4.2.`isNestmateOf()`

这决定了给定的`Class`是否是这个`Class`对象的嵌套:

```java
@Test
public void whenCheckNestmatesForNestedClasses_thenGetTrue() {
    is(Outer.Inner.class.isNestmateOf(Outer.class)).equals(true);
}
```

### 4.3.`getNestMembers()`

这将返回一个数组，其中包含代表这个`Class`对象所属嵌套的所有成员的`Class`对象:

```java
@Test
public void whenGetNestMembersForNestedClasses_thenGetAllNestedClasses() {
    Set<String> nestMembers = Arrays.stream(Outer.Inner.class.getNestMembers())
      .map(Class::getName)
      .collect(Collectors.toSet());

    is(nestMembers.size()).equals(2);

    assertTrue(nestMembers.contains("com.baeldung.Outer"));
    assertTrue(nestMembers.contains("com.baeldung.Outer$Inner"));
}
```

## 5。编译细节

### 5.1.Java 11 之前的桥接方法

让我们深入研究编译器生成的桥接方法的细节。我们可以通过分解生成的类文件来看到这一点:

```java
$ javap -c Outer
Compiled from "Outer.java"
public class com.baeldung.Outer {
  public com.baeldung.Outer();
    Code:
       0: aload_0
       1: invokespecial #2                  // Method java/lang/Object."<init>":()V
       4: return

  public void outerPublic();
    Code:
       0: return

  static void access$000(com.baeldung.Outer);
    Code:
       0: aload_0
       1: invokespecial #1                  // Method outerPrivate:()V
       4: return
}
```

这里，除了默认的构造函数和公共方法`outerPublic()`，**注意方法`access$000()`。编译器将此作为桥接方法生成。**

`innerPublic()`通过这个方法调用`outerPrivate()`:

```java
$ javap -c Outer\$Inner
Compiled from "Outer.java"
class com.baeldung.Outer$Inner {
  final com.baeldung.Outer this$0;

  com.baeldung.Outer$Inner(com.baeldung.Outer);
    Code:
       0: aload_0
       1: aload_1
       2: putfield      #1                  // Field this$0:Lcom/baeldung/Outer;
       5: aload_0
       6: invokespecial #2                  // Method java/lang/Object."<init>":()V
       9: return

  public void innerPublic();
    Code:
       0: aload_0
       1: getfield      #1                  // Field this$0:Lcom/baeldung/Outer;
       4: invokestatic  #3                  // Method com/baeldung/Outer.access$000:(Lcom/baeldung/Outer;)V
       7: return
}
```

注意第 19 行的注释。这里，`innerPublic()`调用桥方法`access$000()`。

### 5.2.与 Java 11 嵌套

Java 11 编译器将生成以下反汇编的`Outer`类文件:

```java
$ javap -c Outer
Compiled from "Outer.java"
public class com.baeldung.Outer {
  public com.baeldung.Outer();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public void outerPublic();
    Code:
       0: return
}
```

注意，这里没有编译器生成的桥接方法。另外，`Inner`类现在可以直接调用`outerPrivate()`方法:

```java
$ javap -c Outer\$Inner.class 
Compiled from "Outer.java"
class com.baeldung.Outer$Inner {
  final com.baeldung.Outer this$0;

  com.baeldung.Outer$Inner(com.baeldung.Outer);
    Code:
       0: aload_0
       1: aload_1
       2: putfield      #1                  // Field this$0:Lcom/baeldung/Outer;
       5: aload_0
       6: invokespecial #2                  // Method java/lang/Object."<init>":()V
       9: return

  public void innerPublic();
    Code:
       0: aload_0
       1: getfield      #1                  // Field this$0:Lcom/baeldung/Outer;
       4: invokevirtual #3                  // Method com/baeldung/Outer.outerPrivate:()V
       7: return
}
```

## 6。结论

在本文中，我们探讨了 Java 11 中引入的基于嵌套的访问控制。

像往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221205232039/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11)