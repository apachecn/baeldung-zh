# 从命令行运行 TestNG 项目

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/testng-run-command-line>

## 1.概观

在这个简短的教程中，我们将看到如何从命令行启动 [TestNG](/web/20220627171049/https://www.baeldung.com/testng) 测试。这对于构建或者如果我们想在开发期间直接运行一个单独的测试是很有用的。
我们可以使用像 [Maven](/web/20220627171049/https://www.baeldung.com/maven) 这样的构建工具来执行我们的测试，或者我们可能希望通过`java`命令直接运行它们。让我们来看看这两种方法。

## 2.示例项目概述

对于我们的示例，让我们使用一些包含一个将日期格式化为字符串的服务的代码:

```java
public class DateSerializerService {
    public String serializeDate(Date date, String format) {
        SimpleDateFormat dateFormat = new SimpleDateFormat(format);
        return dateFormat.format(date);
    }
} 
```

对于测试，让我们用一个测试来检查当一个`null`日期被传递给服务时是否抛出了一个`NullPointerExeception` :

```java
@Test(testName = "Date Serializer")
public class DateSerializerServiceUnitTest {
    private DateSerializerService toTest;

    @BeforeClass
    public void beforeClass() {
        toTest = new DateSerializerService();
    }

    @Test(expectedExceptions = { NullPointerException.class })
    void givenNullDate_whenSerializeDate_thenThrowsException() {
        Date dateToTest = null;

        toTest.serializeDate(dateToTest, "yyyy/MM/dd HH:mm:ss.SSS");
    }
}
```

我们还将**创建一个`pom.xml`，它定义了从命令行**执行 TestNG 所需的依赖关系。我们需要的第一个依赖项是 [TestNG](https://web.archive.org/web/20220627171049/https://mvnrepository.com/artifact/org.testng/testng) :

```java
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>7.4.0</version>
    <scope>test</scope>
</dependency>
```

接下来，我们需要 [JCommander](https://web.archive.org/web/20220627171049/https://mvnrepository.com/artifact/com.beust/jcommander) 。TestNG 用它来解析命令行:

```java
<dependency>
    <groupId>com.beust</groupId>
    <artifactId>jcommander</artifactId>
    <version>1.81</version>
    <scope>test</scope>
</dependency>
```

最后，如果我们希望 TestNG 编写 HTML 测试报告，我们需要为 JQuery 添加 [WebJar 依赖项:](https://web.archive.org/web/20220627171049/https://mvnrepository.com/artifact/org.webjars/jquery)

```java
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.5.1</version>
    <scope>test</scope>
</dependency>
```

## 3.运行 TestNG 命令的设置

### 3.1.使用 Maven 下载依赖项

因为我们有一个 Maven 项目，所以让我们来构建它:

```java
c:\> mvn test
```

该命令应该输出:

```java
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------< com.baeldung.testing_modules:testng_command_line >----------
[INFO] Building testng_command_line 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  5.639 s
[INFO] Finished at: 2021-12-19T15:16:52+01:00
[INFO] ------------------------------------------------------------------------ 
```

现在，我们已经拥有了从命令行运行 TestNG 测试所需的一切。
所有的依赖项都将被下载到 Maven 本地存储库中，该存储库通常位于用户的`.m2`文件夹中。

### 3.2.获取我们的类路径

**要通过`java`命令执行命令，我们需要添加一个`-classpath`选项**:

```java
$ java -cp "~/.m2/repository/org/testng/testng/7.4.0/testng-7.4.0.jar;~/.m2/repository/com/beust/jcommander/1.81/jcommander-1.81.jar;~/.m2/repository/org/webjars/jquery/3.5.1/jquery-3.5.1.jar;target/classes;target/test-classes" org.testng.TestNG ...
```

在稍后的命令行示例中，我们将把它缩写为`-cp <CLASSPATH>`。

## 4.检查 TestNG 命令行

让我们检查一下是否可以通过`java`访问 TestNG:

```java
$ java -cp <CLASSPATH> org.testng.TestNG
```

如果一切正常，控制台将显示一条消息:

```java
You need to specify at least one testng.xml, one class or one method
Usage: <main class> [options] The XML suite files to run
Options:
...
```

## 5.启动测试 NG 单一测试

### 5.1.使用`java`命令运行单个测试

**现在，我们可以快速运行单个测试**，而无需配置单个测试套件文件，只需使用以下命令行:

```java
$ java -cp <CLASSPATH> org.testng.TestNG -testclass "com.baeldung.testng.DateSerializerServiceUnitTest"
```

### 5.2.用 Maven 运行单个测试

如果我们希望 Maven 只执行这个测试，我们可以在`pom.xml`文件中配置`maven-surefire-plugin`:

```java
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <includes>
                        <include>**/DateSerializerServiceUnitTest.java</include>
                    </includes>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

在这个例子中，我们有一个名为`ExecuteSingleTest`的概要文件，配置为执行`DateSerializerServiceUnitTest.java.` ,我们可以运行这个概要文件:

```java
$ mvn -P ExecuteSingleTest test
```

正如我们所见， **Maven 需要比简单的 TestNG 命令行执行更多的配置来执行一个测试**。

## 6.启动 TestNG 测试套件

### 6.1.用`java`命令运行测试套件

测试套件文件定义了测试应该如何运行。我们需要多少就有多少。并且，**我们可以通过指向定义测试套件**的`XML` 文件来运行测试套件:

```java
$ java -cp <CLASSPATH> org.testng.TestNG testng.xml
```

### 6.2.使用 Maven 运行测试套件

如果我们想使用 Maven 执行测试套件，我们应该**配置插件`maven-surefire-plugin`** :

```java
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <suiteXmlFiles>
                        <suiteXmlFile>testng.xml</suiteXmlFile>
                    </suiteXmlFiles>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

这里，我们有一个名为`ExecuteTestSuite`的 Maven 概要文件，它将配置*Maven-surefire 插件*来启动`testng.xml `测试套件。我们可以使用以下命令运行该配置文件:

```java
$ mvn -P ExecuteTestSuite test
```

## 7.结论

在本文中，我们看到了**TestNG 命令行如何有助于运行单个测试文件，而`Maven`应该用于配置和启动全套测试**。
和往常一样，这篇文章的示例代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627171049/https://github.com/eugenp/tutorials/tree/master/testing-modules/testng-command-line)