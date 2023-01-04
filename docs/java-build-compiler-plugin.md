# 创建 Java 编译器插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-build-compiler-plugin>

## 1。概述

Java 8 提供了创建`Javac`插件的 API。不幸的是，很难找到好的文档。

在本文中，我们将展示创建一个编译器扩展的整个过程，该扩展向`*.class`文件添加定制代码。

## 2。设置

首先，我们需要添加 JDK 的`tools.jar`作为我们项目的依赖项:

```java
<dependency>
    <groupId>com.sun</groupId>
    <artifactId>tools</artifactId>
    <version>1.8.0</version>
    <scope>system</scope>
    <systemPath>${java.home}/../lib/tools.jar</systemPath>
</dependency>
```

**每个编译器扩展都是一个实现`com.sun.source.util.Plugin`接口的类。**让我们在我们的例子中创建它:

让我们在示例中创建它:

```java
public class SampleJavacPlugin implements Plugin {

    @Override
    public String getName() {
        return "MyPlugin";
    }

    @Override
    public void init(JavacTask task, String... args) {
        Context context = ((BasicJavacTask) task).getContext();
        Log.instance(context)
          .printRawLines(Log.WriterKind.NOTICE, "Hello from " + getName());
    }
}
```

现在，我们只是打印“Hello ”,以确保我们的代码被成功提取并包含在编译中。

我们的最终目标是创建一个插件，为每个标有给定注释的数字参数添加运行时检查，如果参数不符合条件，抛出异常。

还有一个必要的步骤使扩展可以被`Javac:` **发现，它应该通过`ServiceLoader`框架公开。**

为了实现这一点，我们需要创建一个名为`com.sun.source.util.Plugin`的文件，其内容是我们插件的完全限定类名(`com.baeldung.javac.SampleJavacPlugin`)，并将它放在`META-INF/services` 目录中。

之后，我们可以用`-Xplugin:MyPlugin`开关调用`Javac`:

```java
baeldung/tutorials$ javac -cp ./core-java/target/classes -Xplugin:MyPlugin ./core-java/src/main/java/com/baeldung/javac/TestClass.java
Hello from MyPlugin
```

注意**我们必须总是使用从插件的`getName()`方法返回的`String`作为`-Xplugin`选项值**。

## 3。插件生命周期

编译器只通过`init()`方法调用一次**插件。**

为了获得后续事件的通知，我们必须注册一个回调。这些在每个源文件的每个处理阶段之前和之后到达:

*   `PARSE`–构建一个`Abstract Syntax Tree` (AST)
*   *回车*–源代码导入被解析
*   *分析*–分析解析器输出(AST)的错误
*   *生成*–为目标源文件生成二进制文件

还有两种事件类型—`ANNOTATION_PROCESSING`和`ANNOTATION_PROCESSING_ROUND`,但是我们在这里对它们不感兴趣。

例如，当我们想要通过添加一些基于源代码信息的检查来增强编译时，在`PARSE finished`事件处理程序中这样做是合理的:

```java
public void init(JavacTask task, String... args) {
    task.addTaskListener(new TaskListener() {
        public void started(TaskEvent e) {
        }

        public void finished(TaskEvent e) {
            if (e.getKind() != TaskEvent.Kind.PARSE) {
                return;
            }
            // Perform instrumentation
        }
    });
}
```

## 4。提取 AST 数据

**我们可以通过*task event . getcompilationunit()*得到 Java 编译器生成的 AST。**它的细节可以通过 *TreeVisitor* 界面查看。

注意，只有一个被调用了 *accept()* 方法的*树*元素将事件分派给给定的访问者。

比如我们执行*class tree . accept(visitor)*时，只触发*visit class()*；我们不能期望，比方说， *visitMethod()* 也为给定类中的每个方法激活。

我们可以使用 *TreeScanner* 来解决这个问题:

