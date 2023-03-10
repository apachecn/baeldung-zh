# 用 Gradle 跳过测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-skip-tests>

## 1.介绍

虽然跳过测试通常是一个坏主意，但是在某些情况下它可能是有用的，并且它节省了我们一些时间。例如，假设我们正在开发一个新特性，我们希望在中间版本中看到结果。在这种情况下，我们可能会暂时跳过测试，以减少编译和运行它们的开销。毫无疑问，忽视测试会导致许多严重的问题。

在这个简短的教程中，我们将学习如何在使用 [Gradle](/web/20220823131340/https://www.baeldung.com/gradle) 构建工具时跳过测试。

## 2.使用命令行标志

首先，让我们创建一个想要跳过的简单测试:

```java
@Test
void skippableTest() {
    Assertions.assertTrue(true);
}
```

当我们运行`build`命令时:

```java
gradle build
```

我们将看到正在运行的任务:

```java
> ...
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test
> ...
```

**要从 Gradle 构建中跳过任何任务，我们可以使用`-x`或 `–exclude-task`选项。**在这种情况下，**我们将使用`-x test`从构建中跳过测试。**

要查看它的运行情况，让我们运行带有`-x` 选项的`build`命令:

```java
gradle build -x test
```

我们将看到正在运行的任务:

```java
> Task :compileJava NO-SOURCE 
> Task :processResources NO-SOURCE 
> Task :classes UP-TO-DATE 
> Task :jar 
> Task :assemble 
> Task :check 
> Task :build
```

结果，测试源没有被编译，因此也没有被执行。

## 3.使用 Gradle 构建脚本

我们有更多的选项来跳过使用 Gradle 构建脚本的测试。例如，**我们可以基于某些条件跳过测试，或者只在特定的环境中使用`onlyIf()`方法**。如果这个方法返回`false`，测试将被跳过。

让我们跳过基于检查项目属性的测试:

```java
test.onlyIf { !project.hasProperty('someProperty') }
```

现在我们将运行`build`命令，并将`someProperty` 传递给 Gradle:

```java
gradle build -PsomeProperty
```

因此，Gradle 跳过运行测试:

```java
> ...
> Task :compileTestJava 
> Task :processTestResources NO-SOURCE 
> Task :testClasses 
> Task :test SKIPPED 
> Task :check UP-TO-DATE 
> ...
```

此外，**我们可以使用我们的`build.gradle`文件中的`exclude`属性来排除基于它们的包或类名**的测试:

```java
test {
    exclude 'org/boo/**'
    exclude '**/Bar.class'
}
```

**我们也可以跳过基于正则表达式模式的测试。**例如，我们可以跳过所有类名以单词`Integration`结尾的测试:

```java
test {
    exclude '**/**Integration'
}
```

## 4.结论

在本文中，我们学习了如何在使用 Gradle 构建工具时跳过测试。我们还研究了可以在命令行上使用的所有相关选项，以及可以在 Gradle 构建脚本中使用的选项。