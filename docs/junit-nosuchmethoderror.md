# 修正 NoSuchMethodError JUnit 错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-nosuchmethoderror>

## 1.概观

在本文中，我们将**学习如何修复`NoSuchMethodError` 和`NoClassDefFoundError` JUnit 错误**。当我们的`classpath`中有两个不同的 JUnit 版本时，通常会出现这样的问题。例如，当项目的 JUnit 版本不同于 Maven 或 Gradle 依赖项中使用的版本时，可能会出现这种情况。

## 2.JUnit 在 Spring 项目中的`NoClassDefFoundError`

假设我们有一个使用 Spring Boot 2.1.2 和 [Spring Boot 启动测试](/web/20220626074048/https://www.baeldung.com/spring-boot-testing)依赖的 Maven 项目。有了这样的依赖，我们可以使用 JUnit 5.3.2 编写和运行自动化测试，它是`spring-boot-test`依赖的 JUnit 版本。

现在，假设我们将继续使用 Spring Boot 2.1.2。但是，我们希望使用 JUnit 5.7.1。一种可能的方法是在我们的`pom.xml`文件中包含`junit-jupiter-api`、`junit-jupiter-params`、`junit-jupiter-engine`和`junit-platform-launcher`依赖项:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.7.1</version>
    <scope>test</scope>
</dependency>
...
```

然而，在这种情况下，当我们运行测试时，我们将得到一个`NoClassDefFoundError`:

```java
[ERROR] java.lang.NoClassDefFoundError: org/junit/platform/commons/util/ClassNamePatternFilterUtils
```

如果我们迁移到 JUnit 5.4.0，将会出现一个`NoSuchMethodError`，而不是`NoClassDefFoundError`。

## 3.了解并修复错误

如前一节所述，当我们试图将 JUnit 版本从 5.3.2 迁移到 5.7.1 时，我们以一个`NoClassDefFoundError`结束。

这个错误的发生是因为**我们的`classpath`最终拥有了两个不同版本的 JUnit** 。因此，我们的项目是用较新版本的 JUnit (5.7.1)编译的，但在运行时发现了一个较旧的版本(5.3.2)。因此，JUnit 启动程序试图使用一个在旧版本的 JUnit 中不可用的类。

接下来，我们将学习修复此错误的不同解决方案。

### 3.1.覆盖 Spring 的 JUnit 版本

在我们的示例中，修复错误的有效方法是覆盖由 Spring 管理的 JUnit 版本:

```java
<properties>
    <junit-jupiter.version>5.7.1</junit-jupiter.version>
</properties>
```

现在，我们也可以用下面的代码替换我们的 JUnit 依赖项:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
</dependency>
```

这个单独的依赖项将包括`junit-jupiter-api`、`junit-jupiter-params`和`junit-jupiter-engine`。只有当我们需要以编程方式[运行我们的 JUnit 测试时](/web/20220626074048/https://www.baeldung.com/junit-tests-run-programmatically-from-java)，依赖关系才是必要的。

类似地，**我们也可以覆盖 Gradle 项目中的托管版本**:

```java
ext['junit-jupiter.version'] = '5.7.1'
```

### 3.2.任何项目的解决方案

在上一节中，我们学习了如何修复 Spring 项目中的`NoSuchMethodError`和`NoClassDefFoundError` JUnit 错误，这是最常见的情况。然而，如果这些错误正在发生，并且我们的项目没有使用 Spring，我们可以尝试**修复 Maven** 中的依赖冲突。

与我们在 Spring 的例子中发生的情况类似，由于传递依赖关系，一个项目可能有**多个版本的 JUnit。对于这样的场景，我们有一个关于如何解决 Maven 中冲突的[详细教程。](/web/20220626074048/https://www.baeldung.com/maven-version-collision)**

## 4.结论

在本教程中，我们重现并学习了如何修复`NoSuchMethodError`和`NoClassDefFoundError` JUnit 错误。代码片段一如既往地可以在 GitHub 上找到[。请注意，在这个源代码示例中，我们覆盖了父项目](https://web.archive.org/web/20220626074048/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing)[的`pom.xml`文件中的 JUnit 版本。](https://web.archive.org/web/20220626074048/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules)