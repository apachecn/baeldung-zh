# maven 人工制品分类指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-artifact-classifiers>

## 1.概观

在本教程中，我们将看看 Maven 工件分类器的作用。接下来，我们将看看它们可能有用的各种场景。在结束之前，我们还将简要讨论 Maven 工件类型。

## 2.什么是 Maven 工件分类器？

**一个 Maven 工件分类器是** **一个可选的任意字符串**，它被附加到生成的工件名称的版本号之后。它区分了由相同的 POM 构建但内容不同的工件。

让我们考虑工件的定义:

```java
<groupId>com.baeldung</groupId>
<artifactId>maven-classifier-example-provider</artifactId>
<version>0.0.1-SNAPSHOT</version>
```

为此，Maven jar 插件生成`maven-classifier-example-provider-0.0.1-SNAPSHOT.jar` `.`

### 2.1.用分类器生成工件

让我们向 jar 插件添加一个分类器配置:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.2.2</version>
            <executions>
                <execution>
                    <id>Arbitrary</id>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                    <configuration>
                        <classifier>arbitrary</classifier>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

结果，产生了`maven-classifier-example-provider-0.0.1-SNAPSHOT-arbitrary.jar` 和`maven-classifier-example-provider-0.0.1-SNAPSHOT.jar` 。

### 2.2.使用分类器消费工件

我们现在能够使用分类器生成带有自定义后缀的工件。让我们看看如何使用分类器来消费相同的内容。

为了消费默认工件，我们不需要做任何特殊的事情:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>maven-classifier-example-provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

但是，要使用带有自定义后缀“arbitrary”的工件，我们需要在`dependency`元素中使用`classifier`元素:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>maven-classifier-example-provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <classifier>arbitrary</classifier>
</dependency>
```

做完这些之后，一个好问题是为什么我们要使用分类器？因此，让我们来看一些分类器有用的实际用例。

## 3.使用源分类器

我们可能已经注意到，每个 Maven 工件通常都伴随着一个“`-sources.jar”`工件。它包含源代码，即主工件的`.java`文件。这对于调试非常有用。

### 3.1.生成源工件

首先，让我们为`maven-classifier-example-provider`模块生成一个 sources jar。我们将使用`maven-source-plugin` 来做到这一点:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>3.2.1</version>
    <executions>
        <execution>
            <id>attach-sources</id>
            <phase>verify</phase>
            <goals>
                <goal>jar-no-fork</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

现在，让我们运行:

```java
mvn clean install
```

结果，产生了`maven-classifier-example-provider-0.0.1-SNAPSHOT-sources.jar` 工件`.`

### 3.2.消费源工件

现在，要使用 sources 工件，[有几种方法](/web/20220524053151/https://www.baeldung.com/maven-download-sources-javadoc)。让我们来看看其中的一个。我们可以在依赖关系定义中使用一个分类器元素来提取源 jar 以获得选择性依赖关系:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>maven-classifier-example-producer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <classifier>sources</classifier>
</dependency>
```

现在，让我们运行:

```java
mvn clean install
```

结果，获取了这个特定依赖项的源 jar。一般来说，我们不这样做，因为它会不必要地污染 POM。因此，我们通常更喜欢使用 IDEs 的功能来按需附加源 jar。或者，我们也可以通过`mvn` CLI 命令有选择地获取它们:

```java
mvn dependency:sources -DincludeArtifactIds=maven-classifier-example-provider
```

**简而言之，这里的关键要点是通过使用`sources` 分类器来引用 sources jar。**

## 4.使用 Javadoc 分类器

与 sources jar 用例一样，我们有 Javadoc。它包含主 jar 工件的文档。

### 4.1.生成 Javadoc 工件

让我们使用`maven-javadoc-plugin` 来生成 Javadoc 工件:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>3.3.2</version>
    <executions>
        <execution>
            <id>attach-javadocs</id>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

定义好插件后，让我们运行:

```java
mvn clean install
```

结果，产生了`maven-classifier-example-provider-0.0.1-SNAPSHOT-javadoc.jar `工件。

### 4.2.消费 Javadoc 工件

现在让我们从消费者的角度通过`mvn`下载生成的 Javadoc:

```java
mvn dependency:resolve -Dclassifier=javadoc -DincludeArtifactIds=maven-classifier-example-provider
```

**我们可以注意到`-Dclassifier=javadoc` 选项将分类器指定为`javadoc`。**

## 5.使用测试分类器

现在，让我们看一个分类器可以服务的不同用例。我们可能有各种各样的原因想要得到一个模块的 tests jar。首先，我们可能想要重用某些测试存根。让我们看看如何通过一个分类器独立于主 jar 工件对 jar 进行测试。

### 5.1.生成测试 Jar

首先，让我们为`maven-classifier-example-provider`模块生成测试 jar。为了做到这一点，我们将通过指定`test-jar`目标来使用 Maven jar 插件:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.2</version>
    <executions>
        <execution>
            <id>Test Jar</id>
            <goals>
                <goal>test-jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

现在，让我们运行:

```java
mvn clean install
```

