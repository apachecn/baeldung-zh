# 用 ASM 操作 Java 字节码指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-asm>

## 1。简介

在本文中，我们将看看如何使用 [ASM](https://web.archive.org/web/20220818200429/http://asm.ow2.org/) 库**通过添加字段、添加方法和改变现有方法的行为来操作现有的 Java 类**。

## 2。依赖性

我们需要将 ASM 依赖项添加到我们的`pom.xml`:

```
<dependency>
    <groupId>org.ow2.asm</groupId>
    <artifactId>asm</artifactId>
    <version>6.0</version>
</dependency>
<dependency>
    <groupId>org.ow2.asm</groupId>
    <artifactId>asm-util</artifactId>
    <version>6.0</version>
</dependency> 
```

我们可以从 Maven Central 获得最新版本的 [asm](https://web.archive.org/web/20220818200429/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.ow2.asm%22%20AND%20a%3A%22asm%22) 和 [asm-util](https://web.archive.org/web/20220818200429/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.ow2.asm%22%20AND%20a%3A%22asm-util%22) 。

## 3。ASM API 基础知识

ASM API 为转换和生成提供了两种与 Java 类交互的风格:基于事件的和基于树的。

### 3.1。基于事件的 API

这个 API 在很大程度上基于`Visitor`模式并且**类似于处理 XML 文档的 SAX 解析模型**。其核心由以下组件组成:

*   帮助读取类文件，是转换类的开始
*   `ClassVisitor`–提供在读取原始类文件后用于转换类的方法
*   `ClassWriter`–用于输出类转换的最终产品

在`ClassVisitor`中，我们有所有的访问者方法，我们将使用它们来接触不同的组件(字段、方法等)。)的一个给定 Java 类。我们通过**提供** `**ClassVisitor**` 的子类来实现给定类中的任何变化。

由于需要保持关于 Java 约定和最终字节码的输出类的完整性，这个类需要一个严格的**顺序，在这个顺序中它的方法应该被调用**以生成正确的输出。

基于事件的 API 中的 *ClassVisitor* 方法按以下顺序调用:

```
visit
visitSource?
visitOuterClass?
( visitAnnotation | visitAttribute )*
( visitInnerClass | visitField | visitMethod )*
visitEnd
```

### 3.2。基于树的 API

这个 API 是一个更加面向对象的 API，类似于处理 XML 文档的 JAXB 模型。

它仍然基于基于事件的 API，但是它引入了`ClassNode`根类。这个类是进入类结构的入口点。

## 4。使用基于事件的 ASM API

我们将使用 ASM 修改`java.lang.Integer`类。在这一点上，我们需要掌握一个基本概念:**`ClassVisitor`类包含了创建或修改类**的所有部分所需的所有访问方法。

我们只需要覆盖必要的 visitor 方法来实现我们的更改。让我们从设置必备组件开始:

```
public class CustomClassWriter {

    static String className = "java.lang.Integer"; 
    static String cloneableInterface = "java/lang/Cloneable";
    ClassReader reader;
    ClassWriter writer;

    public CustomClassWriter() {
        reader = new ClassReader(className);
        writer = new ClassWriter(reader, 0);
    }
}
```

我们以此为基础将`Cloneable`接口添加到股票`Integer` 类中，我们还添加了一个字段和一个方法。

### 4.1。使用字段

让我们创建我们的`ClassVisitor`，我们将使用它向`Integer` 类添加一个字段:

```
public class AddFieldAdapter extends ClassVisitor {
    private String fieldName;
    private String fieldDefault;
    private int access = org.objectweb.asm.Opcodes.ACC_PUBLIC;
    private boolean isFieldPresent;

    public AddFieldAdapter(
      String fieldName, int fieldAccess, ClassVisitor cv) {
        super(ASM4, cv);
        this.cv = cv;
        this.fieldName = fieldName;
        this.access = fieldAccess;
    }
} 
```

接下来，让我们用**覆盖`visitField`方法**，首先**检查我们计划添加的字段是否已经存在，并设置一个标志来指示状态**。

我们仍然需要**将方法调用转发给父类**——这需要发生在为类中的每个字段调用`visitField`方法时。**未能转发呼叫意味着没有字段将被写入该类。**

该方法还允许我们**修改现有字段**的可见性或类型:

```
@Override
public FieldVisitor visitField(
  int access, String name, String desc, String signature, Object value) {
    if (name.equals(fieldName)) {
        isFieldPresent = true;
    }
    return cv.visitField(access, name, desc, signature, value); 
} 
```

我们首先检查前面的`visitField`方法中设置的标志，并再次调用`visitField`方法，这次提供名称、访问修饰符和描述。这个方法返回一个`FieldVisitor.`的实例

**`visitEnd` 方法是访问者方法中最后一个叫做**的方法。这是**执行现场插入逻辑**的推荐位置。

然后，我们需要在这个对象上调用`visitEnd`方法来**表示我们已经完成了对这个字段的访问:**

```
@Override
public void visitEnd() {
    if (!isFieldPresent) {
        FieldVisitor fv = cv.visitField(
          access, fieldName, fieldType, null, null);
        if (fv != null) {
            fv.visitEnd();
        }
    }
    cv.visitEnd();
} 
```

**确保所有使用的 ASM 组件来自`org.objectweb.asm` 包**是很重要的——许多库在内部使用 ASM 库，ide 可以自动插入捆绑的 ASM 库。

我们现在在`addField` 方法中使用我们的适配器，**用我们添加的字段获得了`java.lang.Integer`** 的转换版本:

```
public class CustomClassWriter {
    AddFieldAdapter addFieldAdapter;
    //...
    public byte[] addField() {
        addFieldAdapter = new AddFieldAdapter(
          "aNewBooleanField",
          org.objectweb.asm.Opcodes.ACC_PUBLIC,
          writer);
        reader.accept(addFieldAdapter, 0);
        return writer.toByteArray();
    }
}
```

我们已经覆盖了`visitField`和`visitEnd`方法。

与字段相关的所有操作都是通过`visitField` 方法完成的。这意味着我们还可以通过更改传递给`visitField`方法的所需值来修改现有字段(比如，将私有字段转换为公共字段)。

### 4.2。使用方法

在 ASM API 中生成整个方法比类中的其他操作更复杂。这涉及到大量的低级字节码操作，因此超出了本文的范围。

然而，在大多数实际应用中，我们可以**修改一个现有的方法，使其更易访问**(也许将其公开，这样它就可以被覆盖或重载)或者**修改一个类，使其可扩展**。

让我们公开 toUnsignedString 方法:

```
public class PublicizeMethodAdapter extends ClassVisitor {
    public PublicizeMethodAdapter(int api, ClassVisitor cv) {
        super(ASM4, cv);
        this.cv = cv;
    }
    public MethodVisitor visitMethod(
      int access,
      String name,
      String desc,
      String signature,
      String[] exceptions) {
        if (name.equals("toUnsignedString0")) {
            return cv.visitMethod(
              ACC_PUBLIC + ACC_STATIC,
              name,
              desc,
              signature,
              exceptions);
        }
        return cv.visitMethod(
          access, name, desc, signature, exceptions);
   }
} 
```

就像我们对字段修改所做的那样，我们仅仅是**截取访问方法并改变我们想要的参数**。

在这种情况下，我们使用`org.objectweb.asm.Opcodes` 包中的访问修饰符来**改变方法**的可见性。然后我们插入我们的`ClassVisitor`:

```
public byte[] publicizeMethod() {
    pubMethAdapter = new PublicizeMethodAdapter(writer);
    reader.accept(pubMethAdapter, 0);
    return writer.toByteArray();
} 
```

### 4.3。使用类

沿着与修改方法相同的路线，我们**通过拦截适当的访问者方法**来修改类。在这种情况下，我们截取`visit`，这是访问者层次结构中的第一个方法:

```
public class AddInterfaceAdapter extends ClassVisitor {

    public AddInterfaceAdapter(ClassVisitor cv) {
        super(ASM4, cv);
    }

    @Override
    public void visit(
      int version,
      int access,
      String name,
      String signature,
      String superName, String[] interfaces) {
        String[] holding = new String[interfaces.length + 1];
        holding[holding.length - 1] = cloneableInterface;
        System.arraycopy(interfaces, 0, holding, 0, interfaces.length);
        cv.visit(V1_8, access, name, signature, superName, holding);
    }
} 
```

我们覆盖了`visit` 方法，将`Cloneable` 接口添加到由`Integer`类支持的接口数组中。我们把它插上，就像我们的适配器的所有其他用途一样。

## 5。使用修改后的类

所以我们修改了`Integer` 类。现在我们需要能够加载和使用类的修改版本。

除了简单地将`writer.toByteArray`的输出作为一个类文件写入磁盘，还有一些其他方法可以与我们定制的`Integer`类交互。

### 5.1。使用`TraceClassVisitor`

ASM 库提供了`TraceClassVisitor` 实用程序类，我们将用它来**自省修改后的类**。因此，我们可以**确认我们的变化已经发生**。

因为`TraceClassVisitor`是一个`ClassVisitor`，我们可以用它来代替标准的`ClassVisitor`:

```
PrintWriter pw = new PrintWriter(System.out);

public PublicizeMethodAdapter(ClassVisitor cv) {
    super(ASM4, cv);
    this.cv = cv;
    tracer = new TraceClassVisitor(cv,pw);
}

public MethodVisitor visitMethod(
  int access,
  String name,
  String desc,
  String signature,
  String[] exceptions) {
    if (name.equals("toUnsignedString0")) {
        System.out.println("Visiting unsigned method");
        return tracer.visitMethod(
          ACC_PUBLIC + ACC_STATIC, name, desc, signature, exceptions);
    }
    return tracer.visitMethod(
      access, name, desc, signature, exceptions);
}

public void visitEnd(){
    tracer.visitEnd();
    System.out.println(tracer.p.getText());
} 
```

我们在这里所做的是用`TraceClassVisitor`来修改我们传递给先前的`PublicizeMethodAdapter` 的`ClassVisitor` 。

现在所有的访问都将由我们的 tracer 完成，它可以打印出转换后的类的内容，显示我们对它所做的任何修改。

虽然 ASM 文档指出`TraceClassVisitor`可以打印出提供给构造函数的`PrintWriter`,但这在最新版本的 ASM 中似乎不能正常工作。

幸运的是，我们可以访问类中的底层打印机，并且能够在我们覆盖的`visitEnd`方法中手动打印出跟踪器的文本内容。

### 5.2。使用 Java 工具

这是一个更优雅的解决方案，允许我们通过[插装](https://web.archive.org/web/20220818200429/https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/package-summary.html)在更近的层次上与 JVM 一起工作。

为了检测`java.lang.Integer`类，我们**编写一个代理，它将被配置为 JVM** 的命令行参数。该代理需要两个组件:

*   实现名为`premain`的方法的类
*   一个`[ClassFileTransformer](https://web.archive.org/web/20220818200429/https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/ClassFileTransformer.html)` 的实现，其中我们将有条件地提供我们类的修改版本

```
public class Premain {
    public static void premain(String agentArgs, Instrumentation inst) {
        inst.addTransformer(new ClassFileTransformer() {
            @Override
            public byte[] transform(
              ClassLoader l,
              String name,
              Class c,
              ProtectionDomain d,
              byte[] b)
              throws IllegalClassFormatException {
                if(name.equals("java/lang/Integer")) {
                    CustomClassWriter cr = new CustomClassWriter(b);
                    return cr.addField();
                }
                return b;
            }
        });
    }
}
```

我们现在使用 Maven jar 插件在 JAR 清单文件中定义我们的`premain`实现类:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.4</version>
    <configuration>
        <archive>
            <manifestEntries>
                <Premain-Class>
                    com.baeldung.examples.asm.instrumentation.Premain
                </Premain-Class>
                <Can-Retransform-Classes>
                    true
                </Can-Retransform-Classes>
            </manifestEntries>
        </archive>
    </configuration>
</plugin>
```

到目前为止，构建和打包我们的代码产生了可以作为代理加载的 jar。要在假设的“`YourClass.class`”中使用我们定制的`Integer`类:

```
java YourClass -javaagent:"/path/to/theAgentJar.jar"
```

## 6。结论

虽然我们在这里单独实现了我们的转换，但是 ASM 允许我们将多个适配器链接在一起，以实现复杂的类转换。

除了我们在这里讨论的基本转换，ASM 还支持与注释、泛型和内部类的交互。

我们已经看到了 ASM 库的一些强大之处——它消除了我们在第三方库甚至标准 JDK 类中可能遇到的许多限制。

ASM 在一些最流行的库(Spring、AspectJ、JDK 等)下被广泛使用。)在飞行中表演很多“魔术”。

您可以在 [GitHub 项目](https://web.archive.org/web/20220818200429/https://github.com/eugenp/tutorials/tree/master/asm)中找到本文的源代码。