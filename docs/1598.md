# 为什么 Maven 没有找到要运行的 JUnit 测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-cant-find-junit-tests>

## 1.概观

在使用 [Maven](/web/20220907090048/https://www.baeldung.com/category/maven/) 项目时，开发人员经常会犯错误，导致 JUnit 测试在构建期间无法运行。Maven 可能找不到要运行的 JUnit 测试有多种原因。在本教程中，我们将看看这些具体情况，并看看如何修复它们。

## 2.命名规格

[Maven Surefire 插件](/web/20220907090048/https://www.baeldung.com/maven-surefire-plugin)在 Maven 构建生命周期的`test`阶段执行单元测试。重要的是，**每当`test`目标被执行**时，Surefire 插件被 Maven 生命周期隐式调用——例如，当运行“`mvn test`或“`mvn install`”时。

**默认情况下，Surefire 插件将自动包含所有带有以下通配符模式的测试类:**

*   “* */测试*。java–包括其所有子目录和所有以“Test”开头的 Java 文件名
*   “* */* Test . java”-包括其所有子目录和所有以“Test”结尾的 Java 文件名
*   “* */* Tests . java”-包括其所有子目录和所有以“Tests”结尾的 Java 文件名
*   “* */* TestCase . java”——包括其所有子目录和所有以“test case”结尾的 Java 文件名

因此，如果我们的测试不遵循上面的通配符模式，Maven 不会选择运行这些测试。然而，在有些情况下，可以遵循特定于项目的命名模式，而不是这些标准约定。在这些情况下，我们可以通过明确指定我们想要包含(或排除)的测试和其他模式来覆盖 Surefire 插件。

例如，让我们考虑下面的`pom.xml`:

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0-M7</version>
            <configuration>
                <includes>
                    <include>**/*_UT.java</include>
                </includes>
                <excludes>
                    <exclude>**/BaseTest.java</exclude>
                    <exclude>**/TestsUtil.java</exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

在这里，我们可以看到，除了标准的命名约定之外，我们还包括了要运行的特定测试。此外，我们已经排除了某些命名模式，这些模式指示插件在测试阶段不要考虑那些测试。

## 3.不正确的依赖关系

**使用 [JUnit 5 平台](/web/20220907090048/https://www.baeldung.com/junit-5)时，我们至少需要添加一个`TestEngine`实现**。例如，如果我们想用 JUnit Jupiter 编写测试，我们需要将测试工件`junit-jupiter-engine`添加到`pom.xml`中的依赖项中:

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0-M7</version>
            <dependencies>
                <dependency>
                    <groupId>org.junit.jupiter</groupId>
                    <artifactId>junit-jupiter-engine</artifactId>
                    <version>5.4.0</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

如果我们想通过 JUnit 平台编写和执行 JUnit 3 或 4 测试，我们需要将 Vintage 引擎添加到依赖项部分:

```
<dependencies>
    <dependency>
        <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
        <version>5.4.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 4.不正确的测试文件夹

将测试放在错误的文件夹中是不考虑执行测试的另一个原因。这主要是由于一些 ide(比如 Eclipse)中默认的类创建路径设置造成的。

为了提供一些背景知识，Maven 定义了一个标准的目录结构:

```
- src
  - main
    - java
    - resources
    - webapp
  - test
    - java
    - resources

- target
```

`main`目录是与应用程序本身相关的源代码的根目录，而不是测试代码。`test`目录包含测试源代码。任何放置在`src/main/java`目录下的测试都将被跳过。相反，**所有的测试和测试资源应该分别放在`src/test /java` 和`src/test/resources`文件夹下**。

例如，源类文件应该放在这里:

```
src/main/java/Calculator.java
```

并且，相应的测试类文件应该放在这里:

```
src/test/java/CalculatorTest.java
```

## 5.测试方法不是`public`

Java 中默认的访问修饰符是 [`package-private`](/web/20220907090048/https://www.baeldung.com/java-access-modifiers#default) 。这意味着所有创建时没有附加访问修饰符的类只对同一个包中的类可见。一些 ide 有标准的配置，因此在创建测试时，它们会被标记为包私有的。此外，测试方法可能被错误地标记为`private`。

在 JUnit 4 之前，Maven 将只运行标记为`public`的测试类。不过，我们应该注意，这在 JUnit 5+中不会成为问题。然而，**实际的测试方法应该总是标为`public`** 。

让我们看一个例子:

```
public class MySampleTest {
    @Test
    private void givenTestCase1_thenPrintTest1() {
        ...
    }
}
```

在这里，我们可以注意到测试方法被标记为`private`。因此，Maven 不会考虑用这种方法来执行测试。将测试方法标记为`public`将使测试能够运行。

## 6.包装类型

Maven 提供了多个选项来[打包](/web/20220907090048/https://www.baeldung.com/maven-packaging-types)应用程序。常见的包装类型有 `jar`、`war`、`ear`和`pom`。如果我们不指定包装值，它将默认为`jar` 包装。每个包都包含一个[目标列表](https://web.archive.org/web/20220907090048/https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#built-in-lifecycle-bindings)来绑定到一个特定的阶段。

**将包装类型标记为`pom`只会将目标绑定到`deploy`和`install`阶段。在这种情况下，将跳过`test`阶段。**这可能是测试没有按预期运行的情况之一。

有时，由于复制粘贴错误，Maven 打包类型可能被标记为`pom`:

```
<packaging>pom</packaging>
```

要解决这个问题，我们需要指定正确的打包类型。例如:

```
<packaging>jar</packaging>
```

## 7.结论

在本文中，我们研究了 Maven 找不到要运行的 JUnit 测试的具体情况。

首先，我们看到了命名约定是如何影响测试执行的。接下来，我们讨论了为在 JUnit 5 平台上执行的测试添加的依赖项。此外，我们还注意到了不正确地将测试放置在不同的文件夹中或者拥有私有的测试方法会如何阻止测试的运行。最后，我们看到了包装类型是如何与每个阶段的特定目标相联系的。