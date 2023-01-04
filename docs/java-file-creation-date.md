# 用 Java 确定文件创建日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-file-creation-date>

## 1.概观

JDK 7 引入了获取文件创建日期的能力。

在本教程中，我们将学习如何通过`java.nio`访问它。

## 2.`Files.getAttribute`

获得文件创建日期的一种方法是**使用方法** `**Files.getAttribute**` 和给定的`Path`:

```
try {
    FileTime creationTime = (FileTime) Files.getAttribute(path, "creationTime");
} catch (IOException ex) {
    // handle exception
}
```

`creationTime`的类型是`FileTime`，但是由于**方法返回`Object,`，我们不得不将其转换为**。

`FileTime` 将日期值保存为时间戳属性。例如，可以用 `toInstant()`方法将其转换为`Instant`。

如果文件系统没有存储文件的创建日期，那么该方法将返回`null`。

## 3.`Files.readAttributes`

另一种获取创建日期的方法是使用 **`Files.readAttributes`，对于给定的*路径，*会立即返回文件的所有基本属性**:

```
try {
    BasicFileAttributes attr = Files.readAttributes(path, BasicFileAttributes.class);
    FileTime fileTime = attr.creationTime();
} catch (IOException ex) {
    // handle exception
}
```

该方法返回一个`BasicFileAttributes,`,我们可以用它来获得文件的基本属性。方法`creationTime()` 将文件的创建日期返回为`FileTime`。

这一次，如果文件系统没有存储创建文件的日期，那么**该方法将返回最后修改日期**。如果最后修改日期没有被存储，那么将返回纪元(01.01.1970)。

## 4.结论

在本教程中，我们学习了如何在 Java 中确定文件的创建日期。具体来说，我们了解到可以用`Files.getAttribute `和`Files.readAttributes`来做。

与往常一样，GitHub 上的[提供了示例代码。](https://web.archive.org/web/20221013193921/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio)