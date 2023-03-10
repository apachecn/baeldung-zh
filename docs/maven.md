# Apache Maven 教程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven>

## 1。简介

构建一个软件项目通常包括以下任务:下载依赖项、将附加的 JAR 放到类路径上、将源代码编译成二进制代码、运行测试、将编译后的代码打包成可部署的工件(如 JAR、WAR 和 ZIP 文件),以及将这些工件部署到应用服务器或存储库。

Apache Maven 自动执行这些任务，最大限度地降低人工构建软件时出现错误的风险，并将编译和打包代码的工作与代码构建的工作分开。

在本教程中，我们将探索这个强大的工具，它使用一条用 XML 编写的中心信息——`Project Object Model` (POM)来描述、构建和管理 Java 软件项目。

## 2。为什么要用 Maven？

Maven 的主要特性是:

*   **遵循最佳实践的简单项目设置:** Maven 通过提供项目模板(名为`archetypes`)来尽量避免配置
*   **依赖管理:**包括自动更新、下载和验证兼容性，以及报告依赖闭包(也称为传递依赖)
*   **项目依赖关系和插件之间的隔离:**使用 Maven，项目依赖关系从`dependency repositories`中检索，而任何插件的依赖关系都从`plugin repositories,`中检索，从而减少了插件开始下载附加依赖关系时的冲突
*   **中央存储库系统:**项目依赖可以从本地文件系统或者公共存储库中加载，比如 [Maven Central](https://web.archive.org/web/20220629003119/https://search.maven.org/classic/)

**In order to learn how to install Maven on your system, please check [this tutorial on Baeldung](/web/20220629003119/https://www.baeldung.com/install-maven-on-windows-linux-mac).**

## 3。项目对象模型

Maven 项目的配置是通过一个由`pom.xml`文件表示的`Project Object Model (POM)`来完成的。`POM`描述项目，管理依赖关系，并为构建软件配置插件。

`POM`还定义了多模块项目的模块之间的关系。让我们看看典型的`POM`文件的基本结构:

```java
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.baeldung</groupId>
    <artifactId>baeldung</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>com.baeldung</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.8.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
            //...
            </plugin>
        </plugins>
    </build>
</project>
```

让我们仔细看看这些结构。

### 3.1。项目标识符

Maven 使用一组标识符(也称为坐标)来惟一地标识项目，并指定项目工件应该如何打包:

*   `groupId`–创建项目的公司或集团的唯一基本名称
*   `artifactId`–项目的唯一名称
*   `version`–项目的一个版本
*   `packaging`–一种包装方式(如`WAR` / `JAR` / `ZIP`)

其中的前三个(`groupId:artifactId:version`)组合起来形成了唯一标识符，并且是您指定项目将使用哪个版本的外部库(例如 JARs)的机制。

### 3.2。依赖性

项目使用的这些外部库称为依赖项。Maven 中的依赖管理特性确保了从中央存储库中自动下载这些库，因此您不必将它们存储在本地。

这是 Maven 的一个关键特性，提供了以下好处:

*   **通过显著减少从远程存储库下载的次数，使用更少的存储空间**
*   **更快地签出项目**
*   **为在组织内外交换二进制工件提供了一个有效的平台，无需每次都从源代码构建工件**

为了声明对外部库的依赖，您需要提供库的`groupId, artifactId`和`version`。让我们来看一个例子:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.3.16</version>
</dependency>
```

当 Maven 处理依赖项时，它会将 Spring 核心库下载到您的本地 Maven 存储库中。

### 3.3。储存库

Maven 中的存储库用于保存不同类型的构建工件和依赖项。默认的本地存储库位于用户主目录下的`.m2/repository`文件夹中。

如果一个工件或插件在本地存储库中可用，Maven 就会使用它。否则，它将从中央存储库中下载并存储在本地存储库中。默认的中央存储库是 [Maven Central](https://web.archive.org/web/20220629003119/https://search.maven.org/classic/#search|ga|1|centra) 。

有些库，如 JBoss 服务器，在中央存储库中不可用，但在备用存储库中可用。对于这些库，您需要在`pom.xml`文件中提供备用存储库的 URL:

```java
<repositories>
    <repository>
        <id>JBoss repository</id>
        <url>http://repository.jboss.org/nexus/content/groups/public/</url>
    </repository>
</repositories>
```

请注意，您可以在您的项目中使用多个存储库。

### 3.4。属性

自定义属性有助于使您的`pom.xml`文件更容易阅读和维护。在经典用例中，您将使用自定义属性来定义项目依赖项的版本。

**Maven 属性是值占位符，可以通过使用符号`${name}`** 在`pom.xml`中的任何地方访问，其中`name`是属性。

让我们看一个例子:

```java
<properties>
    <spring.version>5.3.16</spring.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
</dependencies>
```

现在，如果您想将 Spring 升级到一个新的版本，您只需更改`<spring.version>` 属性标签中的值，所有在`<version>`标签中使用该属性的依赖项都将被更新。

属性也经常用于定义构建路径变量:

```java
<properties>
    <project.build.folder>${project.build.directory}/tmp/</project.build.folder>
</properties>

<plugin>
    //...
    <outputDirectory>${project.resources.build.folder}</outputDirectory>
    //...
</plugin>
```

### 3.5。构建

`build`部分也是 Maven `POM.`的一个非常重要的部分，它提供了关于默认 Maven `goal`、编译项目的目录以及应用程序的最终名称的信息。默认的`build`部分如下所示:

```java
<build>
    <defaultGoal>install</defaultGoal>
    <directory>${basedir}/target</directory>
    <finalName>${artifactId}-${version}</finalName>
    <filters>
      <filter>filters/filter1.properties</filter>
    </filters>
    //...
</build>
```

**编译后工件的默认输出文件夹名为`target`，打包后工件的最终名称由`artifactId`和`version`组成，但您可以随时更改。**

### 3.6。使用`Profiles`

Maven 的另一个重要特性是它对`profiles.`的支持，一个`profile`基本上是一组配置值。通过使用`profiles`，您可以为不同的环境定制构建，比如生产/测试/开发:

```java
<profiles>
    <profile>
        <id>production</id>
        <build>
            <plugins>
                <plugin>
                //...
                </plugin>
            </plugins>
        </build>
    </profile>
    <profile>
        <id>development</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <build>
            <plugins>
                <plugin>
                //...
                </plugin>
            </plugins>
        </build>
     </profile>
 </profiles>
```

正如您在上面的例子中所看到的，默认配置文件被设置为`development`。如果你想运行`production` `profile`，你可以使用下面的 Maven 命令:

```java
mvn clean install -Pproduction
```

## 4。Maven 构建生命周期

每个 Maven 构建都遵循一个指定的`lifecycle`。您可以执行几个构建`lifecycle` `goals`，包括那些到`compile`项目的代码，创建一个`package,`和`install`本地 Maven 依赖库中的归档文件。

### 4.1。`Lifecycle Phases`

以下列表显示了最重要的 Maven `lifecycle`阶段:

*   `validate`–检查项目的正确性
*   `compile`–将提供的源代码编译成二进制工件
*   `test`–执行单元测试
*   `package`–将编译后的代码打包成一个归档文件
*   `integration-test`–执行需要打包的附加测试
*   `verify`–检查包是否有效
*   `install`–将包文件安装到本地 Maven 存储库中
*   `deploy`–将程序包文件部署到远程服务器或存储库

### 4.2。*外挂*和*目标*

Maven `plugin`是一个或多个`goals`的集合。目标是分阶段执行的，这有助于确定执行`goals`的顺序。

Maven 官方支持的丰富插件列表可以在[这里](https://web.archive.org/web/20220629003119/https://maven.apache.org/plugins/)找到。**还有一篇有趣的文章是关于[如何使用各种插件在 Baeldung](/web/20220629003119/https://www.baeldung.com/executable-jar-with-maven) 上构建一个可执行文件`JAR`。**

为了更好地理解默认情况下哪个`goals`在哪个阶段运行，看一下[默认 Maven `lifecycle`绑定](https://web.archive.org/web/20220629003119/https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings)。

要经历上述任何一个阶段，我们只需调用一个命令:

```java
mvn <phase>
```

例如，`mvn clean install`将删除先前创建的 jar/war/zip 文件和编译的类(`clean)` ，并执行安装新归档文件(`install)`)所需的所有阶段。

**请注意，`plugins`提供的`goals`可以关联到`lifecycle`的不同相位。**

## 5。你的第一个 Maven 项目

在这一节中，我们将使用 Maven 的命令行功能来创建一个 Java 项目。

### 5.1。生成一个简单的 Java 项目

为了构建一个简单的 Java 项目，让我们运行以下命令:

```java
mvn archetype:generate \
  -DgroupId=com.baeldung \
  -DartifactId=baeldung \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DarchetypeVersion=1.4 \
  -DinteractiveMode=false
