# 列出特定类装入器中装入的所有类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-classes-class-loader>

## 1.概观

在本教程中，我们将分析技术**，使用 [Java `Instrumentation` API](/web/20220706104710/https://www.baeldung.com/java-instrumentation) 列出 Java 中由特定类加载器**加载的所有类。我们还将看到如何创建和加载一个 Java 代理来获取一个`Instrumentation`实例，并调用所需的方法来完成我们的任务。

## 2.Java 中的类装入器

类加载器是 JRE (Java 运行时环境)不可或缺的一部分。他们的工作是**将类动态加载到 Java 虚拟机**中。换句话说，当应用程序需要时，它们按需将类加载到内存中。关于 [Java 类加载器](/web/20220706104710/https://www.baeldung.com/java-classloaders)的文章讨论了它们的不同类型，并提供了它们如何工作的详细理解。

## 3.使用`Instrumentation API`

`Instrumentation`接口提供了`[getInitiatedClasses(Classloader loader)](https://web.archive.org/web/20220706104710/https://docs.oracle.com/en/java/javase/14/docs/api/java.instrument/java/lang/instrument/Instrumentation.html#getInitiatedClasses(java.lang.ClassLoader))`方法，该方法可以被**调用以** **返回一个数组，该数组包含由特定加载器**加载的所有类。让我们看看这是如何工作的。

首先，我们需要创建并加载一个代理来获取一个`Instrumentation`接口的实例。Java 代理是在 JVM (Java 虚拟机)上运行程序的工具。

换句话说，为了收集数据，它可以添加或修改方法的字节码。我们将需要一个`agent`来获得`Instrumentation`实例的句柄，并调用所需的方法。

[创建和加载代理](/web/20220706104710/https://www.baeldung.com/java-instrumentation)有多种方式。在本教程中，我们将使用静态加载方法，使用`premain`方法和`-javaagent`选项。

### 3.1.创建 Java 代理

要创建一个 Java 代理，我们需要**定义** **`premain`** **方法，在代理加载**时`Instrumentation`实例将被传递给该方法。现在让我们创建`ListLoadedClassesAgent`类:

```java
public class ListLoadedClassesAgent {

    private static Instrumentation instrumentation;

    public static void premain(String agentArgs, Instrumentation instrumentation) {
        ListLoadedClassesAgent.instrumentation = instrumentation;
    }
}
```

### 3.2.定义`listLoadedClasses`方法

除了定义代理之外，我们还将定义并公开一个静态方法，为给定的类装入器返回一组装入的类。

注意，**如果我们将带有`null `值的类加载器传递给`getInitiatedClasses`方法，它将返回由引导类加载器**加载的类。

让我们看看实际运行的代码:

```java
public static Class<?>[] listLoadedClasses(String classLoaderType) {
    return instrumentation.getInitiatedClasses(
      getClassLoader(classLoaderType));
}

private static ClassLoader getClassLoader(String classLoaderType) {
    ClassLoader classLoader = null;
    switch (classLoaderType) {
        case "SYSTEM":
            classLoader = ClassLoader.getSystemClassLoader();
            break;
        case "EXTENSION":
            classLoader = ClassLoader.getSystemClassLoader().getParent();
            break;
        case "BOOTSTRAP":
            break;
        default:
            break;
    }
    return classLoader;
}
```

注意，如果我们使用 Java 9 或更高版本，我们可以使用`getPlatformClassLoader` 方法。这将列出由平台类加载器加载的类。在这种情况下，开关盒还将包含:

```java
case "PLATFORM":
    classLoader = ClassLoader.getPlatformClassLoader();
    break;
```

### 3.3.创建代理清单文件

现在，让我们创建一个清单文件`MANIFEST.MF`，它具有合适的属性供代理运行，包括:

```java
Premain-Class: com.baeldung.loadedclasslisting.ListLoadedClassesAgent
```

代理 JAR 文件的清单属性的完整列表可以在`[java.lang.instrument](https://web.archive.org/web/20220706104710/https://docs.oracle.com/en/java/javase/14/docs/api/java.instrument/java/lang/instrument/package-summary.html)`包的官方文档中找到。

### 3.4.加载代理并运行应用程序

现在让我们加载代理并运行应用程序。首先，我们需要代理 JAR 文件和包含`Premain-Class`信息的清单文件。此外，我们需要应用程序 JAR 文件和包含`Main-Class` 信息的清单文件。包含`main`方法的`Launcher class`将启动我们的应用程序。然后我们将能够打印由不同类型的类装入器装入的类:

```java
public class Launcher {

    public static void main(String[] args) {
        printClassesLoadedBy("BOOTSTRAP");
        printClassesLoadedBy("SYSTEM");
        printClassesLoadedBy("EXTENSION");
    }

    private static void printClassesLoadedBy(String classLoaderType) {
        System.out.println(classLoaderType + " ClassLoader : ");
        Class<?>[] classes = ListLoadedClassesAgent.listLoadedClasses(classLoaderType);
        Arrays.asList(classes)
            .forEach(clazz -> System.out.println(clazz.getCanonicalName()));
    }
}
```

接下来，让我们静态地[加载 Java 代理](/web/20220706104710/https://www.baeldung.com/java-instrumentation#loading-a-java-agent)并启动我们的应用程序:

```java
java -javaagent:agent.jar -jar app.jar
```

运行以上命令后，我们将看到输出:

```java
BOOTSTRAP ClassLoader :
java.lang.ClassValue.Entry[]
java.util.concurrent.ConcurrentHashMap.Segment
java.util.concurrent.ConcurrentHashMap.Segment[]
java.util.StringTokenizer
..............
SYSTEM ClassLoader : 
java.lang.Object[]
java.lang.Object[][]
java.lang.Class
java.lang.Class[]
..............
EXTENSION ClassLoader :
byte[]
char[]
int[]
int[][]
short[] 
```

## 4.结论

在本教程中，我们学习了列出特定类装入器中装入的所有类的技术。

首先，我们创建了 Java 代理。之后，我们使用 Java `Instrumentation` API 定义了列出加载的类的方法。最后，我们创建代理清单文件，加载代理，并运行我们的应用程序。

和往常一样，这个例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220706104710/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm-2)