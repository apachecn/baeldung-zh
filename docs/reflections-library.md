# 反思库指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reflections-library>

## 1.介绍

[`Reflections`](https://web.archive.org/web/20221107170009/https://github.com/ronmamo/reflections) 库是一个类路径扫描器。它将扫描的元数据编入索引，并允许我们在运行时查询它。它还可以保存这些信息，因此我们可以在项目过程中的任何时候收集和使用这些信息，而不必再次重新扫描类路径。

在本教程中，我们将展示如何配置`Reflections`库并在我们的 Java 项目中使用它。

## 2.Maven 依赖性

为了使用`Reflections`，我们需要在我们的项目中包含它的依赖关系:

```java
<dependency>
    <groupId>org.reflections</groupId>
    <artifactId>reflections</artifactId>
    <version>0.9.11</version>
</dependency>
```

我们可以在 Maven Central 上找到最新版本的库[。](https://web.archive.org/web/20221107170009/https://search.maven.org/search?q=g:org.reflections%20AND%20a:reflections)

## 3.配置`Reflections`

接下来，我们需要配置库。配置的主要元素是 URL 和扫描器。

URL 告诉库要扫描类路径的哪些部分，而扫描器是扫描给定 URL 的对象。

如果没有配置扫描仪，库将使用`TypeAnnotationsScanner`和`SubTypesScanner` 作为默认扫描仪。

### 3.1.添加 URL

我们可以通过提供配置的元素作为 varargs 构造函数的参数，或者通过使用`ConfigurationBuilder`对象来配置`Reflections`。

例如，我们可以通过使用代表包名、类或类加载器的`String`实例化`Reflections`来添加 URL:

```java
Reflections reflections = new Reflections("com.baeldung.reflections");
Reflections reflections = new Reflections(MyClass.class);
Reflections reflections = new Reflections(MyClass.class.getClassLoader());
```

此外，因为`Reflections`有一个 varargs 构造函数，我们可以组合所有上述配置的类型来实例化它:

```java
Reflections reflections = new Reflections("com.baeldung.reflections", MyClass.class);
```

这里，我们通过指定要扫描的包和类来添加 URL。

我们可以通过使用`ConfigurationBuilder`获得相同的结果:

```java
Reflections reflections = new Reflections(new ConfigurationBuilder()
  .setUrls(ClasspathHelper.forPackage("com.baeldung.reflections"))));
```

与`forPackage()`方法一起，`Classp` `ath` `Helpe` `r`提供了其他方法，如`forClass()`和`forClassLoader()`，用于将 URL 添加到配置中。

### 3.2.添加扫描仪

**倒影库带有许多内置扫描仪:**

*   `FieldAnnotationsScanner –`查找字段的注释
*   `MethodParameterScanner –`扫描方法/构造函数，然后索引参数，并返回类型和参数注释
*   检查方法/构造函数，然后索引参数名
*   `TypeElementsScanner –`检查字段和方法，然后将完全限定名存储为键，将元素存储为值
*   `MemberUsageScanner –`扫描方法/构造函数/字段用法
*   `TypeAnnotationsScanner –`寻找类的运行时注释
*   `SubTypesScanner –`搜索`super`类和类的接口，允许反向查找子类型
*   `MethodAnnotationsScanner –`扫描方法的注释
*   `ResourcesScanner –`在集合中收集所有非类资源

我们可以将扫描仪作为`Reflections`构造器的参数添加到配置中。

例如，让我们添加上面列表中的前两个扫描仪:

```java
Reflections reflections = new Reflections("com.baeldung.reflections"), 
  new FieldAnnotationsScanner(), 
  new MethodParameterScanner());
```

同样，这两个扫描器可以通过使用`ConfigurationBuilder`助手类来配置:

```java
Reflections reflections = new Reflections(new ConfigurationBuilder()
  .setUrls(ClasspathHelper.forPackage("com.baeldung.reflections"))
  .setScanners(new FieldAnnotationsScanner(), new MethodParameterScanner()));
```

### 3.3.添加`ExecutorService`

除了 URL 和扫描器， **`Reflections`还为我们提供了使用 [`ExecutorService`](/web/20221107170009/https://www.baeldung.com/java-executor-service-tutorial)** 异步扫描类路径的可能性。

我们可以把它作为一个参数添加到`Reflections`构造函数中，或者通过`ConfigurationBuilder`:

```java
Reflections reflections = new Reflections(new ConfigurationBuilder()
  .setUrls(ClasspathHelper.forPackage("com.baeldung.reflections"))
  .setScanners(new SubTypesScanner(), new TypeAnnotationsScanner())
  .setExecutorService(Executors.newFixedThreadPool(4)));
```

另一种选择是简单地调用`useParallelExecutor()`方法。该方法配置一个默认的`FixedThreadPool` `ExecutorService`，其大小等于可用核心处理器的数量。

### 3.4.添加过滤器

另一个重要的配置元素是过滤器。**当扫描类路径**时，过滤器告诉扫描器包括什么，排除什么。

举例来说，我们可以配置过滤器来排除对测试包的扫描:

```java
Reflections reflections = new Reflections(new ConfigurationBuilder()
  .setUrls(ClasspathHelper.forPackage("com.baeldung.reflections"))
  .setScanners(new SubTypesScanner(), new TypeAnnotationsScanner())
  .filterInputsBy(new FilterBuilder().excludePackage("com.baeldung.reflections.test")));
```

到目前为止，我们已经对`Reflections`配置的不同元素进行了快速概述。接下来，我们将了解如何使用库。

## 4.使用反射进行查询

在调用其中一个`Reflections`构造函数后，配置的扫描器扫描所有提供的 URL。然后，**对于每个扫描仪，库将结果放入`Multimap`存储器**。因此，为了使用`Reflections`，我们需要通过调用提供的查询方法来查询这些商店。

让我们看看这些查询方法的一些例子。

### 4.1.子类型

让我们从检索`Reflections`提供的所有扫描仪开始:

```java
public Set<Class<? extends Scanner>> getReflectionsSubTypes() {
    Reflections reflections = new Reflections(
      "org.reflections", new SubTypesScanner());
    return reflections.getSubTypesOf(Scanner.class);
}
```

### 4.2.批注类型

接下来，我们可以获得实现给定注释的所有类和接口。

所以，让我们检索一下`java.util.function `包的所有功能接口:

```java
public Set<Class<?>> getJDKFunctinalInterfaces() {
    Reflections reflections = new Reflections("java.util.function", 
      new TypeAnnotationsScanner());
    return reflections.getTypesAnnotatedWith(FunctionalInterface.class);
}
```

### 4.3.带注释的方法

现在，让我们使用`MethodAnnotationsScanner`来获取用给定注释注释的所有方法:

```java
public Set<Method> getDateDeprecatedMethods() {
    Reflections reflections = new Reflections(
      "java.util.Date", 
      new MethodAnnotationsScanner());
    return reflections.getMethodsAnnotatedWith(Deprecated.class);
}
```

### 4.4.带注释的构造函数

此外，我们可以获得所有不推荐使用的构造函数:

```java
public Set<Constructor> getDateDeprecatedConstructors() {
    Reflections reflections = new Reflections(
      "java.util.Date", 
      new MethodAnnotationsScanner());
    return reflections.getConstructorsAnnotatedWith(Deprecated.class);
}
```

### 4.5.方法参数

此外，我们可以使用`MethodParameterScanner`找到给定参数类型的所有方法:

```java
public Set<Method> getMethodsWithDateParam() {
    Reflections reflections = new Reflections(
      java.text.SimpleDateFormat.class, 
      new MethodParameterScanner());
    return reflections.getMethodsMatchParams(Date.class);
}
```

### 4.6.方法的返回类型

此外，我们还可以使用同一个扫描器来获取给定返回类型的所有方法。

假设我们想要找到返回`void`的`SimpleDateFormat` 的所有方法:

```java
public Set<Method> getMethodsWithVoidReturn() {
    Reflections reflections = new Reflections(
      "java.text.SimpleDateFormat", 
      new MethodParameterScanner());
    return reflections.getMethodsReturn(void.class);
}
```

### 4.7.资源

最后，让我们使用`ResourcesScanner`在我们的类路径中查找给定的文件名:

```java
public Set<String> getPomXmlPaths() {
    Reflections reflections = new Reflections(new ResourcesScanner());
    return reflections.getResources(Pattern.compile(".*pom\\.xml"));
}
```

### 4.8.其他查询方法

以上只是展示如何使用`Reflections'`查询方法的几个例子。然而，还有其他的查询方法我们没有在这里介绍:

*   `getMethodsWithAnyParamAnnotated`
*   `getConstructorsMatchParams`
*   `getConstructorsWithAnyParamAnnotated`
*   `getFieldsAnnotatedWith`
*   `getMethodParamNames`
*   `getConstructorParamNames`
*   `getFieldUsage`
*   `getMethodUsage`
*   `getConstructorUsage`

## 5.将`Reflections`集成到构建生命周期中

我们可以使用 [`gmavenplus-plugin`](https://web.archive.org/web/20221107170009/https://search.maven.org/search?q=a:gmavenplus-plugin%20AND%20g:org.codehaus.gmavenplus) 轻松地将`Reflections`集成到我们的 Maven 构建中。

让我们将其配置为将扫描结果保存到文件中:

```java
<plugin>
    <groupId>org.codehaus.gmavenplus</groupId>
    <artifactId>gmavenplus-plugin</artifactId>
    <version>1.5</version>
    <executions>
        <execution>
            <phase>generate-resources</phase>
            <goals>
                <goal>execute</goal>
            </goals>
            <configuration>
                <scripts>
                    <script><![CDATA[
                        new org.reflections.Reflections(
                          "com.baeldung.refelections")
                            .save("${outputDirectory}/META-INF/reflections/reflections.xml")]]>
                    </script>
                </scripts>
            </configuration>
        </execution>
    </executions>
</plugin>
```

稍后，通过调用`collect()`方法，**我们可以检索保存的结果** **并使它们可供进一步使用，而不必执行新的扫描:**

```java
Reflections reflections
  = isProduction() ? Reflections.collect() : new Reflections("com.baeldung.reflections");
```

## 6.结论

在本文中，我们探索了`Reflections`库。我们讨论了不同的配置元素及其用法。最后，我们看到了如何将`Reflections`集成到 Maven 项目的构建生命周期中。

和往常一样，完整的代码可以在 [GitHub](https://web.archive.org/web/20221107170009/https://github.com/eugenp/tutorials/tree/master/libraries-6) 上找到。