结果，生成并安装了测试 jar `maven-classifier-example-provider-0.0.1-SNAPSHOT-tests.jar`。

### 5.2.消费测试罐

现在，让我们将测试 jar 依赖关系放入我们的消费者模块`maven-classifier-example-consumer`。我们使用`tests`分类器来完成这项工作:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>maven-classifier-example-provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <classifier>tests</classifier>
</dependency>
```

这样，我们应该能够访问`maven-classifier-example-provider` `. `测试包中可用的类

## 6.使用分类器支持多个 Java 版本

之前，我们使用了一个任意的分类器来为我们的`maven-classifier-example-provider`模块构建第二个 jar。现在让我们把它投入到更实际的应用中。

Java 现在以 6 个月一次的速度发布了新版本。因此，它要求模块开发者能够支持多个版本的 Java。为用不同 Java 版本编译的模块构建多个 jar 听起来可能是一项具有挑战性的任务。这就是 Maven 分类器的用处。我们可以使用单个 POM 构建用不同 Java 版本编译的多个 jar。

### 6.1.使用多个 Java 版本编译

首先，我们将配置 Maven 编译器插件，使用 JDK 8 和 JDK 11 生成编译后的类文件:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.10.0</version>
    <executions>
        <execution>
            <id>JDK 8</id>
            <phase>compile</phase>
            <goals>
                <goal>compile</goal>
            </goals>
            <configuration>
                <source>8</source>
                <target>8</target>
                <fork>true</fork>
            </configuration>
        </execution>
        <execution>
            <id>JDK 11</id>
            <phase>compile</phase>
            <goals>
                <goal>compile</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.build.outputDirectory}_jdk11</outputDirectory>
                <executable>${jdk.11.executable.path}</executable>
                <source>8</source>
                <target>11</target>
                <fork>true</fork>
            </configuration>
        </execution>
    </executions>
</plugin>
```

我们设置了两个执行块，一个用于 Java 8，另一个用于 Java 11。

接下来，我们已经为 Java 11 配置了不同的`outputDirectory `。另一方面，Java 8 编译将使用默认的`outputDirectory`。

接下来是源和目标 Java 版本配置。源代码是使用 Java 8 编写的，因此我们将源版本保持为 8，并将 Java 11 执行块的目标版本修改为 11。

我们还明确提供了 Java 11 的编译器的可执行路径`javac.`对于 Java 8，Maven 将使用系统上配置的默认`javac`，在本例中是 Java 8 的`javac`:

```java
mvn clean compile
```

**因此，在目标文件夹中会生成两个文件夹——`classes`和`classes_jdk11`** 。

### 6.2.为多个 Java 版本生成 Jar 工件

其次，我们可以使用 Maven jar 插件将模块打包到两个独立的 jar 中:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.2</version>
    <executions>
        <execution>
            <id>default-package-jdk11</id>
            <phase>package</phase>
            <goals>
                <goal>jar</goal>
            </goals>
            <configuration>
                <classesDirectory>${project.build.outputDirectory}_jdk11</classesDirectory>
                <classifier>jdk11</classifier>
            </configuration>
        </execution>
    </executions>
</plugin>
```

**我们可以注意到分类器被设置为 jdk11** 。

现在，让我们运行:

```java
mvn clean install
```

结果，生成了两个 jar—`maven-classifier-example-provider-0.0.1-SNAPSHOT-jdk11.jar` 和`maven-classifier-example-provider-0.0.1-SNAPSHOT.jar.` ,第一个使用 Java 11 编译器编译，第二个使用 Java 8 编译器编译。

### 6.3.消费特定 Java 版本的 Jar 工件

最后，我们模块的消费者可以根据他们使用的 Java 版本，选择与他们相关的 jar 工件。让我们从消费者的角度来看看如何使用这些罐子。

如果我们正在进行一个 Java 8 项目，我们不需要做任何特别的事情。我们将只添加一个普通的依赖项:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>maven-classifier-example-provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

然而，如果我们正在处理一个 Java 11 项目，我们必须通过一个分类器明确地请求用 Java 11 编译的 jar:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>maven-classifier-example-provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <classifier>jdk11</classifier>
</dependency>
```

## 7.工件分类器与工件类型

在我们结束这篇文章之前，让我们简单地看一下工件类型。虽然工件类型和分类器密切相关，但是它们是不可互换的。工件类型定义了工件的打包格式，例如 POM 或 jar。在大多数情况下，它也转化为文件扩展名。它也可能有相应的分类器。

**换句话说，[工件类型](https://web.archive.org/web/20220524053151/https://maven.apache.org/ref/3.8.4/maven-core/artifact-handlers.html)处理工件的打包，并且可以使用一个分类器来修改所生成工件的名称。**

我们之前看到的 Java 源代码和 Javadoc 这两个用例是使用`sources`和`javadoc`作为分类器的工件类型的例子。

## 8.结论

在本文中，我们看到了分类器如何提供从单个 POM 文件生成多个工件的方法。使用分类器，消费者可以根据需要从特定模块的各种工件中挑选。

与往常一样，代码示例可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220524053151/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-classifier)