```java
public void finished(TaskEvent e) {
    if (e.getKind() != TaskEvent.Kind.PARSE) {
        return;
    }
    e.getCompilationUnit().accept(new TreeScanner<Void, Void>() {
        @Override
        public Void visitClass(ClassTree node, Void aVoid) {
            return super.visitClass(node, aVoid);
        }

        @Override
        public Void visitMethod(MethodTree node, Void aVoid) {
            return super.visitMethod(node, aVoid);
        }
    }, null);
}
```

在这个例子中，需要调用 *super.visitXxx(node，value)* 来递归处理当前节点的子节点。

## 5。修改 AST

为了展示我们如何修改 AST，我们将为所有标有`@Positive`注释的数字参数插入运行时检查。

这是一个可以应用于方法参数的简单注释:

```java
@Documented
@Retention(RetentionPolicy.CLASS)
@Target({ElementType.PARAMETER})
public @interface Positive { }
```

下面是一个使用注释的示例:

```java
public void service(@Positive int i) { }
```

最后，我们希望字节码看起来像是从这样的源代码编译的:

```java
public void service(@Positive int i) {
    if (i <= 0) {
        throw new IllegalArgumentException("A non-positive argument ("
          + i + ") is given as a @Positive parameter 'i'");
    }
}
```

**这意味着我们希望对于每个标有等于或小于 0 的`@Positive`的参数抛出一个`IllegalArgumentException`。**

### 5.1。在哪里安装仪器

让我们看看如何定位应该应用仪器的目标位置:

```java
private static Set<String> TARGET_TYPES = Stream.of(
  byte.class, short.class, char.class, 
  int.class, long.class, float.class, double.class)
 .map(Class::getName)
 .collect(Collectors.toSet()); 
```

为了简单起见，我们在这里只添加了基本的数字类型。

接下来，让我们定义一个`shouldInstrument()`方法来检查参数在 TARGET_TYPES 集合中是否有类型以及`@Positive`注释:

```java
private boolean shouldInstrument(VariableTree parameter) {
    return TARGET_TYPES.contains(parameter.getType().toString())
      && parameter.getModifiers().getAnnotations().stream()
      .anyMatch(a -> Positive.class.getSimpleName()
        .equals(a.getAnnotationType().toString()));
}
```

然后我们将继续我们的`SampleJavacPlugin`类中的`finished()`方法，对满足我们条件的所有参数进行检查:

```java
public void finished(TaskEvent e) {
    if (e.getKind() != TaskEvent.Kind.PARSE) {
        return;
    }
    e.getCompilationUnit().accept(new TreeScanner<Void, Void>() {
        @Override
        public Void visitMethod(MethodTree method, Void v) {
            List<VariableTree> parametersToInstrument
              = method.getParameters().stream()
              .filter(SampleJavacPlugin.this::shouldInstrument)
              .collect(Collectors.toList());

              if (!parametersToInstrument.isEmpty()) {
                Collections.reverse(parametersToInstrument);
                parametersToInstrument.forEach(p -> addCheck(method, p, context));
            }
            return super.visitMethod(method, v);
        }
    }, null); 
```

在这个例子中，我们颠倒了参数列表，因为有可能不止一个参数被标记为`@Positive.`，因为每个检查都是作为第一个方法指令添加的，我们对它们进行 RTL 处理以确保正确的顺序。

### 5.2。如何检测

问题是“读取 AST”位于*公共* API 区域，而“修改 AST”操作如“添加空值检查”是一个*私有* `API`。

为了解决这个问题，**我们将通过一个 *TreeMaker* 实例创建新的 AST 元素。**

首先，我们需要获得一个`Context`实例:

```java
@Override
public void init(JavacTask task, String... args) {
    Context context = ((BasicJavacTask) task).getContext();
    // ...
}
```

然后，我们可以通过`TreeMarker.instance(Context)`方法获得`TreeMarker`对象。

现在我们可以构建新的 AST 元素，例如，如果通过调用`TreeMaker.If()`可以构建表达式，那么就构建一个*:*

