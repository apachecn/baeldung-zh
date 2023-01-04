# 装有 Spring Boot 的薄广口瓶

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-thin-jar>

## 1。简介

在本教程中，我们将看看**如何使用 [`spring-boot-thin-launcher`](https://web.archive.org/web/20221225070758/https://github.com/dsyer/spring-boot-thin-launcher) 项目将一个 Spring Boot 项目构建成一个瘦 JAR 文件。**

Spring Boot 以其“胖”JAR 部署而闻名，其中一个可执行工件包含应用程序代码及其所有依赖项。

Boot 也被广泛用于开发微服务。这有时会与“fat JAR”方法相冲突，因为在许多工件中反复包含相同的依赖关系会成为一种重要的资源浪费。

## 2。先决条件

首先，我们当然需要一个 Spring Boot 项目。在本文中，我们将研究 Maven 构建和 Gradle 构建的最常见配置。

不可能涵盖所有的构建系统和构建配置，但是，希望我们会看到足够多的一般原则，您应该能够将它们应用到您的特定设置中。

### 2.1。Maven 项目

在用 Maven 构建的引导项目中，我们应该在项目的`pom.xml` 文件、其父文件或其祖先文件中配置 Spring Boot Maven 插件:

```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>    
</plugin>
```

Spring Boot 依赖项的版本通常通过使用 BOM 或从父 POM 继承来决定，如我们的参考项目:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
    <relativePath/>
</parent>
```

### 2.2。Gradle 项目

在用 Gradle 构建的引导项目中，我们将拥有引导 Gradle 插件:

```java
buildscript {
    ext {
        springBootPlugin = 'org.springframework.boot:spring-boot-gradle-plugin'
        springBootVersion = '2.4.0'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("${springBootPlugin}:${springBootVersion}")
    }
}

// elided

apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

springBoot {
    mainClassName = 'com.baeldung.DemoApplication'
}
```

注意，在本文中，我们将只考虑 Boot 2.x 和更高版本的项目。瘦启动程序也支持早期版本，但是它需要一个稍微不同的 Gradle 配置，为了简单起见我们省略了这个配置。请查看该项目的主页了解更多详情。

## 3。如何打造一个薄罐子？

Spring Boot 瘦启动程序是一个小的库，它从打包在存档中的文件中读取工件的依赖项，从 Maven 存储库中下载它们，最后启动应用程序的主类。

因此，**当我们用这个库构建一个项目时，我们会得到一个包含我们代码的 JAR 文件，一个枚举其依赖项的文件，以及执行上述任务的库中的主类。**

当然，事情比我们简化的解释更微妙一些；我们将在本文后面深入讨论一些主题。

## 4。基本用法

现在让我们看看如何从常规的 Spring Boot 应用程序构建一个“瘦”JAR。

我们将使用常规的`java -jar <my-app-1.0.jar>,` 启动应用程序，并使用可选的附加命令行参数来控制瘦启动器。我们将在接下来的章节中看到其中的一些；该项目的主页上有完整的列表。

### 4.1。Maven 项目

在一个 Maven 项目中，我们必须修改引导插件的声明(参见 2.1 节)以包含对定制“瘦”布局的依赖:

```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <dependencies>
        <!-- The following enables the "thin jar" deployment option. -->
        <dependency>
            <groupId>org.springframework.boot.experimental</groupId>
            <artifactId>spring-boot-thin-layout</artifactId>
            <version>1.0.11.RELEASE</version>
        </dependency>
    </dependencies>
</plugin>
```

[启动器](https://web.archive.org/web/20221225070758/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-thin-layout%22)将从`pom.xml` 文件中读取依赖项，Maven 将该文件存储在`META-INF/maven` 目录下生成的 JAR 中。

**我们将照常执行构建，例如使用`mvn install`。**

如果我们希望能够产生瘦构建和胖构建(例如在一个有多个模块的项目中)，我们可以在一个专用的 Maven 概要文件中声明定制布局。

### 4.2。`thin.properties`美芬与依赖:

**除了`pom.xml`之外，我们还可以让 Maven 生成一个`thin.properties` 文件。**在这种情况下，文件将包含依赖项的完整列表，包括可传递的依赖项，启动程序将更喜欢它而不是`pom.xml`。

这样做的 mojo(插件)是`spring-boot-thin-maven-plugin:properties,` ，默认情况下，它在`src/main/resources/META-INF`中输出`thin.properties` 文件，但是我们可以用`thin.output` 属性指定它的位置:

```java
$ mvn org.springframework.boot.experimental:spring-boot-thin-maven-plugin:properties -Dthin.output=.
```

请注意，输出目录必须存在，目标才能成功，即使我们保留了默认目录。

### 4.3。Gradle 项目

相反，在 Gradle 项目中，我们添加了一个专用插件:

```java
buildscript {
    ext {
        //...
        thinPlugin = 'org.springframework.boot.experimental:spring-boot-thin-gradle-plugin'
        thinVersion = '1.0.11.RELEASE'
    }
    //...
    dependencies {
        //...
        classpath("${thinPlugin}:${thinVersion}")
    }
}

//elided

apply plugin: 'maven'
apply plugin: 'org.springframework.boot.experimental.thin-launcher'
```

为了获得瘦构建，我们将告诉 Gradle 执行`thinJar` 任务:

```java
~/projects/baeldung/spring-boot-gradle $ ./gradlew thinJar
```

### 4.4。`pom.xml`

在前一节的代码示例中，除了 Thin Launcher 之外，我们还声明了 Maven 插件(以及我们在先决条件一节中已经看到的引导和依赖管理插件)。

这是因为，就像我们之前看到的 Maven 案例一样，工件将包含并利用一个`pom.xml` 文件来枚举应用程序的依赖项。`pom.xml` 文件由一个名为`thinPom`的任务生成，它是任何 jar 任务的隐式依赖项。

**我们可以用专门的任务定制生成的`pom.xml` 文件。**在这里，我们将自动复制瘦插件已经做的事情:

```java
task createPom {
    def basePath = 'build/resources/main/META-INF/maven'
    doLast {
        pom {
            withXml(dependencyManagement.pomConfigurer)
        }.writeTo("${basePath}/${project.group}/${project.name}/pom.xml")
    }
}
```

为了使用我们的定制`pom.xml`文件，我们将上述任务添加到 jar 任务的依赖项中:

```java
bootJar.dependsOn = [createPom]
```

### 4.5。`thin.properties`

**我们也可以让 Gradle 生成一个`thin.properties` 文件，而不是像我们之前对 Maven 所做的那样生成`pom.xml`、**。

生成`thin.properties` 文件的任务称为`thinProperties,` ，默认情况下不使用它。我们可以将其添加为 jar 任务的依赖项:

```java
bootJar.dependsOn = [thinProperties]
```

## 5。存储依赖关系

瘦 jar 的全部意义在于避免将依赖项与应用程序捆绑在一起。然而，依赖关系不会神奇地消失，它们只是被存储在其他地方。

特别是，瘦启动程序使用 Maven 基础设施来解决依赖性，因此:

1.  它检查本地 Maven 存储库，默认情况下该存储库位于`~/.m2/repository` 中，但是可以移动到其他地方；
2.  然后，它从 Maven Central(或任何其他已配置的存储库)下载缺失的依赖项；
3.  最后，它将它们缓存在本地存储库中，这样我们下次运行应用程序时就不必再下载它们了。

当然，**下载阶段是该过程中缓慢且容易出错的部分，**因为它需要通过互联网访问 Maven Central，或者访问本地代理，我们都知道这些东西通常是不可靠的。

幸运的是，有各种方法可以将依赖项与应用程序一起部署，例如，在预打包的容器中进行云部署。

### 5.1。运行应用程序进行预热

缓存依赖项的最简单方法是在目标环境中对应用程序进行预热运行。正如我们前面看到的，这将导致依赖项被下载并缓存在本地 Maven 存储库中。如果我们运行不止一个应用程序，那么存储库将包含所有的依赖项，而不会重复。

由于运行一个应用程序可能会有不必要的副作用，**我们也可以执行一个“模拟运行”，只解析和下载依赖项，而不运行任何用户代码:**

```java
$ java -Dthin.dryrun=true -jar my-app-1.0.jar
```

注意，按照 Spring Boot 惯例，我们也可以用应用程序的`–thin.dryrun` 命令行参数或者用`THIN_DRYRUN` 系统属性来设置`-Dthin.dryrun` 属性。除了`false` 之外的任何值都将指示瘦启动器执行一次试运行。

### 5.2。在构建过程中打包依赖关系

另一个选择是在构建过程中收集依赖项，而不是将它们捆绑在 JAR 中。然后，作为部署过程的一部分，我们可以将它们复制到目标环境中。

这通常更简单，因为不需要在目标环境中运行应用程序。然而，如果我们正在部署多个应用程序，我们将不得不合并它们的依赖关系，无论是手动还是使用脚本。

用于 Maven 和 Gradle 的瘦插件在构建期间打包依赖项的格式与 Maven 本地存储库相同:

```java
root/
    repository/
        com/
        net/
        org/
        ...
```

事实上，我们可以在运行时用`thin.root` 属性将使用瘦启动器的应用程序指向任何这样的目录(包括本地 Maven 存储库):

```java
$ java -jar my-app-1.0.jar --thin.root=my-app/deps
```

我们还可以通过一个接一个地复制来安全地合并多个这样的目录，从而获得一个包含所有必需依赖项的 Maven 存储库。

### 5.3。用 Maven 打包依赖关系

为了让 Maven 为我们打包依赖项，我们使用了`spring-boot-thin-maven-plugin.` 的`resolve` 目标，我们可以在我们的`pom.xml:`中手动或自动调用它

```java
<plugin>
    <groupId>org.springframework.boot.experimental</groupId>
    <artifactId>spring-boot-thin-maven-plugin</artifactId>
    <version>${thin.version}</version>
    <executions>
        <execution>
        <!-- Download the dependencies at build time -->
        <id>resolve</id>
        <goals>
            <goal>resolve</goal>
        </goals>
        <inherited>false</inherited>
        </execution>
    </executions>
</plugin>
```

在构建项目之后，我们将找到一个目录`target/thin/root/` ,它具有我们在上一节中讨论过的结构。

### 5.4。用 Gradle 打包依赖关系

如果我们使用 Gradle 和`thin-launcher`插件，我们有一个`thinResolve` 任务可用。该任务将应用程序及其依赖项保存在`build/thin/root/` 目录中，类似于上一节的 Maven 插件:

```java
$ gradlew thinResolve
```

## 6。结论和进一步阅读

在这篇文章中，我们已经看到了如何使我们的薄罐子。我们还看到了如何使用 Maven 基础设施来下载和存储它们的依赖项。

thin launcher 的[主页](https://web.archive.org/web/20221225070758/https://github.com/dsyer/spring-boot-thin-launcher)提供了更多的操作指南，例如云部署到 Heroku，以及支持的命令行参数的完整列表。

所有 Maven 示例和代码片段的实现可以在 GitHub 项目中找到——作为一个 Maven 项目，所以它应该很容易导入和运行。

类似地，所有 Gradle 的例子都引用了这个 GitHub 项目。