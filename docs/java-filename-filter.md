# 快速使用 FilenameFilter

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-filename-filter>

## 1。简介

在添加信息注释`@FunctionalInterface`之前，Java 已经有了[功能接口](/web/20220526040357/https://www.baeldung.com/java-8-functional-interfaces)。`FilenameFilter`就是这样一个接口。

我们将简要地看一下它的用法，并理解它在当今 Java 世界中的位置。

## 2。`FilenameFilter`

因为**是一个函数接口——我们必须有一个抽象方法**,并且`FilenameFilter`遵循这个定义:

```java
boolean accept(File dir, String name);
```

## 3.使用

我们几乎专门使用`FilenameFilter`来列出目录中满足指定过滤器的所有文件。

`java.io.File`中重载的`list(..)` 和 `listFiles(..)` 方法接受`FilenameFilter`的一个实例，并返回满足过滤器的所有文件的一个`array`。

下面的测试用例过滤一个目录中的所有`json`文件:

```java
@Test
public void whenFilteringFilesEndingWithJson_thenEqualExpectedFiles() {
    FilenameFilter filter = (dir, name) -> name.endsWith(".json");

    String[] expectedFiles = { "people.json", "students.json" };
    File directory = new File(getClass().getClassLoader()
      .getResource("testFolder")
      .getFile());
    String[] actualFiles = directory.list(filter);

    Assert.assertArrayEquals(expectedFiles, actualFiles);
}
```

### 3.1.`FileFilter`为`BiPredicate`

Oracle 在 Java 8 中增加了 40 多个功能接口，与传统接口不同，这些接口是通用的。这意味着我们可以将它们用于任何引用类型。

[`BiPredicate<T, U>`](https://web.archive.org/web/20220526040357/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiPredicate.html) 就是这样一个界面。它的‘单一抽象方法’有这样的定义:

```java
boolean test(T t, U u);
```

**这意味着`FilenameFilter`只是`BiPredicate`的一个特例，其中`T`是`File`,`U`是`String`。**

## 4。结论

即使我们现在有了通用的`Predicate`和`BiPredicate` 函数接口，我们还会继续看到`FilenameFilter`的出现，只是因为它已经在现有的 Java 库中使用了。

此外，它很好地服务于它的单一目的，因此没有理由在适用时不使用它。

和往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220526040357/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis)