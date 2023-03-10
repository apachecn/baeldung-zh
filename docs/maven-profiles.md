# Maven 配置文件指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-profiles>

## 1.概观

**[Maven](/web/20220926182013/https://www.baeldung.com/maven) 概要文件可以用来创建定制的构建配置**，比如针对测试粒度级别或特定的部署环境。

在本教程中，我们将学习如何使用 Maven 概要文件。

## 2.一个基本例子

通常**当我们运行`mvn package,`时，单元测试也会被执行**。但是**如果我们想快速打包工件并运行**来看看它是否工作呢？

首先，我们将创建一个`no-tests`概要文件，它将`maven.test.skip`属性设置为`true:`

```java
<profile>
    <id>no-tests</id>
    <properties>
        <maven.test.skip>true</maven.test.skip>
    </properties>
</profile>
```

接下来，我们将通过运行`mvn package -Pno-tests`命令来执行概要文件。**现在工件被创建，测试被跳过。**在这种情况下，`mvn package -Dmaven.test.skip`命令会更简单。

然而，这只是对 Maven 概要文件的介绍。让我们来看看一些更复杂的设置。

## 3.声明配置文件

在上一节中，我们看到了如何创建一个概要文件。**我们可以通过给他们唯一的 id 来配置任意多的配置文件**。

假设我们想要创建一个只运行我们的集成测试的概要文件，以及另一个用于一组突变测试的概要文件。

我们将从在我们的`pom.xml` 文件中为每一个指定一个`id `开始:

```java
<profiles>
    <profile>
        <id>integration-tests</id>
    </profile>
    <profile>
        <id>mutation-tests</id>
    </profile>
</profiles>
```

在每个`profile`、**元素中，我们可以配置很多元素，如`dependencies`、`plugins`、`resources`、`finalName`、**。

因此，对于上面的例子，我们可以分别为`integration-tests`和`mutation-tests`添加插件及其依赖项。

将测试分离到概要文件中可以使默认构建更快，比如说，只关注单元测试。

### 3.1.配置文件范围

现在，我们只是将这些概要文件放在我们的`pom.xml` 文件中，该文件只为我们的项目声明它们。

但是，在 Maven 3 中，我们实际上可以将概要文件添加到三个位置中的任何一个:

1.  特定于项目的概要文件进入项目的`pom.xml `文件
2.  用户特定的配置文件进入用户的`settings.xml`文件
3.  全局配置文件进入全局`settings.xml`文件

注意，Maven 2 确实支持第四个位置，但是在 Maven 3 中已经删除了。

**我们尽可能在`pom.xml`中配置配置文件。**原因是我们希望在开发机器和构建机器上都使用这些概要文件。使用`settings.xml`更加困难并且容易出错，因为我们必须自己在构建环境中分发它。

## 4.激活配置文件

在我们创建了一个或多个概要文件**之后，我们可以开始使用它们，或者换句话说，`activating`它们。**

### 4.1.查看哪些配置文件是活动的

**让我们使用`help:active-profiles` 目标来查看在我们的默认构建中哪些概要文件是活动的**:

```java
mvn help:active-profiles
```

实际上，由于我们还没有激活任何东西，我们得到:

```java
The following profiles are active:
```

嗯，没什么。

我们马上就会激活它们。但是很快，另一种查看什么被激活的方法是**将`maven-help-plugin`包含在我们的`pom.xml `中，并将`active-profiles`目标与`compile `阶段:**

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-help-plugin</artifactId>
            <version>3.2.0</version>
            <executions>
                <execution>
                    <id>show-profiles</id>
                    <phase>compile</phase>
                    <goals>
                        <goal>active-profiles</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

现在，让我们开始使用它们吧！我们来看看几种不同的方式。

### 4.2.使用`-P`

实际上，我们在开始时已经看到了一种方法，那就是我们可以用`-P`参数来**激活概要文件。**

因此，让我们从启用`integration-tests`配置文件开始:

```java
mvn package -P integration-tests
```

如果我们使用`maven-help-plugin`或`mvn help:active-profiles -P integration-tests`命令来验证活动配置文件，我们将得到以下结果:

```java
The following profiles are active:

 - integration-tests
```

如果我们想同时激活多个配置文件，我们使用逗号分隔的配置文件列表:

```java
mvn package -P integration-tests,mutation-tests
```

### 4.3.默认情况下处于活动状态

如果我们总是想要执行一个概要文件，我们可以在默认情况下激活一个概要文件:

```java
<profile>
    <id>integration-tests</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
</profile>
```

然后，我们可以在不指定概要文件的情况下运行`mvn package`，并且我们可以验证 `integration-test`概要文件是活动的。

**然而，如果我们运行 Maven 命令并启用另一个概要文件，那么`activeByDefault`概要文件将被跳过。**所以当我们运行`mvn package -P mutation-tests`时，只有`mutation-tests`配置文件是活动的。

当我们以其他方式激活时，`activeByDefault`配置文件也会被跳过，我们将在接下来的章节中看到。

### 4.4.基于属性

我们可以在命令行上激活配置文件。然而，有时如果它们被自动激活会更方便。例如，**我们可以基于一个`-D` 系统属性:**

```java
<profile>
    <id>active-on-property-environment</id>
    <activation>
        <property>
            <name>environment</name>
        </property>
    </activation>
</profile>
```

我们现在用`mvn package -Denvironment`命令激活轮廓。

如果属性不存在，也可以激活配置文件:

```java
<property>
    <name>!environment</name>
</property>
```

或者，如果属性具有特定值:，我们可以激活配置文件

```java
<property>
    <name>environment</name>
    <value>test</value>
</property>
```

我们现在可以使用`mvn package -Denvironment=test.`运行配置文件

最后，如果属性的值不是指定的值，我们可以激活配置文件:

```java
<property>
    <name>environment</name>
    <value>!test</value>
</property>
```

### 4.5.基于`JDK`版本

另一个选项是启用基于计算机上运行的 JDK 的配置文件。在这种情况下，如果 JDK 版本以 11:

```java
<profile>
    <id>active-on-jdk-11</id>
    <activation>
        <jdk>11</jdk>
    </activation>
</profile>
```

我们也可以为 JDK 版本使用范围，如 [Maven 版本范围语法](/web/20220926182013/https://www.baeldung.com/maven-dependency-latest-version)中所解释的。

### 4.6.基于操作系统

或者，我们可以根据一些操作系统信息激活配置文件。

如果我们不确定，我们可以首先使用`mvn enforcer:display-info`命令，它会在我的机器上给出以下输出:

```java
Maven Version: 3.5.4
JDK Version: 11.0.2 normalized as: 11.0.2
OS Info: Arch: amd64 Family: windows Name: windows 10 Version: 10.0
```

之后，我们可以配置一个仅在 Windows 10 上激活的配置文件:

```java
<profile>
    <id>active-on-windows-10</id>
    <activation>
        <os>
            <name>windows 10</name>
            <family>Windows</family>
            <arch>amd64</arch>
            <version>10.0</version>
        </os>
    </activation>
</profile>
```

### 4.7.基于一份文件

另一个选择是运行一个配置文件**，如果一个文件`exists`或者是`missing`。**

因此，让我们创建一个仅在`testreport.html`不存在时才执行的测试概要文件:

```java
<activation>
    <file>
        <missing>target/testreport.html</missing>
    </file>
</activation>
```

## 5.停用配置文件

我们已经看到了许多激活配置文件的方法，但有时我们也需要禁用其中一种。

要禁用配置文件，我们可以使用'！'或者'-'。

因此，为了禁用`active-on-jdk-11`概要文件，我们执行`mvn compile -P -active-on-jdk-11`命令。

## 6.结论

在本文中，我们已经看到了如何使用 Maven 概要文件，因此我们可以创建不同的构建配置。

这些概要文件有助于在我们需要的时候执行构建的特定元素。这优化了我们的构建过程，并有助于向开发人员提供更快的反馈。

在 [GitHub](https://web.archive.org/web/20220926182013/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-simple/maven-profiles) 上随意看看完成的`pom.xml`文件。**