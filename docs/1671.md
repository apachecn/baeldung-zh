# 从类中获取 JAR 文件的完整路径

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-full-path-of-jar-from-class>

## 1.概观

JAR 文件是 Java 档案。在构建 Java 应用程序时，我们可能会包含各种 JAR 文件作为库。

在本教程中，我们将探索如何从给定的类中找到 JAR 文件及其完整路径。

## 2.问题简介

假设我们在运行时有一个`Class`对象。我们的目标是找出这个类属于哪个 JAR 文件。

举个例子也许能帮助我们快速理解这个问题。假设我们有了[番石榴](/web/20220824180106/https://www.baeldung.com/guava-guide)的`Ascii`类的类实例。我们想创建一个方法来找出保存`Ascii`类的 JAR 文件的完整路径。

我们将主要讨论两种不同的方法来获取 JAR 文件的完整路径。此外，我们将讨论它们的利弊。

为了简单起见，我们将通过单元测试断言来验证结果。

接下来，让我们看看他们的行动。

## 3.使用`getProtectionDomain()`方法

Java 的类对象提供了 [`getProtectionDomain()`](https://web.archive.org/web/20220824180106/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Class.html#getProtectionDomain()) 方法来获取`ProtectionDomain`对象。然后，我们可以通过`ProtectionDomain`对象得到 [`CodeSource`](https://web.archive.org/web/20220824180106/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/CodeSource.html) 。`CodeSource`实例将是我们正在寻找的 JAR 文件。此外， [`CodeSource.getLocation()`](https://web.archive.org/web/20220824180106/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/CodeSource.html#getLocation()) 方法为我们提供了 JAR 文件的 [URL](/web/20220824180106/https://www.baeldung.com/java-url-vs-uri) 对象。最后，我们可以使用 [`Paths`](/web/20220824180106/https://www.baeldung.com/java-nio-2-path) 类来获取 JAR 文件的完整路径。

### 3.1.实施`byGetProtectionDomain()`方法

如果我们将上面提到的所有步骤都包含在一个方法中，几行代码就可以完成这项工作:

```
public class JarFilePathResolver {
    String byGetProtectionDomain(Class clazz) throws URISyntaxException {
        URL url = clazz.getProtectionDomain().getCodeSource().getLocation();
        return Paths.get(url.toURI()).toString();
    }
} 
```

接下来，让我们以 Guava `Ascii`类为例来测试我们的方法是否如预期的那样工作:

```
String jarPath = jarFilePathResolver.byGetProtectionDomain(Ascii.class);
assertThat(jarPath).endsWith(".jar").contains("guava");
assertThat(new File(jarPath)).exists(); 
```

正如我们所看到的，我们已经通过两个断言验证了返回的`jarPath`:

*   首先，路径应该指向 Guava JAR 文件
*   如果`jarPath`是一个有效的完整路径，我们可以从`jarPath,` 创建一个`File`对象，这个文件应该存在

如果我们进行测试，就会通过。所以`byGetProtectionDomain()`方法如预期的那样工作。

### 3.2.`getProtectionDomain()`方法的一些限制

如上面的代码所示，我们的`byGetProtectionDomain()`方法非常简洁明了。然而，如果我们阅读`getProtectionDomain()`方法的 JavaDoc，它说**`getProtectionDomain()`方法可能抛出`SecurityException`** 。

我们已经编写了一个单元测试，测试通过了。这是因为我们正在本地开发环境中测试该方法。在我们的例子中，番石榴罐子位于我们本地的 [Maven](/web/20220824180106/https://www.baeldung.com/maven) 仓库中。所以没有提出`SecurityException`。

但是，**一些平台，比如 [Java/OpenWebStart](/web/20220824180106/https://www.baeldung.com/java-web-start) 和一些应用服务器，可能会禁止通过调用`getProtectionDomain()`方法来获取`ProtectionDomain`对象。因此，如果我们将应用程序部署到这些平台上，我们的方法将失败并抛出`SecurityException.`**

接下来，让我们看看另一种获取 JAR 文件完整路径的方法。

## 4.使用`getResource()`方法

我们知道，我们调用 [`Class.getResource`](https://web.archive.org/web/20220824180106/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Class.html#getResource(java.lang.String)) ()方法来获取类的资源的`URL`对象。所以让我们从这个方法开始，最终解析相应 JAR 文件的完整路径。

### 4.1.实施`byGetResource()`方法

让我们先看看实现，然后理解它是如何工作的:

```
String byGetResource(Class clazz) {
    URL classResource = clazz.getResource(clazz.getSimpleName() + ".class");
    if (classResource == null) {
        throw new RuntimeException("class resource is null");
    }
    String url = classResource.toString();
    if (url.startsWith("jar:file:")) {
        // extract 'file:......jarName.jar' part from the url string
        String path = url.replaceAll("^jar:(file:.*[.]jar)!/.*", "$1");
        try {
            return Paths.get(new URL(path).toURI()).toString();
        } catch (Exception e) {
            throw new RuntimeException("Invalid Jar File URL String");
        }
    }
    throw new RuntimeException("Invalid Jar File URL String");
} 
```

与`byGetProtectionDomain`方法相比，上面的方法看起来更复杂。但事实上，这也很容易理解。

接下来，让我们快速浏览一下这个方法，了解它是如何工作的。为了简单起见，我们对各种异常情况抛出`RuntimeException`。

### 4.2.了解它是如何工作的

首先，我们调用`Class.getResource(className)`方法来获取给定类的 URL。

**如果该类来自本地文件系统上的 JAR 文件，URL 字符串应该是这样的格式**:

```
jar:file:/FULL/PATH/TO/jarName.jar!/PACKAGE/HIERARCHY/TO/CLASS/className.class
```

例如，下面是 Linux 系统上 Guava 的`Ascii`类的 URL 字符串:

```
jar:file:/home/kent/.m2/repository/com/google/guava/guava/31.0.1-jre/guava-31.0.1-jre.jar!/com/google/common/base/Ascii.class
```

正如我们所看到的，JAR 文件的完整路径位于 URL 字符串的中间。

**由于不同操作系统上的文件 URL 格式可能不同，我们将提取“`file:…..jar`”部分，将其转换回一个`URL`对象，并使用`Paths`类获取路径作为`String`。**

我们构建一个正则表达式，并使用`String`的`[replaceAll()](/web/20220824180106/https://www.baeldung.com/string/replace-all)`方法提取我们需要的部分:`String path = url.replaceAll(“^jar:(file:.*[.]jar)!/.*”, “$1”);`

接下来，类似于`byGetProtectionDomain()`方法，我们使用`Paths`类获得最终结果。

现在，让我们创建一个测试来验证我们的方法是否适用于 Guava 的`Ascii`类:

```
String jarPath = jarFilePathResolver.byGetResource(Ascii.class);
assertThat(jarPath).endsWith(".jar").contains("guava");
assertThat(new File(jarPath)).exists(); 
```

如果我们试一试，考试就会通过。

## 5.结合两种方法

到目前为止，我们已经看到了解决这个问题的两种方法。`byGetProtectionDomain`方法简单可靠，但是由于安全限制，在某些平台上可能会失败。

另一方面，`byGetResource`方法没有安全问题。然而，我们需要做更多的手动操作，比如处理不同的异常情况和使用 regex 提取 JAR 文件的 URL 字符串。

### 5.1.实施`getJarFilePath()`方法

我们可以把这两种方法结合起来。首先，让我们尝试用`byGetProtectionDomain()`解析 JAR 文件的路径。如果失败，我们调用`byGetResource()`方法作为后备:

```
String getJarFilePath(Class clazz) {
    try {
        return byGetProtectionDomain(clazz);
    } catch (Exception e) {
        // cannot get jar file path using byGetProtectionDomain
        // Exception handling omitted
    }
    return byGetResource(clazz);
} 
```

### 5.2.测试`getJarFilePath()`方法

为了在我们的本地开发环境中模拟`byGetProtectionDomain()`抛出`SecurityException`，让我们添加 [Mockito](/web/20220824180106/https://www.baeldung.com/mockito-series) 依赖项，**使用 [`@Spy`](/web/20220824180106/https://www.baeldung.com/mockito-spy) 注释**部分模拟`JarFilePathResolver`:

```
@ExtendWith(MockitoExtension.class)
class JarFilePathResolverUnitTest {
    @Spy
    JarFilePathResolver jarFilePathResolver;
    ...
}
```

接下来，我们先测试一下`getProtectionDomain()`方法不抛出`SecurityException`的场景:

```
String jarPath = jarFilePathResolver.getJarFilePath(Ascii.class);
assertThat(jarPath).endsWith(".jar").contains("guava");
assertThat(new File(jarPath)).exists();
verify(jarFilePathResolver, times(1)).byGetProtectionDomain(Ascii.class);
verify(jarFilePathResolver, never()).byGetResource(Ascii.class); 
```

如上面的代码所示，除了测试路径是否有效之外，我们还验证了如果我们可以通过`byGetProtectionDomain()`方法获得 JAR 文件的路径，那么就不应该调用`byGetResource()`方法。

当然，如果`byGetProtectionDomain()`抛出`SecurityException`，这两个方法会被调用一次:

```
when(jarFilePathResolver.byGetProtectionDomain(Ascii.class)).thenThrow(new SecurityException("not allowed"));
String jarPath = jarFilePathResolver.getJarFilePath(Ascii.class);
assertThat(jarPath).endsWith(".jar").contains("guava");
assertThat(new File(jarPath)).exists();
verify(jarFilePathResolver, times(1)).byGetProtectionDomain(Ascii.class);
verify(jarFilePathResolver, times(1)).byGetResource(Ascii.class); 
```

如果我们执行测试，两个测试都通过。

## 6.结论

在本文中，我们学习了如何从给定的类中获取 JAR 文件的完整路径。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220824180106/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jar)