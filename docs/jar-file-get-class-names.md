# 获取 JAR 文件中的类名

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jar-file-get-class-names>

## 1.概观

大多数 Java 库都可以作为 [JAR 文件](/web/20221206071122/https://www.baeldung.com/java-create-jar)获得。在本教程中，我们将介绍如何从命令行和 Java 程序中获取给定 JAR 文件中的类名。

然后，我们将看一个在运行时从给定的 JAR 文件加载类的 Java 程序示例。

## 2.示例 JAR 文件

在本教程中，我们将以`[stripe-0.0.1-SNAPSHOT.jar](https://web.archive.org/web/20221206071122/https://github.com/eugenp/tutorials/tree/master/stripe)`文件为例来说明如何获取 JAR 文件中的类名:

[![a jar example 1](img/2a328174a64e1c3c65d24aa2d4a01de9.png)](/web/20221206071122/https://www.baeldung.com/wp-content/uploads/2020/10/a_jar_example-1.png)

## 3.使用`jar`命令

**JDK 的战舰带着 [`jar`](https://web.archive.org/web/20221206071122/https://docs.oracle.com/en/java/javase/11/tools/jar.html) 的命令。我们可以使用这个带有`t`和`f`选项的命令来列出 JAR 文件**的内容:

```java
$ jar tf stripe-0.0.1-SNAPSHOT.jar 
META-INF/
META-INF/MANIFEST.MF
...
templates/result.html
templates/checkout.html
application.properties
com/baeldung/stripe/StripeApplication.class
com/baeldung/stripe/ChargeRequest.class
com/baeldung/stripe/StripeService.class
com/baeldung/stripe/ChargeRequest$Currency.class
...
```

由于我们只对存档中的`*.class`文件感兴趣，我们可以使用 [`grep`](/web/20221206071122/https://www.baeldung.com/linux/common-text-search) 命令过滤输出:

```java
$ jar tf stripe-0.0.1-SNAPSHOT.jar | grep '\.class

这为我们提供了 JAR 文件中的类文件列表。

## 4.用 Java 获取 JAR 文件的类名

使用`jar`命令打印 JAR 文件中的类名非常简单。然而，有时我们想在 Java 程序中从 JAR 文件加载一些类。在这种情况下，命令行输出是不够的。

为了实现我们的目标，我们需要从 Java 程序中扫描 JAR 文件并获取类名。

让我们看看如何使用`[JarFile](https://web.archive.org/web/20221206071122/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/jar/JarFile.html)`和`[JarEntry](https://web.archive.org/web/20221206071122/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/jar/JarEntry.html)`类从示例 JAR 文件**中提取类名:**

```
public static Set<String> getClassNamesFromJarFile(File givenFile) throws IOException {
    Set<String> classNames = new HashSet<>();
    try (JarFile jarFile = new JarFile(givenFile)) {
        Enumeration<JarEntry> e = jarFile.entries();
        while (e.hasMoreElements()) {
            JarEntry jarEntry = e.nextElement();
            if (jarEntry.getName().endsWith(".class")) {
                String className = jarEntry.getName()
                  .replace("/", ".")
                  .replace(".class", "");
                classNames.add(className);
            }
        }
        return classNames;
    }
} 
```java

现在，让我们仔细看看上面方法中的代码，并理解它是如何工作的:

*   `try (JarFile jarFile = new JarFile(givenFile))`–这里，我们使用了一个 [try-with-resources 语句](/web/20221206071122/https://www.baeldung.com/java-try-with-resources)从给定的`File`对象中获取`jarFile`
*   `if (jarEntry.getName().endsWith(“.class”)){…}`–我们取每个类`jarEntry`，把类文件的路径改成合格的类名，比如把`“package1/package2/SomeType.class”`改成`“package1.package2.SomeType”`

让我们通过一个单元测试方法来验证这个方法是否能够从我们的示例 JAR 文件中提取类名:

```
private static final String JAR_PATH = "example-jar/stripe-0.0.1-SNAPSHOT.jar";
private static final Set<String> EXPECTED_CLASS_NAMES = Sets.newHashSet(
  "com.baeldung.stripe.StripeApplication",
  "com.baeldung.stripe.ChargeRequest",
  "com.baeldung.stripe.StripeService",
  "com.baeldung.stripe.ChargeRequest$Currency",
  "com.baeldung.stripe.ChargeController",
  "com.baeldung.stripe.CheckoutController");

@Test
public void givenJarFilePath_whenLoadClassNames_thenGetClassNames() throws IOException, URISyntaxException {
    File jarFile = new File(
      Objects.requireNonNull(getClass().getClassLoader().getResource(JAR_PATH)).toURI());

    Set<String> classNames = GetClassNamesFromJar.getClassNamesFromJarFile(jarFile);

    Assert.assertEquals(EXPECTED_CLASS_NAMES, classNames);
} 
```java

## 5.用 Java 从 JAR 文件中获取类

我们已经看到了如何从 JAR 文件中获取类名。有时，我们希望在运行时从 JAR 文件中动态加载一些类。

在这种情况下，我们可以首先使用我们的`getClassNamesFromJarFile` 方法从给定的 JAR 文件中获取类名。

接下来，我们可以创建一个`ClassLoader`来按名称加载所需的类:

```
public static Set<Class> getClassesFromJarFile(File jarFile) throws IOException, ClassNotFoundException {
    Set<String> classNames = getClassNamesFromJarFile(jarFile);
    Set<Class> classes = new HashSet<>(classNames.size());
    try (URLClassLoader cl = URLClassLoader.newInstance(
           new URL[] { new URL("jar:file:" + jarFile + "!/") })) {
        for (String name : classNames) {
            Class clazz = cl.loadClass(name); // Load the class by its name
            classes.add(clazz);
        }
    }
    return classes;
}
```java

在上面的方法中，我们创建了一个`[URLClassLoader](https://web.archive.org/web/20221206071122/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/URLClassLoader.html)`对象来加载类。实现非常简单。

然而，可能有必要稍微解释一下 JAR URL 的语法。**一个有效的 JAR URL 包含三个部分:“`jar:`+`[the location of the JAR file]`+`!/”.`+**

**终止的“`!/`”表示 JAR URL 指向整个 JAR 文件。**让我们看几个 JAR URL 的例子:

```
jar:http://www.example.com/some_jar_file.jar!/
jar:file:/local/path/to/some_jar_file.jar!/
jar:file:/C:/windows/path/to/some_jar_file.jar!/
```java

在我们的`getClassesFromJarFile`方法中，JAR 文件位于本地文件系统上，因此，URL 的前缀是“`file:`”。

现在，让我们编写一个测试方法来验证我们的方法是否可以获得所有预期的`Class`对象:

```
@Test
public void givenJarFilePath_whenLoadClass_thenGetClassObjects()
  throws IOException, ClassNotFoundException, URISyntaxException {
    File jarFile
      = new File(Objects.requireNonNull(getClass().getClassLoader().getResource(JAR_PATH)).toURI());
    Set<Class> classes = GetClassNamesFromJar.getClassesFromJarFile(jarFile);
    Set<String> names = classes.stream().map(Class::getName).collect(Collectors.toSet());
    Assert.assertEquals(EXPECTED_CLASS_NAMES, names);
} 
```java

一旦我们有了所需的`Class`对象，我们就可以使用 [Java 反射](/web/20221206071122/https://www.baeldung.com/java-reflection)来创建类的实例并调用方法。

## 6.结论

在本文中，我们学习了从给定的 JAR 文件中获取类名的两种不同方法。

`jar`命令可以打印类名。如果我们需要检查一个 JAR 文件是否包含一个给定的类，这是非常方便的。然而，如果我们需要从正在运行的 Java 程序中获取类名，`JarFile`和`JarEntry`可以帮助我们实现这一点。

最后，我们还看到了一个 Java 程序在运行时从 JAR 文件加载类的例子。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221206071122/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jar)
com/baeldung/stripe/StripeApplication.class
com/baeldung/stripe/ChargeRequest.class
com/baeldung/stripe/StripeService.class
com/baeldung/stripe/ChargeRequest$Currency.class
com/baeldung/stripe/ChargeController.class
com/baeldung/stripe/CheckoutController.class
```

这为我们提供了 JAR 文件中的类文件列表。

## 4.用 Java 获取 JAR 文件的类名

使用`jar`命令打印 JAR 文件中的类名非常简单。然而，有时我们想在 Java 程序中从 JAR 文件加载一些类。在这种情况下，命令行输出是不够的。

为了实现我们的目标，我们需要从 Java 程序中扫描 JAR 文件并获取类名。

让我们看看如何使用`[JarFile](https://web.archive.org/web/20221206071122/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/jar/JarFile.html)`和`[JarEntry](https://web.archive.org/web/20221206071122/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/jar/JarEntry.html)`类从示例 JAR 文件**中提取类名:**

[PRE2]

现在，让我们仔细看看上面方法中的代码，并理解它是如何工作的:

*   `try (JarFile jarFile = new JarFile(givenFile))`–这里，我们使用了一个 [try-with-resources 语句](/web/20221206071122/https://www.baeldung.com/java-try-with-resources)从给定的`File`对象中获取`jarFile`
*   `if (jarEntry.getName().endsWith(“.class”)){…}`–我们取每个类`jarEntry`，把类文件的路径改成合格的类名，比如把`“package1/package2/SomeType.class”`改成`“package1.package2.SomeType”`

让我们通过一个单元测试方法来验证这个方法是否能够从我们的示例 JAR 文件中提取类名:

[PRE3]

## 5.用 Java 从 JAR 文件中获取类

我们已经看到了如何从 JAR 文件中获取类名。有时，我们希望在运行时从 JAR 文件中动态加载一些类。

在这种情况下，我们可以首先使用我们的`getClassNamesFromJarFile` 方法从给定的 JAR 文件中获取类名。

接下来，我们可以创建一个`ClassLoader`来按名称加载所需的类:

[PRE4]

在上面的方法中，我们创建了一个`[URLClassLoader](https://web.archive.org/web/20221206071122/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/URLClassLoader.html)`对象来加载类。实现非常简单。

然而，可能有必要稍微解释一下 JAR URL 的语法。**一个有效的 JAR URL 包含三个部分:“`jar:`+`[the location of the JAR file]`+`!/”.`+**

**终止的“`!/`”表示 JAR URL 指向整个 JAR 文件。**让我们看几个 JAR URL 的例子:

[PRE5]

在我们的`getClassesFromJarFile`方法中，JAR 文件位于本地文件系统上，因此，URL 的前缀是“`file:`”。

现在，让我们编写一个测试方法来验证我们的方法是否可以获得所有预期的`Class`对象:

[PRE6]

一旦我们有了所需的`Class`对象，我们就可以使用 [Java 反射](/web/20221206071122/https://www.baeldung.com/java-reflection)来创建类的实例并调用方法。

## 6.结论

在本文中，我们学习了从给定的 JAR 文件中获取类名的两种不同方法。

`jar`命令可以打印类名。如果我们需要检查一个 JAR 文件是否包含一个给定的类，这是非常方便的。然而，如果我们需要从正在运行的 Java 程序中获取类名，`JarFile`和`JarEntry`可以帮助我们实现这一点。

最后，我们还看到了一个 Java 程序在运行时从 JAR 文件加载类的例子。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221206071122/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jar)