```java
private static JCTree.JCIf createCheck(VariableTree parameter, Context context) {
    TreeMaker factory = TreeMaker.instance(context);
    Names symbolsTable = Names.instance(context);

    return factory.at(((JCTree) parameter).pos)
      .If(factory.Parens(createIfCondition(factory, symbolsTable, parameter)),
        createIfBlock(factory, symbolsTable, parameter), 
        null);
}
```

请注意，当我们的检查抛出异常时，我们希望显示正确的堆栈跟踪行。这就是为什么我们在用`factory.at(((JCTree) parameter).pos)`创建新元素之前调整 AST 工厂位置。

`createIfCondition()`方法构建了“`parameterId`<0”`if`条件:

```java
private static JCTree.JCBinary createIfCondition(TreeMaker factory, 
  Names symbolsTable, VariableTree parameter) {
    Name parameterId = symbolsTable.fromString(parameter.getName().toString());
    return factory.Binary(JCTree.Tag.LE, 
      factory.Ident(parameterId), 
      factory.Literal(TypeTag.INT, 0));
}
```

接下来，`createIfBlock()`方法构建一个返回`IllegalArgumentException:`的块

```java
private static JCTree.JCBlock createIfBlock(TreeMaker factory, 
  Names symbolsTable, VariableTree parameter) {
    String parameterName = parameter.getName().toString();
    Name parameterId = symbolsTable.fromString(parameterName);

    String errorMessagePrefix = String.format(
      "Argument '%s' of type %s is marked by @%s but got '", 
      parameterName, parameter.getType(), Positive.class.getSimpleName());
    String errorMessageSuffix = "' for it";

    return factory.Block(0, com.sun.tools.javac.util.List.of(
      factory.Throw(
        factory.NewClass(null, nil(), 
          factory.Ident(symbolsTable.fromString(
            IllegalArgumentException.class.getSimpleName())),
            com.sun.tools.javac.util.List.of(factory.Binary(JCTree.Tag.PLUS, 
            factory.Binary(JCTree.Tag.PLUS, 
              factory.Literal(TypeTag.CLASS, errorMessagePrefix), 
              factory.Ident(parameterId)), 
              factory.Literal(TypeTag.CLASS, errorMessageSuffix))), null))));
}
```

既然我们能够构建新的 AST 元素，我们需要将它们插入到由解析器准备的 AST 中。我们可以通过将 *public* A *PI* 元素转换为 *private* API 类型来实现这一点:

```java
private void addCheck(MethodTree method, VariableTree parameter, Context context) {
    JCTree.JCIf check = createCheck(parameter, context);
    JCTree.JCBlock body = (JCTree.JCBlock) method.getBody();
    body.stats = body.stats.prepend(check);
}
```

## 6。测试插件

我们需要能够测试我们的插件。它包括以下内容:

*   编译测试源
*   运行编译后的二进制文件，并确保它们的行为符合预期

为此，我们需要介绍几个辅助类。

`SimpleSourceFile`将给定源文件的文本暴露给`Javac`:

```java
public class SimpleSourceFile extends SimpleJavaFileObject {
    private String content;

    public SimpleSourceFile(String qualifiedClassName, String testSource) {
        super(URI.create(String.format(
          "file://%s%s", qualifiedClassName.replaceAll("\\.", "/"),
          Kind.SOURCE.extension)), Kind.SOURCE);
        content = testSource;
    }

    @Override
    public CharSequence getCharContent(boolean ignoreEncodingErrors) {
        return content;
    }
}
```

`SimpleClassFile`将编译结果保存为字节数组:

```java
public class SimpleClassFile extends SimpleJavaFileObject {

    private ByteArrayOutputStream out;

    public SimpleClassFile(URI uri) {
        super(uri, Kind.CLASS);
    }

    @Override
    public OutputStream openOutputStream() throws IOException {
        return out = new ByteArrayOutputStream();
    }

    public byte[] getCompiledBinaries() {
        return out.toByteArray();
    }

    // getters
}
```

确保编译器使用我们的字节码持有者:

