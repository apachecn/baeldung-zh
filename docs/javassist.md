# Javassist 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javassist>

## 1。概述

在本文中，我们将关注`[Javasisst](https://web.archive.org/web/20220625080632/https://jboss-javassist.github.io/javassist/) (Java Programming Assistant)` 库。

简单地说，这个库通过使用一个比 JDK 中的 API 更高级的 API，使得操纵 Java 字节码的过程变得更简单。

## 2。Maven 依赖关系

要将 Javassist 库添加到我们的项目中，我们需要将`[javassist](https://web.archive.org/web/20220625080632/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22javassist%22%20AND%20a%3A%22javassist%22)`添加到 pom 中:

```java
<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>${javaassist.version}</version>
</dependency>

<properties>
    <javaassist.version>3.21.0-GA</javaassist.version>
</properties>
```

## 3。字节码是什么？

在一个非常高的层次上，每一个用纯文本格式编写并编译成字节码的 Java 类——一个可以被 Java 虚拟机处理的指令集。JVM 将字节码指令翻译成机器级汇编指令。

假设我们有一个`Point` 类:

```java
public class Point {
    private int x;
    private int y;

    public void move(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // standard constructors/getters/setters
}
```

编译后，将创建包含字节码的`Point.class` 文件。我们可以通过执行`javap` 命令来查看该类的字节码:

```java
javap -c Point.class
```

这将打印以下输出:

```java
public class com.baeldung.javasisst.Point {
  public com.baeldung.javasisst.Point(int, int);
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: iload_1
       6: putfield      #2                  // Field x:I
       9: aload_0
      10: iload_2
      11: putfield      #3                  // Field y:I
      14: return

  public void move(int, int);
    Code:
       0: aload_0
       1: iload_1
       2: putfield      #2                  // Field x:I
       5: aload_0
       6: iload_2
       7: putfield      #3                  // Field y:I
      10: return
}
```

所有这些指令都是由 Java 语言指定的；[大量可用](https://web.archive.org/web/20220625080632/https://en.wikipedia.org/wiki/Java_bytecode_instruction_listings)。

让我们分析一下`move()`方法的字节码指令:

*   `aload_0` 指令从局部变量 0 加载一个引用到堆栈上
*   `iload_1`正在从局部变量 1 中加载一个 int 值
*   `putfield`正在设置我们对象的一个字段`x`。所有操作与`y`领域类似
*   最后一条指令是`return`

每一行 Java 代码都用适当的指令编译成字节码。Javassist 库使得操作字节码变得相对容易。

## 4。生成 Java 类

Javassist 库可用于生成新的 Java 类文件。

假设我们想要生成一个实现了一个`java.lang.Cloneable` 接口的`JavassistGeneratedClass` 类。我们希望该类有一个`int`类型`.` 的`id` 字段，`ClassFile` 用于创建一个新的类文件，`FieldInfo`用于向类`:`添加一个新字段

```java
ClassFile cf = new ClassFile(
  false, "com.baeldung.JavassistGeneratedClass", null);
cf.setInterfaces(new String[] {"java.lang.Cloneable"});

FieldInfo f = new FieldInfo(cf.getConstPool(), "id", "I");
f.setAccessFlags(AccessFlag.PUBLIC);
cf.addField(f); 
```

在我们创建了一个`JavassistGeneratedClass.class` 之后，我们可以断言它实际上有一个`id` 字段:

```java
ClassPool classPool = ClassPool.getDefault();
Field[] fields = classPool.makeClass(cf).toClass().getFields();

assertEquals(fields[0].getName(), "id");
```

## 5。加载类的字节码指令

如果我们想要加载一个已经存在的类方法的字节码指令，我们可以得到该类的一个特定方法的`CodeAttribute` 。然后我们可以得到一个`CodeIterator` 来迭代该方法的所有字节码指令。

让我们加载`Point`类的`move()` 方法的所有字节码指令:

```java
ClassPool cp = ClassPool.getDefault();
ClassFile cf = cp.get("com.baeldung.javasisst.Point")
  .getClassFile();
MethodInfo minfo = cf.getMethod("move");
CodeAttribute ca = minfo.getCodeAttribute();
CodeIterator ci = ca.iterator();

List<String> operations = new LinkedList<>();
while (ci.hasNext()) {
    int index = ci.next();
    int op = ci.byteAt(index);
    operations.add(Mnemonic.OPCODE[op]);
}

assertEquals(operations,
  Arrays.asList(
  "aload_0", 
  "iload_1", 
  "putfield", 
  "aload_0", 
  "iload_2",  
  "putfield", 
  "return"));
```

通过将字节码聚集到操作列表中，我们可以看到`move()` 方法的所有字节码指令，如上面的断言所示。

## 6。向现有类字节码添加字段

假设我们想在现有类的字节码中添加一个`int`类型的字段。我们可以使用`ClassPoll` 加载该类，并向其中添加一个字段:

```java
ClassFile cf = ClassPool.getDefault()
  .get("com.baeldung.javasisst.Point").getClassFile();

FieldInfo f = new FieldInfo(cf.getConstPool(), "id", "I");
f.setAccessFlags(AccessFlag.PUBLIC);
cf.addField(f); 
```

我们可以使用反射来验证`Point` 类中是否存在`id`字段:

```java
ClassPool classPool = ClassPool.getDefault();
Field[] fields = classPool.makeClass(cf).toClass().getFields();
List<String> fieldsList = Stream.of(fields)
  .map(Field::getName)
  .collect(Collectors.toList());

assertTrue(fieldsList.contains("id"));
```

## 7。将构造函数添加到类字节码

我们可以通过使用一个`addInvokespecial()` 方法将一个构造函数添加到前面一个例子中提到的现有类中。

我们可以通过调用`java.lang.Object`类中的`<init>` 方法来添加无参数构造函数:

```java
ClassFile cf = ClassPool.getDefault()
  .get("com.baeldung.javasisst.Point").getClassFile();
Bytecode code = new Bytecode(cf.getConstPool());
code.addAload(0);
code.addInvokespecial("java/lang/Object", MethodInfo.nameInit, "()V");
code.addReturn(null);

MethodInfo minfo = new MethodInfo(
  cf.getConstPool(), MethodInfo.nameInit, "()V");
minfo.setCodeAttribute(code.toCodeAttribute());
cf.addMethod(minfo);
```

我们可以通过迭代字节码来检查新创建的构造函数是否存在:

```java
CodeIterator ci = code.toCodeAttribute().iterator();
List<String> operations = new LinkedList<>();
while (ci.hasNext()) {
    int index = ci.next();
    int op = ci.byteAt(index);
    operations.add(Mnemonic.OPCODE[op]);
}

assertEquals(operations,
  Arrays.asList("aload_0", "invokespecial", "return"));
```

## 8。结论

在本文中，我们介绍了 Javassist 库，目的是使字节码操作更容易。

我们关注核心特性，并从 Java 代码中生成了一个类文件；我们还对已经创建的 Java 类进行了一些字节码操作。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220625080632/https://github.com/eugenp/tutorials/tree/master/libraries)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。