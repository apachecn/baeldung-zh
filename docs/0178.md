# 修复 Spring Boot 中的非主清单属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-fix-the-no-main-manifest-attribute>

## 1.概观

每当我们在 Spring Boot 的可执行 jar 中遇到`“no main manifest attribute”`消息，那是因为我们缺少来自文件`MANIFEST.MF`的 [`Main-Class`](/web/20221218224710/https://www.baeldung.com/spring-boot-main-class "spring-boot-main-class") 元数据属性的声明，该文件位于`META-INF`文件夹下。

在这个简短的教程中，我们来看看问题的原因以及如何解决它。

## 2.当问题出现时

一般来说，如果我们从 Spring Initializr 中取出我们的`pom`,我们不会有任何问题。然而，如果我们通过将 [`spring-boot-starter-parent`](https://web.archive.org/web/20221218224710/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-parent "spring-boot-starter-parent") 添加到`pom.xml`来手动构建我们的项目，我们可能会遇到这个问题。我们可以通过尝试构建一个干净的 jar 来复制它:

```
$ mvn clean package
```

我们在运行 jar 时会遇到错误:

```
$ java -jar target\spring-boot-artifacts-2.jar
```

```
no main manifest attribute, in target\spring-boot-artifacts-2.jar
```

在本例中，`MANIFEST.MF`文件的内容是:

```
Manifest-Version: 1.0
Archiver-Version: Plexus Archiver
Created-By: Apache Maven 3.6.3
Built-By: Baeldung
Build-Jdk: 11.0.13
```

## 3.使用 Maven 插件修复

### 3.1.添加插件

在这种情况下，最常见的问题是我们没有将`[spring-boot-maven-plugin](https://web.archive.org/web/20221218224710/https://search.maven.org/search?q=a:spring-boot-maven-plugin "spring-boot-maven-plugin")`声明添加到我们的`pom.xml`文件中。

让我们将`plugin` 定义添加到我们的`pom.xml`中，并在`plugins`标签下添加`Main-Class`声明:

```
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <mainClass>com.baeldung.demo.DemoApplication</mainClass>
            <layout>JAR</layout>
        </configuration>
    </plugin>
</plugins>
```

然而，**这可能不足以解决我们的问题**。在重新构建并运行我们的 jar 之后，我们可能仍然会收到`“no main manifest attribute”`消息。

让我们看看我们有什么额外的配置和替代方案来解决这个问题。

### 3.2.Maven 插件执行目标

让我们将`repackage`目标添加到紧接`configuration`标签之后的`spring-boot-maven-plugin`声明中:

```
<executions>
    <execution>
        <goals>
            <goal>repackage</goal>
        </goals>
    </execution>
</executions>
```

### 3.3.Maven 属性和内联命令执行目标

或者，**将属性`start-class`添加到我们的`pom.xml`文件的`properties`标签中，允许在构建过程中有更多的灵活性**:

```
<properties>
    <start-class>com.baeldung.demo.DemoApplication</start-class>
</properties>
```

现在，我们必须通过使用 Maven 内联命令 [`spring-boot:repackage`](/web/20221218224710/https://www.baeldung.com/spring-boot-repackage-vs-mvn-package "spring-boot-repackage-vs-mvn-package") 来构建 jar 执行目标:

```
$ mvn package spring-boot:repackage
```

## 4.检查`MANIFEST.MF`文件内容

让我们应用我们的解决方案，构建 jar，然后检查`MANIFEST.MF`文件。

我们注意到[的存在`Main-Class`和`Start-Class`的](/web/20221218224710/https://www.baeldung.com/spring-boot-main-class)属性:

```
Manifest-Version: 1.0
Archiver-Version: Plexus Archiver
Created-By: Apache Maven 3.6.3
Built-By: Baeldung
Build-Jdk: 11.0.13
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.baeldung.demo.DemoApplication
Spring-Boot-Version: 2.7.5
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
Spring-Boot-Layers-Index: BOOT-INF/layers.idx
```

现在执行 jar，`“no main manifest attribute”`消息问题不再出现，应用程序运行。

## 5.结论

在本文中，我们看到了在执行 Spring Boot 可执行 jar 时如何解决`“no main manifest attribute”`消息。

我们看到了这是如何来自手动创建的`pom.xml`文件，以及如何添加和配置 Spring Maven 插件来修复它。

和往常一样，示例代码[可以在 GitHub](https://web.archive.org/web/20221218224710/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-artifacts-2) 上获得。