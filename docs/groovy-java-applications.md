# 将 Groovy 集成到 Java 应用程序中

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-java-applications>

## 1.介绍

在本教程中，我们将探索将 Groovy 集成到 Java 应用程序中的最新技术。

## 2.关于 Groovy 的几句话

Groovy 编程语言是一种强大的、**可选类型的动态语言**。它得到了 Apache 软件基金会和 Groovy 社区的支持，有 200 多名开发人员做出了贡献。

它可以用来构建一个完整的应用程序，创建一个模块或一个与我们的 Java 代码交互的附加库，或者运行动态评估和编译的脚本。

更多信息，请阅读[Groovy 语言介绍](/web/20220626203305/https://www.baeldung.com/groovy-language)或前往[官方文档](https://web.archive.org/web/20220626203305/http://groovy-lang.org/)。

## 3.Maven 依赖性

在撰写本文时，最新的稳定版本是 2.5.7，而 Groovy 2.6 和 3.0(均始于 2017 年秋季)仍处于 alpha 阶段。

类似于 Spring Boot，**我们只需要包含`[groovy-all](https://web.archive.org/web/20220626203305/https://search.maven.org/search?q=g:org.codehaus.groovy%20a:groovy-all)` pom 来添加我们可能需要的所有依赖项**，而不用担心它们的版本:

```java
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-all</artifactId>
    <version>${groovy.version}</version>
    <type>pom</type>
</dependency>
```

## 4.联合编译

在深入了解如何配置 Maven 的细节之前，我们需要了解我们在处理什么。

我们的代码将包含 Java 和 Groovy 文件。Groovy 在查找 Java 类时不会有任何问题，但是如果我们想让 Java 查找 Groovy 类和方法呢？

联合编译来拯救我们了！

**联合编译是一个过程，旨在用一个 Maven 命令编译同一个项目中的 Java 和 Groovy** 文件。

通过联合编译，Groovy 编译器将:

*   解析源文件
*   根据实现，创建与 Java 编译器兼容的存根
*   调用 Java 编译器来编译存根和 Java 源代码——这样 Java 类就可以找到 Groovy 依赖项
*   编译 Groovy 源代码——现在我们的 Groovy 源代码可以找到它们的 Java 依赖项了

根据实现它的插件，我们可能需要将文件分离到特定的文件夹中，或者告诉编译器在哪里可以找到它们。

如果没有联合编译，Java 源文件会像 Groovy 源文件一样被编译。有时这可能行得通，因为大多数 Java 1.7 语法与 Groovy 兼容，但语义会有所不同。

## 5.Maven 编译器插件

有几个编译器插件支持联合编译，它们各有优缺点。

Maven 最常用的两个是 Groovy——Eclipse Maven 和 GMaven+。

### 5.1.Groovy-Eclipse Maven 插件

[Groovy-Eclipse Maven 插件](https://web.archive.org/web/20220626203305/https://github.com/groovy/groovy-eclipse/wiki/Groovy-Eclipse-Maven-plugin#why-another-groovy-compiler-for-maven-what-about-gmaven) **通过避免存根生成**简化了联合编译，这对于 GMaven `+`等其他编译器来说仍然是一个必不可少的步骤，但它呈现出一些配置怪癖。

为了能够检索最新的编译器工件，我们必须添加 Maven Bintray 存储库:

```java
<pluginRepositories>
    <pluginRepository>
        <id>bintray</id>
        <name>Groovy Bintray</name>
        <url>https://dl.bintray.com/groovy/maven</url>
        <releases>
            <!-- avoid automatic updates -->
            <updatePolicy>never</updatePolicy>
        </releases>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>
```

然后，在插件部分，**我们告诉 Maven 编译器它必须使用哪个 Groovy 编译器版本。**

事实上，我们将使用的插件——[Maven 编译器插件](/web/20220626203305/https://www.baeldung.com/maven-compiler-plugin)——实际上并不编译，而是将工作委托给[工件](https://web.archive.org/web/20220626203305/https://search.maven.org/search?q=g:org.codehaus.groovy%20a:groovy-eclipse-batch):

```java
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.0</version>
    <configuration>
        <compilerId>groovy-eclipse-compiler</compilerId>
        <source>${java.version}</source>
        <target>${java.version}</target>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy-eclipse-compiler</artifactId>
            <version>3.3.0-01</version>
        </dependency>
        <dependency>
            <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy-eclipse-batch</artifactId>
            <version>${groovy.version}-01</version>
        </dependency>
    </dependencies>
</plugin>
```

`groovy-all`依赖版本应该与编译器版本相匹配。

最后，我们需要配置我们的源代码自动发现:默认情况下，编译器会查看诸如`src/main/java`和`src/main/groovy,`这样的文件夹，但是**如果我们的 java 文件夹是空的，编译器就不会查找我们的 groovy 源代码**。

同样的机制也适用于我们的测试。

为了强制文件发现，我们可以在`src/main/java` 和`src/test/java`中添加任何文件，或者简单地添加 [`groovy-eclipse-compiler`插件](https://web.archive.org/web/20220626203305/https://search.maven.org/search?q=g:org.codehaus.groovy%20a:groovy-eclipse-compiler):

```java
<plugin>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-eclipse-compiler</artifactId>
    <version>3.3.0-01</version>
    <extensions>true</extensions>
</plugin>
```

`<extension>`部分是强制的，让插件添加额外的构建阶段和目标，包含两个 Groovy 源文件夹。

### 5.2.GMavenPlus 插件

GMavenPlus 插件的名字可能与旧的 GMaven 插件相似，但作者没有创建一个简单的补丁，而是努力**简化编译器并将其与特定的 Groovy 版本**分离。

为了做到这一点，插件将自己与编译器插件的标准指南分离开来。

GMavenPlus 编译器**增加了对当时其他编译器**还没有的功能的支持，比如 [invokedynamic](https://web.archive.org/web/20220626203305/http://groovy-lang.org/indy.html) ，交互式 shell 控制台和 Android。

另一方面，它也带来了一些问题:

*   它**修改 Maven 的源目录**以包含 Java 和 Groovy 源，但不包含 Java 存根
*   它**要求我们管理存根**,如果我们没有以适当的目标删除它们

为了配置我们的项目，我们需要添加 [gmavenplus-plugin](https://web.archive.org/web/20220626203305/https://search.maven.org/search?q=g:org.codehaus.gmavenplus%20a:gmavenplus-plugin) :

```java
<plugin>
    <groupId>org.codehaus.gmavenplus</groupId>
    <artifactId>gmavenplus-plugin</artifactId>
    <version>1.7.0</version>
    <executions>
        <execution>
            <goals>
                <goal>execute</goal>
                <goal>addSources</goal>
                <goal>addTestSources</goal>
                <goal>generateStubs</goal>
                <goal>compile</goal>
                <goal>generateTestStubs</goal>
                <goal>compileTests</goal>
                <goal>removeStubs</goal>
                <goal>removeTestStubs</goal>
            </goals>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy-all</artifactId>
            <!-- any version of Groovy \>= 1.5.0 should work here -->
            <version>2.5.6</version>
            <scope>runtime</scope>
            <type>pom</type>
        </dependency>
    </dependencies>
</plugin>
```

为了测试这个插件，我们在示例中创建了第二个 pom 文件`gmavenplus-pom.xml `。

### 5.3.用 Eclipse-Maven 插件编译

现在一切都配置好了，我们终于可以构建我们的类了。

在我们提供的例子中，我们在源文件夹`src/main/java`中创建了一个简单的 Java 应用程序，在`src/main/groovy`中创建了一些 Groovy 脚本，我们可以在其中创建 Groovy 类和脚本。

让我们用 Eclipse-Maven 插件构建一切:

```java
$ mvn clean compile
...
[INFO] --- maven-compiler-plugin:3.8.0:compile (default-compile) @ core-groovy-2 ---
[INFO] Changes detected - recompiling the module!
[INFO] Using Groovy-Eclipse compiler to compile both Java and Groovy files
...
```

这里我们看到 **Groovy 正在编译一切**。

### 5.4.使用 GMavenPlus 编译

GMavenPlus 显示了一些差异:

```java
$ mvn -f gmavenplus-pom.xml clean compile
...
[INFO] --- gmavenplus-plugin:1.7.0:generateStubs (default) @ core-groovy-2 ---
[INFO] Using Groovy 2.5.7 to perform generateStubs.
[INFO] Generated 2 stubs.
[INFO]
...
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ core-groovy-2 ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 3 source files to XXX\Baeldung\TutorialsRepo\core-groovy-2\target\classes
[INFO]
...
[INFO] --- gmavenplus-plugin:1.7.0:compile (default) @ core-groovy-2 ---
[INFO] Using Groovy 2.5.7 to perform compile.
[INFO] Compiled 2 files.
[INFO]
...
[INFO] --- gmavenplus-plugin:1.7.0:removeStubs (default) @ core-groovy-2 ---
[INFO]
...
```

我们马上注意到 GMavenPlus 经过了额外的步骤:

1.  生成存根，每个 groovy 文件一个
2.  编译 Java 文件——存根和 Java 代码
3.  编译 Groovy 文件

通过生成存根，GMavenPlus 继承了一个弱点，在过去几年里，当使用联合编译时，这个弱点让开发人员非常头疼。

在理想的场景中，一切都很好，但是引入更多的步骤，我们也有更多的失败点:例如，构建可能在清理存根之前就失败了。

如果发生这种情况，遗留下来的旧存根可能会混淆我们的 IDE，这将显示编译错误，而我们知道一切都应该是正确的。

只有干净的构建才能避免痛苦而漫长的政治迫害。

### 5.5.将依赖项打包到 Jar 文件中

为了让 **[从命令行](/web/20220626203305/https://www.baeldung.com/executable-jar-with-maven)**将程序作为 jar 运行，我们添加了[`maven-assembly-plugin`](https://web.archive.org/web/20220626203305/https://search.maven.org/search?q=g:org.apache.maven.plugins%20a:maven-assembly-plugin)，它将所有 Groovy 依赖项包含在一个“胖 jar”中，这个“胖 jar”以属性`descriptorRef:`中定义的后缀命名

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <!-- get all project dependencies -->
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <!-- MainClass in mainfest make a executable jar -->
        <archive>
            <manifest>
                <mainClass>com.baeldung.MyJointCompilationApp</mainClass>
            </manifest>
        </archive>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <!-- bind to the packaging phase -->
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

编译完成后，我们可以使用以下命令运行代码:

```java
$ java -jar target/core-groovy-2-1.0-SNAPSHOT-jar-with-dependencies.jar com.baeldung.MyJointCompilationApp
```

## 6.动态加载 Groovy 代码

Maven 编译允许我们在项目中包含 Groovy 文件，并从 Java 中引用它们的类和方法。

但是，如果我们想在运行时改变逻辑，这是不够的:编译在运行时阶段之外运行，所以**我们仍然必须重启我们的应用程序，以便看到我们的改变**。

为了利用 Groovy 的动态能力(和风险),我们需要探索在应用程序已经运行时加载文件的技术。

### 6.1.`GroovyClassLoader`

为了实现这一点，我们需要能够解析文本或文件格式的源代码并生成结果类对象的`GroovyClassLoader,` 。

**当源文件是一个文件时，编译结果也被缓存**，以避免当我们向加载器请求同一个类的多个实例时的开销。

相反，直接来自 **`String`对象的脚本不会被缓存**，因此多次调用同一个脚本仍然会导致内存泄漏。

是其他集成系统的基础。

实现相对简单:

```java
private final GroovyClassLoader loader;

private Double addWithGroovyClassLoader(int x, int y) 
  throws IllegalAccessException, InstantiationException, IOException {
    Class calcClass = loader.parseClass(
      new File("src/main/groovy/com/baeldung/", "CalcMath.groovy"));
    GroovyObject calc = (GroovyObject) calcClass.newInstance();
    return (Double) calc.invokeMethod("calcSum", new Object[] { x, y });
}

public MyJointCompilationApp() {
    loader = new GroovyClassLoader(this.getClass().getClassLoader());
    // ...
} 
```

### 6.2.`GroovyShell`

Shell 脚本加载器`parse()`方法接受文本或文件格式的源代码，而**生成一个`Script`类的实例。**

这个实例从`Script`继承了`run()`方法，该方法从上到下执行整个文件，并返回最后一行执行的结果。

如果我们愿意，我们也可以在代码中扩展`Script`,并覆盖默认实现来直接调用我们的内部逻辑。

调用`Script.run()` 的实现如下所示:

```java
private Double addWithGroovyShellRun(int x, int y) throws IOException {
    Script script = shell.parse(new File("src/main/groovy/com/baeldung/", "CalcScript.groovy"));
    return (Double) script.run();
}

public MyJointCompilationApp() {
    // ...
    shell = new GroovyShell(loader, new Binding());
    // ...
} 
```

请注意，`run()`不接受参数，所以我们需要在文件中添加一些全局变量，并通过`Binding`对象初始化它们。

当这个对象在`GroovyShell`初始化中被传递时，变量被所有的`Script`实例共享。

如果我们喜欢更细粒度的控制，我们可以使用`invokeMethod()`，它可以通过反射访问我们自己的方法，并直接传递参数。

让我们来看看这个实现:

```java
private final GroovyShell shell;

private Double addWithGroovyShell(int x, int y) throws IOException {
    Script script = shell.parse(new File("src/main/groovy/com/baeldung/", "CalcScript.groovy"));
    return (Double) script.invokeMethod("calcSum", new Object[] { x, y });
}

public MyJointCompilationApp() {
    // ...
    shell = new GroovyShell(loader, new Binding());
    // ...
} 
```

在幕后，`GroovyShell`依赖于`GroovyClassLoader`来编译和缓存生成的类，因此前面解释的相同规则以同样的方式适用。

### 6.3.`GroovyScriptEngine`

`GroovyScriptEngine`类尤其适用于那些**依赖于重新加载脚本及其依赖项**的应用程序。

虽然我们有这些额外的功能，但实现只有一些小的不同:

```java
private final GroovyScriptEngine engine;

private void addWithGroovyScriptEngine(int x, int y) throws IllegalAccessException,
  InstantiationException, ResourceException, ScriptException {
    Class<GroovyObject> calcClass = engine.loadScriptByName("CalcMath.groovy");
    GroovyObject calc = calcClass.newInstance();
    Object result = calc.invokeMethod("calcSum", new Object[] { x, y });
    LOG.info("Result of CalcMath.calcSum() method is {}", result);
}

public MyJointCompilationApp() {
    ...
    URL url = null;
    try {
        url = new File("src/main/groovy/com/baeldung/").toURI().toURL();
    } catch (MalformedURLException e) {
        LOG.error("Exception while creating url", e);
    }
    engine = new GroovyScriptEngine(new URL[] {url}, this.getClass().getClassLoader());
    engineFromFactory = new GroovyScriptEngineFactory().getScriptEngine(); 
}
```

这一次我们必须配置源根目录，我们只使用它的名字来引用脚本，这样更简洁一些。

查看`loadScriptByName`方法内部，我们可以立即看到检查`isSourceNewer`，其中引擎检查当前缓存中的源是否仍然有效。

**每次我们的文件改变时，`GroovyScriptEngine`会自动重新加载那个特定的文件和依赖于它的所有类。**

虽然这是一个方便而强大的功能，但它可能会导致一个非常危险的副作用:**多次重新加载大量文件将会导致 CPU 开销，而不会有任何警告。**

如果发生这种情况，我们可能需要实现自己的缓存机制来处理这个问题。

### 6.4.`GroovyScriptEngineFactory` (JSR-223)

[JSR-223](https://web.archive.org/web/20220626203305/https://jcp.org/aboutJava/communityprocess/final/jsr223/index.html) 提供了一个**标准 API，用于调用 Java 6 以来的脚本框架**。

实现看起来很相似，尽管我们回到通过完整的文件路径加载:

```java
private final ScriptEngine engineFromFactory;

private void addWithEngineFactory(int x, int y) throws IllegalAccessException, 
  InstantiationException, javax.script.ScriptException, FileNotFoundException {
    Class calcClas = (Class) engineFromFactory.eval(
      new FileReader(new File("src/main/groovy/com/baeldung/", "CalcMath.groovy")));
    GroovyObject calc = (GroovyObject) calcClas.newInstance();
    Object result = calc.invokeMethod("calcSum", new Object[] { x, y });
    LOG.info("Result of CalcMath.calcSum() method is {}", result);
}

public MyJointCompilationApp() {
    // ...
    engineFromFactory = new GroovyScriptEngineFactory().getScriptEngine();
}
```

如果我们将我们的应用程序与几种脚本语言集成在一起，这很好，但是它的功能集更加有限。比如**它不支持类重载**。因此，如果我们只与 Groovy 集成，那么坚持使用早期的方法可能会更好。

## 7.动态编译的陷阱

使用上面的任何一种方法，我们可以创建一个应用程序，**从 jar 文件**之外的特定文件夹中读取脚本或类。

这将给予我们在系统运行时添加新特性的灵活性(除非我们在 Java 部分需要新代码)，从而实现某种连续交付开发。

但是要小心这把双刃剑:我们现在需要非常小心地保护自己免受编译时和运行时都可能发生的**失败，事实上确保我们的代码安全地失败。**

## 8.在 Java 项目中运行 Groovy 的陷阱

### 8.1.表演

我们都知道，当一个系统需要非常高的性能时，有一些黄金法则可以遵循。

可能对我们的项目影响更大的两个因素是:

*   避免反射
*   最小化字节码指令的数量

特别是反射，由于检查类、字段、方法、方法参数等等的过程，它是一个开销很大的操作。

如果我们分析从 Java 到 Groovy 的方法调用，例如，当运行示例`addWithCompiledClasses`时，`.calcSum`和实际 Groovy 方法的第一行之间的操作栈看起来像:

```java
calcSum:4, CalcScript (com.baeldung)
addWithCompiledClasses:43, MyJointCompilationApp (com.baeldung)
addWithStaticCompiledClasses:95, MyJointCompilationApp (com.baeldung)
main:117, App (com.baeldung)
```

这与 Java 是一致的。当我们转换加载器返回的对象并调用它的方法时，也会发生同样的情况。

然而，这就是`invokeMethod`调用的作用:

```java
calcSum:4, CalcScript (com.baeldung)
invoke0:-1, NativeMethodAccessorImpl (sun.reflect)
invoke:62, NativeMethodAccessorImpl (sun.reflect)
invoke:43, DelegatingMethodAccessorImpl (sun.reflect)
invoke:498, Method (java.lang.reflect)
invoke:101, CachedMethod (org.codehaus.groovy.reflection)
doMethodInvoke:323, MetaMethod (groovy.lang)
invokeMethod:1217, MetaClassImpl (groovy.lang)
invokeMethod:1041, MetaClassImpl (groovy.lang)
invokeMethod:821, MetaClassImpl (groovy.lang)
invokeMethod:44, GroovyObjectSupport (groovy.lang)
invokeMethod:77, Script (groovy.lang)
addWithGroovyShell:52, MyJointCompilationApp (com.baeldung)
addWithDynamicCompiledClasses:99, MyJointCompilationApp (com.baeldung)
main:118, MyJointCompilationApp (com.baeldung)
```

在这种情况下，我们可以体会到 Groovy 的强大背后真正的东西:`MetaClass`。

一个 **`MetaClass`定义了任何给定的 Groovy 或 Java 类的行为，因此每当有动态操作要执行**时，Groovy 就会查看它，以便找到目标方法或字段。一旦找到，标准反射流就执行它。

一个调用方法打破了两条黄金法则！

如果我们需要处理数百个动态 Groovy 文件，**我们如何调用我们的方法将在我们的系统中产生巨大的性能差异**。

### 8.2.找不到方法或属性

如前所述，如果我们想在 CD 生命周期中**部署新版本的 Groovy 文件**，我们需要**将它们视为独立于我们核心系统的 API** 。

这意味着要实施**多重故障安全检查和代码设计限制**,这样我们新加入的开发人员就不会因为一次错误的推动而破坏生产系统。

每个的例子是:有一个 CI 管道和使用方法弃用而不是删除。

如果不这样会怎么样？由于缺少方法和错误的参数计数和类型，我们会遇到可怕的异常。

如果我们认为编译可以拯救我们，让我们看看 Groovy 脚本的方法`calcSum2()`:

```java
// this method will fail in runtime
def calcSum2(x, y) {
    // DANGER! The variable "log" may be undefined
    log.info "Executing $x + $y"
    // DANGER! This method doesn't exist!
    calcSum3()
    // DANGER! The logged variable "z" is undefined!
    log.info("Logging an undefined variable: $z")
}
```

通过查看整个文件，我们立即发现了两个问题:方法`calcSum3()`和变量`z`没有在任何地方定义。

即便如此，无论是在 Maven 中静态地还是在 GroovyClassLoader 中动态地，脚本都成功地编译了，甚至没有一个警告。

只有当我们试图调用它时，它才会失败。

只有当我们的 Java 代码直接引用`calcSum3()`时，Maven 的静态编译才会显示一个错误，就像我们在`addWithCompiledClasses()`方法中做的那样，在转换`GroovyObject`之后，但是如果我们使用反射，它仍然是无效的。

## 9.结论

在本文中，我们探讨了如何在 Java 应用程序中集成 Groovy，研究了不同的集成方法以及混合语言中可能遇到的一些问题。

像往常一样，示例中使用的源代码可以在 [GitHub](https://web.archive.org/web/20220626203305/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-2) 上找到。