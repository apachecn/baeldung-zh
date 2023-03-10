# 从命令行运行 JUnit 测试用例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-run-from-command-line>

## 1。概述

在本教程中，我们将了解**如何直接从命令行**运行 [JUnit 5 测试](/web/20220625075039/https://www.baeldung.com/junit)。

## 2。测试场景

之前，我们已经介绍了如何以编程方式运行 JUnit 测试。对于我们的示例，我们将使用相同的 JUnit 测试:

```java
public class FirstUnitTest {

    @Test
    public void whenThis_thenThat() {
        assertTrue(true);
    }

    @Test
    public void whenSomething_thenSomething() {
        assertTrue(true);
    }

    @Test
    public void whenSomethingElse_thenSomethingElse() {
        assertTrue(true);
    }
}
```

```java
public class SecondUnitTest {

    @Test
    public void whenSomething_thenSomething() {
        assertTrue(true);
    }

    @Test
    public void whensomethingElse_thenSomethingElse() {
        assertTrue(true);
    }
}
```

## 3。运行 JUnit 5 测试

我们可以使用 JUnit 的控制台启动器运行 JUnit 5 测试用例。这个 jar 的可执行文件可以从 Maven Central 的`[junit-platform-console-standalone](https://web.archive.org/web/20220625075039/https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone)`目录下下载。

此外，我们还需要一个目录来包含所有已编译的类:

```java
$ mkdir target
```

让我们看看如何使用控制台启动器运行不同的测试用例。

### 3.1.运行单个测试类

在运行测试类之前，让我们编译它:

```java
$ javac -d target -cp target:junit-platform-console-standalone-1.7.2.jar src/test/java/com/baeldung/commandline/FirstUnitTest.java
```

现在，我们将使用 Junit 控制台启动器运行编译后的测试类:

```java
$ java -jar junit-platform-console-standalone-1.7.2.jar --class-path target --select-class com.baeldung.commandline.FirstUnitTest
```

这将为我们提供测试运行结果:

```java
Test run finished after 60 ms
[         3 containers found      ]
[         0 containers skipped    ]
[         3 containers started    ]
[         0 containers aborted    ]
[         3 containers successful ]
[         0 containers failed     ]
[         3 tests found           ]
[         0 tests skipped         ]
[         3 tests started         ]
[         0 tests aborted         ]
[         3 tests successful      ]
[         0 tests failed          ] 
```

### 3.2.运行多个测试类

同样，让我们编译我们想要运行的测试类:

```java
$ javac -d target -cp target:junit-platform-console-standalone-1.7.2.jar src/test/java/com/baeldung/commandline/FirstUnitTest.java src/test/java/com/baeldung/commandline/SecondUnitTest.java 
```

我们现在将使用控制台启动器运行编译后的测试类:

```java
$ java -jar junit-platform-console-standalone-1.7.2.jar --class-path target --select-class com.baeldung.commandline.FirstUnitTest --select-class com.baeldung.commandline.SecondUnitTest
```

我们现在的结果表明，所有五种测试方法都是成功的:

```java
Test run finished after 68 ms
...
[         5 tests found           ]
...
[         5 tests successful      ]
[         0 tests failed          ] 
```

### 3.3.  运行一个包中的所有测试类

要运行包中的所有测试类，让我们编译包中的所有测试类:

```java
$ javac -d target -cp target:junit-platform-console-standalone-1.7.2.jar src/test/java/com/baeldung/commandline/*.java
```

同样，我们将运行我们的包中已编译的测试类:

```java
$ java -jar junit-platform-console-standalone-1.7.2.jar --class-path target --select-package com.baeldung.commandline
...
Test run finished after 68 ms
...
[         5 tests found           ]
...
[         5 tests successful      ]
[         0 tests failed          ] 
```

### 3.4.运行所有的测试类

让我们运行所有的测试用例:

```java
$ java -jar junit-platform-console-standalone-1.7.2.jar --class-path target  --scan-class-path
...
Test run finished after 68 ms
...
[         5 tests found           ]
...
[         5 tests successful      ]
[         0 tests failed          ] 
```

## 4.使用 Maven 运行 JUnit

如果我们使用 [**Maven 作为我们的构建工具**](/web/20220625075039/https://www.baeldung.com/maven) ，我们可以直接从命令行执行测试用例。

### 4.1.运行单个测试用例

要在控制台上运行单个测试用例，让我们通过指定测试类名来执行以下命令:

```java
$ mvn test -Dtest=SecondUnitTest 
```

这将为我们提供测试运行结果:

```java
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.069 s - in com.baeldung.commandline.SecondUnitTest 
[INFO] 
[INFO] Results:
[INFO]
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------ 
[INFO] BUILD SUCCESS 
[INFO] ------------------------------------------------------------------------ 
[INFO] Total time: 7.211 s [INFO] Finished at: 2021-08-02T23:13:41+05:30
[INFO] ------------------------------------------------------------------------
```

### 4.2.运行多个测试用例

为了在控制台上运行多个测试用例，让我们执行命令，指定我们想要执行的所有测试类的名称:

```java
$ mvn test -Dtest=FirstUnitTest,SecondUnitTest
...
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.069 s - in com.baeldung.commandline.SecondUnitTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.069 s - in com.baeldung.commandline.FirstUnitTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.211 s
[INFO] Finished at: 2021-08-02T23:13:41+05:30
[INFO] ------------------------------------------------------------------------ 
```

### 4.3.运行包中的所有测试用例

要在控制台上运行一个包中的所有测试用例，我们需要在命令中指定包名:

```java
$ mvn test -Dtest="com.baeldung.commandline.**"
...
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.069 s - in com.baeldung.commandline.SecondUnitTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.069 s - in com.baeldung.commandline.FirstUnitTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.211 s
[INFO] Finished at: 2021-08-02T23:13:41+05:30
[INFO] ------------------------------------------------------------------------ 
```

### 4.4.运行所有测试用例

最后，为了在控制台上使用 Maven 运行所有测试用例，我们只需执行`mvn clean test`:

```java
$ mvn clean test
...
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.069 s - in com.baeldung.commandline.SecondUnitTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.069 s - in com.baeldung.commandline.FirstUnitTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.211 s
[INFO] Finished at: 2021-08-02T23:13:41+05:30
[INFO] ------------------------------------------------------------------------ 
```

## 5.结论

在本文中，我们已经了解了如何直接从命令行运行 JUnit 测试，包括有和没有 Maven 的 JUnit 5。

GitHub 上的[提供了此处所示示例的实现。](https://web.archive.org/web/20220625075039/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-advanced)