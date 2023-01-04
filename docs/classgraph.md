# 类图库指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/classgraph>

## 1。概述

在这个简短的教程中，我们将讨论 [Classgraph](https://web.archive.org/web/20221205135656/https://github.com/classgraph/classgraph) 库——它有什么帮助以及我们如何使用它。

Classgraph 帮助我们在 Java 类路径中找到目标资源，构建关于找到的资源的元数据，并提供方便的 API 来处理元数据。

这种用例在基于 Spring 的应用程序中非常流行，在这些应用程序中，用原型注释标记的组件会自动注册到应用程序上下文中。然而，我们也可以将这种方法用于定制任务。例如，我们可能希望找到所有带有特定注释的类，或者所有带有特定名称的资源文件。

酷的是 **Classgraph 很快，因为它在字节码级别**工作，这意味着被检查的类没有被加载到 JVM，并且它不使用反射进行处理。

## 2.Maven 依赖性

首先，让我们将[类图](https://web.archive.org/web/20221205135656/https://search.maven.org/search?q=g:io.github.classgraph%20AND%20a:classgraph)库添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>io.github.classgraph</groupId>
    <artifactId>classgraph</artifactId>
    <version>4.8.28</version>
</dependency>
```

在接下来的部分中，我们将研究几个使用这个库的 API 的实际例子。

## 3。基本用法

使用该库有三个基本步骤:

1.  设置扫描选项–例如，目标包
2.  执行扫描
3.  处理扫描结果

让我们为示例设置创建以下域:

```java
@Target({TYPE, METHOD, FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {

    String value() default "";
}
```

```java
@TestAnnotation
public class ClassWithAnnotation {
}
```

现在让我们看看上面用`@TestAnnotation`查找类的例子的 3 个步骤:

```java
try (ScanResult result = new ClassGraph().enableClassInfo().enableAnnotationInfo()
  .whitelistPackages(getClass().getPackage().getName()).scan()) {

    ClassInfoList classInfos = result.getClassesWithAnnotation(TestAnnotation.class.getName());

    assertThat(classInfos).extracting(ClassInfo::getName).contains(ClassWithAnnotation.class.getName());
}
```

我们来分解一下上面的例子:

*   我们从**设置扫描选项**开始(我们已经配置扫描器只解析类和注释信息，并指示它只解析目标包中的文件)
*   我们**使用`ClassGraph.scan()`方法执行了扫描**
*   我们**使用`ScanResult`** 通过调用`getClassWithAnnotation()`方法找到带注释的类

正如我们将在接下来的例子中看到的，`ScanResult`对象可以包含许多关于我们想要检查的 API 的信息，比如`ClassInfoList.`

## 4。通过方法注释过滤

让我们把例子扩展到方法注释:

```java
public class MethodWithAnnotation {

    @TestAnnotation
    public void service() {
    }
}
```

**我们可以使用类似的方法找到所有包含由目标注释标记的方法的类— `getClassesWithMethodAnnotations()` :**

```java
try (ScanResult result = new ClassGraph().enableAllInfo()
  .whitelistPackages(getClass().getPackage().getName()).scan()) {

    ClassInfoList classInfos = result.getClassesWithMethodAnnotation(TestAnnotation.class.getName());

    assertThat(classInfos).extracting(ClassInfo::getName).contains(MethodWithAnnotation.class.getName());
}
```

**该方法返回一个`ClassInfoList`对象，包含与扫描匹配的类的信息。**

## 5。通过注释参数过滤

让我们看看如何找到所有带有目标注释标记的方法和目标注释参数值的类。

首先，让我们定义包含带有两个不同参数值的`@TestAnnotation,`的方法的类:

```java
public class MethodWithAnnotationParameterDao {

    @TestAnnotation("dao")
    public void service() {
    }
}
```

```java
public class MethodWithAnnotationParameterWeb {

    @TestAnnotation("web")
    public void service() {
    }
}
```

现在，让我们遍历`ClassInfoList`结果，并验证每个方法的注释:

```java
try (ScanResult result = new ClassGraph().enableAllInfo()
  .whitelistPackages(getClass().getPackage().getName()).scan()) {

    ClassInfoList classInfos = result.getClassesWithMethodAnnotation(TestAnnotation.class.getName());
    ClassInfoList webClassInfos = classInfos.filter(classInfo -> {
        return classInfo.getMethodInfo().stream().anyMatch(methodInfo -> {
            AnnotationInfo annotationInfo = methodInfo.getAnnotationInfo(TestAnnotation.class.getName());
            if (annotationInfo == null) {
                return false;
            }
            return "web".equals(annotationInfo.getParameterValues().getValue("value"));
        });
    });

    assertThat(webClassInfos).extracting(ClassInfo::getName)
      .contains(MethodWithAnnotationParameterWeb.class.getName());
}
```

**这里，我们使用了`AnnotationInfo`和`MethodInfo`元数据类来查找我们想要检查的方法和注释的元数据。**

## 6。通过字段注释过滤

我们还可以使用`getClassesWithFieldAnnotation()`方法根据字段注释过滤`ClassInfoList`结果:

```java
public class FieldWithAnnotation {

    @TestAnnotation
    private String s;
}
```

```java
try (ScanResult result = new ClassGraph().enableAllInfo()
  .whitelistPackages(getClass().getPackage().getName()).scan()) {

    ClassInfoList classInfos = result.getClassesWithFieldAnnotation(TestAnnotation.class.getName());

    assertThat(classInfos).extracting(ClassInfo::getName).contains(FieldWithAnnotation.class.getName());
}
```

## 7。寻找资源

最后，我们将看看如何找到关于类路径资源的信息。

让我们在`classgraph`类路径根目录中创建一个资源文件——例如，`src/test/resources/classgraph/my.config`——并赋予它一些内容:

```java
my data
```

我们现在可以找到资源并获取其内容:

```java
try (ScanResult result = new ClassGraph().whitelistPaths("classgraph").scan()) {
    ResourceList resources = result.getResourcesWithExtension("config");
    assertThat(resources).extracting(Resource::getPath).containsOnly("classgraph/my.config");
    assertThat(resources.get(0).getContentAsString()).isEqualTo("my data");
}
```

我们可以看到，我们已经使用了`ScanResult'` s `getResourcesWithExtension()`方法来查找我们的特定文件。**该类还有一些其他有用的资源相关方法，比如`getAllResources(), getResourcesWithPath()`和** `**getResourcesMatchingPattern()**.`

这些方法返回一个`ResourceList`对象，该对象可以进一步用于遍历和操作`Resource`对象。

## 8.实例化

当我们想要实例化找到的类时，重要的是不要通过`Class.forName,`而是使用库方法`ClassInfo.loadClass`来完成。

原因是 Classgraph 使用自己的类加载器从一些 JAR 文件中加载类。因此，如果我们使用`Class.forName`，同一个类可能会被不同的类加载器加载多次，这可能会导致不小的错误。

## 9。结论

在本文中，我们学习了如何有效地找到类路径资源，并使用类图库检查它们的内容。

像往常一样，本文的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221205135656/https://github.com/eugenp/tutorials/tree/master/libraries-2)