# 用 Gradle 构建 Java 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-building-a-java-app>

## 1.概观

本教程提供了如何使用 Gradle 构建基于 Java 的项目的实用指南。

我们将解释手动创建项目结构、执行初始配置以及添加 Java 插件和 JUnit 依赖项的步骤。然后，我们将构建并运行应用程序。

最后，在最后一节中，我们将给出一个例子来说明如何使用 Gradle Build Init 插件实现这一点。一些基本的介绍也可以在文章[Gradle](/web/20220626194841/https://www.baeldung.com/gradle)介绍中找到。

## 2.Java 项目结构

在我们手动创建一个 Java 项目并准备构建之前，我们需要[安装 Gradle](https://web.archive.org/web/20220626194841/https://docs.gradle.org/current/userguide/installation.html) 。

让我们开始使用名为`gradle-employee-app`的 PowerShell 控制台创建一个项目文件夹:

```java
> mkdir gradle-employee-app
```

之后，让我们导航到项目文件夹并创建子文件夹:

```java
> mkdir src/main/java/employee
```

结果输出如下所示:

```java
Directory: D:\gradle-employee-app\src\main\java

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        4/10/2020  12:14 PM                employee
```

在上面的项目结构中，让我们创建两个类。一个是简单的`Employee`类，包含姓名、电子邮件地址和出生年份等数据:

```java
public class Employee {
    String name;
    String emailAddress;
    int yearOfBirth;
}
```

第二个是打印`Employee `数据的主`Employee App`类:

```java
public class EmployeeApp {

    public static void main(String[] args){
        Employee employee = new Employee();

        employee.name = "John";
        employee.emailAddress = "[[email protected]](/web/20220626194841/https://www.baeldung.com/cdn-cgi/l/email-protection)";
        employee.yearOfBirth = 1978;

        System.out.println("Name: " + employee.name);
        System.out.println("Email Address: " + employee.emailAddress);
        System.out.println("Year Of Birth:" + employee.yearOfBirth);
    }
}
```

## 3.构建一个 Java 项目

**接下来，为了让** **构建我们的 Java 项目，我们在项目根文件夹**中创建一个`build.gradle `配置文件。

以下是 PowerShell 命令行中的内容:

```java
Echo > build.gradle
```

我们跳过与输入参数相关的下一步:

```java
cmdlet Write-Output at command pipeline position 1
Supply values for the following parameters:
InputObject[0]:
```

为了使构建成功，我们需要添加 [**应用程序插件**](https://web.archive.org/web/20220626194841/https://docs.gradle.org/current/userguide/application_plugin.html) :

```java
plugins {
    id 'application'
}
```

然后，我们应用一个应用程序插件， **添加一个主类的全限定名** :

```java
apply plugin: 'application'
mainClassName = 'employee.EmployeeApp'
```

每个项目由`tasks`组成。任务表示构建执行的一项工作，如编译源代码。

例如，我们可以在配置文件中添加一个任务，打印一条关于已完成项目配置的消息:

```java
println 'This is executed during configuration phase'
task configured {
    println 'The project is configured'
}
```

**通常情况下，`gradle build`是首要任务，也是使用最多的一个。这个任务编译、测试和汇编代码到一个 JAR 文件**。通过键入以下命令开始构建:

```java
> gradle build 
```

执行上面的命令以输出:

```java
> Configure project :
This is executed during configuration phase
The project is configured
BUILD SUCCESSFUL in 1s
2 actionable tasks: 2 up-to-date
```

**要查看构建结果，让我们看看构建文件夹，其中包含子文件夹:** **类、发行版、库和报告**。键入`Tree / F`给出构建文件夹的结构:

```java
├───build
│   ├───classes
│   │   └───java
│   │       ├───main
│   │       │   └───employee
│   │       │           Employee.class
│   │       │           EmployeeApp.class
│   │       │
│   │       └───test
│   │           └───employee
│   │                   EmployeeAppTest.class
│   │
│   ├───distributions
│   │       gradle-employee-app.tar
│   │       gradle-employee-app.zip

│   ├───libs
│   │       gradle-employee-app.jar
│   │
│   ├───reports
│   │   └───tests
│   │       └───test
│   │           │   index.html
│   │           │
│   │           ├───classes
│   │           │       employee.EmployeeAppTest.html
```

如您所见，`classes`子文件夹包含我们之前创建的两个编译过的`.class`文件。`distributions`子文件夹包含应用程序 jar 包的归档版本。而`l` `ibs`保存我们应用程序的 jar 文件。

通常，在运行 JUnit 测试时会生成一些文件。

现在，通过键入`gradle run.`退出时执行应用程序的结果:，一切都准备好了，可以运行 Java 项目

```java
> Configure project :
This is executed during configuration phase
The project is configured

> Task :run
Name: John
Email Address: [[email protected]](/web/20220626194841/https://www.baeldung.com/cdn-cgi/l/email-protection)
Year Of Birth:1978

BUILD SUCCESSFUL in 1s
2 actionable tasks: 1 executed, 1 up-to-date 
```

### 3.1.使用 Gradle 包装器构建

Gradle 包装器是一个调用 Gradle 声明版本的脚本。

首先，让我们在`build.gradle`文件中定义一个包装器任务:

```java
task wrapper(type: Wrapper){
    gradleVersion = '5.3.1'
}
```

让我们使用 Power Shell 中的`gradle wrapper`来运行这个任务:

```java
> Configure project :
This is executed during configuration phase
The project is configured

BUILD SUCCESSFUL in 1s
1 actionable task: 1 executed
```

项目文件夹下会创建几个文件，包括`/gradle/wrapper`位置下的文件:

```java
│   gradlew
│   gradlew.bat
│   
├───gradle
│   └───wrapper
│           gradle-wrapper.jar
│           gradle-wrapper.properties
```

*   `gradlew`:用于在 Linux 上创建 Gradle 任务的 shell 脚本
*   `gradlew.bat`:一个`.bat`脚本，让 Windows 用户创建梯度任务
*   我们的应用程序的一个包装器可执行的 jar
*   `gradle-wrapper.properties`:配置包装器的属性文件

## 4.添加 Java 依赖项并运行一个简单的测试

首先，在我们的配置文件中，我们需要设置一个远程存储库，从那里下载依赖关系 jar。大多数情况下，这些存储库要么是`mavenCentral()`要么是`jcenter()`。让我们选择第二个:

```java
repositories {
    jcenter()
}
```

创建了存储库之后，我们就可以指定下载哪些依赖项了。在这个例子中，我们添加了 Apache Commons 和 JUnit 库。**要实现，在依赖配置**中添加`testImplementation`和`testRuntime`部件。

它建立在一个额外的测试块上:

```java
dependencies {
    compile group: 'org.apache.commons', name: 'commons-lang3', version: '3.12.0'
    testImplementation('junit:junit:4.13')
    testRuntime('junit:junit:4.13')
}
test {
    useJUnit()
}
```

完成后，让我们在一个简单的测试上尝试 JUnit 的工作。导航到`src`文件夹，并为测试创建子文件夹:

```java
src> mkdir test/java/employee
```

在最后一个子文件夹中，让我们创建`EmployeeAppTest.java`:

```java
public class EmployeeAppTest {

    @Test
    public void testData() {

        Employee testEmp = this.getEmployeeTest();
        assertEquals(testEmp.name, "John");
        assertEquals(testEmp.emailAddress, "[[email protected]](/web/20220626194841/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        assertEquals(testEmp.yearOfBirth, 1978);
    }

    private Employee getEmployeeTest() {

        Employee employee = new Employee();
        employee.name = "John";
        employee.emailAddress = "[[email protected]](/web/20220626194841/https://www.baeldung.com/cdn-cgi/l/email-protection)";
        employee.yearOfBirth = 1978;

        return employee;
    }
}
```

与之前类似，让我们从命令行运行一个`gradle clean test`，测试应该会顺利通过。

## 5.使用 Gradle 初始化 Java 项目

在这一节中，我们将解释到目前为止创建和构建 Java 应用程序的步骤。不同的是，这一次，我们在 Gradle Build Init 插件的帮助下工作。

创建一个新的项目文件夹并将其命名为`gradle-java-example.`,然后切换到那个空的项目文件夹并运行 init 脚本:

```java
> gradle init
```

Gradle 会问我们一些问题，并提供创建项目的选项。第一个问题是我们想要生成什么类型的项目:

```java
Select type of project to generate:
  1: basic
  2: cpp-application
  3: cpp-library
  4: groovy-application
  5: groovy-library
  6: java-application
  7: java-library
  8: kotlin-application
  9: kotlin-library
  10: scala-library
Select build script DSL:
  1: groovy
  2: kotlin
Enter selection [1..10] 6
```

**为项目类型选择选项 6，然后为构建脚本**选择第一个选项(groovy)。

接下来，会出现一个问题列表:

```java
Select test framework:
  1: junit
  2: testng
  3: spock
Enter selection (default: junit) [1..3] 1

Project name (default: gradle-java-example):
Source package (default: gradle.java.example): employee

BUILD SUCCESSFUL in 57m 45s
2 actionable tasks: 2 executed
```

这里，我们选择第一个选项 junit 作为测试框架。为我们的项目选择默认名称，并键入“employee”作为源包的名称。

要查看`/src`项目文件夹中的完整目录结构，让我们在 Power Shell 中键入`Tree /F` :

```java
├───main
│   ├───java
│   │   └───employee
│   │           App.java
│   │
│   └───resources
└───test
    ├───java
    │   └───employee
    │           AppTest.java
    │
    └───resources
```

最后，如果我们用`gradle run,` 构建项目，我们在退出时得到`“Hello World”`:

```java
> Task :run
Hello world.

BUILD SUCCESSFUL in 1s
2 actionable tasks: 1 executed, 1 up-to-date
```

## 6.结论

在本文中，我们介绍了两种使用 Gradle 创建和构建 Java 应用程序的方法。事实是，我们做了手工工作，从命令行开始编译和构建应用程序需要时间。在这种情况下，如果应用程序使用多个库，我们应该注意一些必需的包和类的导入。

另一方面， **Gradle `init`脚本具有生成我们项目的轻量级框架的特性，以及一些与 Gradle** 相关的配置文件。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626194841/https://github.com/eugenp/tutorials/tree/master/gradle/gradle-employee-app)