```

`groupId`是一个参数，表示创建项目的团体或个人，通常是一个反向的公司域名。`artifactId`是项目中使用的基础包名，我们使用标准的`archetype`。在这里，我们使用最新的原型版本来确保我们的项目是用更新和最新的结构创建的。

由于我们没有指定版本和打包类型，这些将被设置为默认值——默认情况下，版本将被设置为`1.0-SNAPSHOT,`,打包将被设置为`jar` 。

**如果你不知道要提供哪些参数，你总是可以指定`interactiveMode` = `true`，这样 Maven 就会要求所有需要的参数。**

命令完成后，我们在`src/main/java`文件夹中有一个包含`App.java`类的 Java 项目，这只是一个简单的“Hello World”程序。

我们在`src/test/java`中也有一个示例测试类。这个项目的`pom.xml`看起来类似于这个:

```java
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.baeldung</groupId>
    <artifactId>baeldung</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>baeldung</name>
    <url>http://www.example.com</url>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

如您所见，`JUnit`依赖项是默认提供的。

### 5.2。编译和打包项目

下一步是编译项目:

```java
mvn compile
```

Maven 将运行`compile`阶段构建项目源代码所需的所有`lifecycle`阶段。如果您只想运行`test`阶段，您可以使用:

```java
mvn test
```

现在让我们调用`package`阶段`,` ，它将产生编译后的归档`jar`文件:

```java
mvn package
```

### 5.3。执行应用程序

最后，我们将使用`exec-maven-plugin`执行我们的 Java 项目。让我们在`pom.xml`中配置必要的插件:

```java
<build>
    <sourceDirectory>src</sourceDirectory>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.6.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <version>3.0.0</version>
            <configuration>
                <mainClass>com.baeldung.java.App</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>
```

第一个插件， [`maven-compiler-plugin`](https://web.archive.org/web/20220629003119/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22maven-compiler-plugin%22) ，负责使用 Java 版本编译源代码。 [`exec-maven-plugin`](https://web.archive.org/web/20220629003119/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22exec-maven-plugin%22) 在我们的项目中搜索`mainClass`。

为了执行应用程序，我们运行以下命令:

```java
mvn exec:java
```

## 6。结论

在本文中，我们讨论了 Apache Maven 构建工具的一些更受欢迎的特性。

Baeldung 上的所有代码示例都是使用 Maven 构建的，所以你可以很容易地查看我们的 [GitHub 项目网站](https://web.archive.org/web/20220629003119/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-simple)来查看各种 Maven 配置。