```java
public class SimpleFileManager
  extends ForwardingJavaFileManager<StandardJavaFileManager> {

    private List<SimpleClassFile> compiled = new ArrayList<>();

    // standard constructors/getters

    @Override
    public JavaFileObject getJavaFileForOutput(Location location,
      String className, JavaFileObject.Kind kind, FileObject sibling) {
        SimpleClassFile result = new SimpleClassFile(
          URI.create("string://" + className));
        compiled.add(result);
        return result;
    }

    public List<SimpleClassFile> getCompiled() {
        return compiled;
    }
}
```

最后，所有这些都与内存编译有关:

```java
public class TestCompiler {
    public byte[] compile(String qualifiedClassName, String testSource) {
        StringWriter output = new StringWriter();

        JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
        SimpleFileManager fileManager = new SimpleFileManager(
          compiler.getStandardFileManager(null, null, null));
        List<SimpleSourceFile> compilationUnits 
          = singletonList(new SimpleSourceFile(qualifiedClassName, testSource));
        List<String> arguments = new ArrayList<>();
        arguments.addAll(asList("-classpath", System.getProperty("java.class.path"),
          "-Xplugin:" + SampleJavacPlugin.NAME));
        JavaCompiler.CompilationTask task 
          = compiler.getTask(output, fileManager, null, arguments, null,
          compilationUnits);

        task.call();
        return fileManager.getCompiled().iterator().next().getCompiledBinaries();
    }
}
```

之后，我们只需要运行二进制文件:

```java
public class TestRunner {

    public Object run(byte[] byteCode, String qualifiedClassName, String methodName,
      Class<?>[] argumentTypes, Object... args) throws Throwable {
        ClassLoader classLoader = new ClassLoader() {
            @Override
            protected Class<?> findClass(String name) throws ClassNotFoundException {
                return defineClass(name, byteCode, 0, byteCode.length);
            }
        };
        Class<?> clazz;
        try {
            clazz = classLoader.loadClass(qualifiedClassName);
        } catch (ClassNotFoundException e) {
            throw new RuntimeException("Can't load compiled test class", e);
        }

        Method method;
        try {
            method = clazz.getMethod(methodName, argumentTypes);
        } catch (NoSuchMethodException e) {
            throw new RuntimeException(
              "Can't find the 'main()' method in the compiled test class", e);
        }

        try {
            return method.invoke(null, args);
        } catch (InvocationTargetException e) {
            throw e.getCause();
        }
    }
}
```

测试可能是这样的:

```java
public class SampleJavacPluginTest {

    private static final String CLASS_TEMPLATE
      = "package com.baeldung.javac;\n\n" +
        "public class Test {\n" +
        "    public static %1$s service(@Positive %1$s i) {\n" +
        "        return i;\n" +
        "    }\n" +
        "}\n" +
        "";

    private TestCompiler compiler = new TestCompiler();
    private TestRunner runner = new TestRunner();

    @Test(expected = IllegalArgumentException.class)
    public void givenInt_whenNegative_thenThrowsException() throws Throwable {
        compileAndRun(double.class,-1);
    }

    private Object compileAndRun(Class<?> argumentType, Object argument) 
      throws Throwable {
        String qualifiedClassName = "com.baeldung.javac.Test";
        byte[] byteCode = compiler.compile(qualifiedClassName, 
          String.format(CLASS_TEMPLATE, argumentType.getName()));
        return runner.run(byteCode, qualifiedClassName, 
        "service", new Class[] {argumentType}, argument);
    }
}
```

这里我们用一个`service()`方法编译一个`Test`类，该方法有一个用`@Positive.`注释的参数，然后，我们通过为方法参数设置一个双精度值-1 来运行`Test`类。

作为用我们的插件运行编译器的结果，测试将为负参数抛出一个`IllegalArgumentException`。

## 7。结论

在本文中，我们展示了创建、测试和运行 Java 编译器插件的完整过程。

这些例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220701021033/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-sun)