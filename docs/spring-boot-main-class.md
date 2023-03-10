# Spring Boot:配置主类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-main-class>

## 1.概观

这篇快速教程提供了通过 Maven 和 Gradle 定义 Spring Boot 应用程序入口点的不同方法。

Spring Boot 应用程序的主类是一个包含启动 Spring `ApplicationContext`的`public static void main()`方法的类。**默认情况下，如果没有显式指定主类，Spring 将在编译时在类路径中搜索一个，如果没有找到或找到多个，将无法启动。**

与传统的 Java 应用程序不同，本教程中讨论的主类在 META-INF/MANIFEST 中并不显示为`Main-Class`元数据属性。结果 JAR 或 WAR 文件的 MF。

Spring Boot 希望工件的`Main-Class`元数据属性被设置为`org.springframework.boot.loader.JarLauncher` (或`WarLauncher` ) ，这意味着将我们的主类直接传递到 java 命令行不会正确启动我们的 Spring Boot 应用程序。

清单示例如下所示:

```java
Manifest-Version: 1.0
Start-Class: com.baeldung.DemoApplication
Main-Class: org.springframework.boot.loader.JarLauncher
```

相反，我们需要在清单中定义由`JarLauncher`评估的`Start-Class`属性来启动应用程序。

让我们看看如何使用 Maven 和 Gradle 来控制这个属性。

## 2.专家

主类可以被定义为`pom.xml`的属性部分中的`a start-class`元素:

```java
<properties>
      <!-- The main class to start by executing "java -jar" -->
      <start-class>com.baeldung.DemoApplication</start-class>
</properties> 
```

请注意，**只有在我们将[spring-boot-starter-parent](https://web.archive.org/web/20221026041503/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-maven-plugin%22)作为`<parent>`添加到我们的`pom.xml`中时，才会对该属性进行评估。**

或者，**主类可以被定义为我们的`pom.xml`的插件部分中的`spring-boot-maven-plugin`** 的`mainClass`元素:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>             
            <configuration>    
                <mainClass>com.baeldung.DemoApplication</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>
```

这个 Maven 配置的例子可以在 GitHub 上找到。

## 3\. Gradle

如果我们使用的是 **Spring Boot Gradle 插件**，有一些从`org.springframework.boot `继承的配置，我们可以在那里指定我们的主类。

在项目的 Gradle 文件中，`mainClassName`可以在`springBoot`配置块中定义**。这里做的这个改动被`bootRun` 和 `bootJar`任务拾取:**

```java
springBoot {
    mainClassName = 'cpm.baeldung.DemoApplication'
}
```

**或者，主类可以定义为`bootJar` Gradle task:** 的`mainClassName`属性

```java
bootJar {
    mainClassName = 'cpm.baeldung.DemoApplication'
}
```

**或者作为`bootJar`任务的一个显式属性:**

```java
bootJar {
    manifest {
	attributes 'Start-Class': 'com.baeldung.DemoApplication'
    }
}
```

注意在`bootJar`配置块中指定的主类只影响任务本身产生的 JAR。这一改变不会影响其他 Spring Boot 等级任务的行为，比如`bootRun`。

另外，如果将 **Gradle 应用插件**应用到项目中， **`mainClassName`可以被定义为一个全局属性:**

```java
mainClassName = 'com.baeldung.DemoApplication' 
```

我们可以在 GitHub 上找到这些梯度配置的例子。

## 4.使用 CLI

我们也可以通过命令行界面指定一个主类。

Spring Boot 的`org.springframework.boot.loader.PropertiesLauncher`带有一个 JVM 参数，让你覆盖名为`loader.main`的逻辑主类:

```java
java -cp bootApp.jar -Dloader.main=com.baeldung.DemoApplication org.springframework.boot.loader.PropertiesLauncher
```

## 5.结论

有很多方法可以指定 Spring Boot 应用程序的入口点。重要的是要知道所有这些配置只是修改 JAR 或 WAR 文件清单的不同方式。

工作代码示例可以在[这里](https://web.archive.org/web/20221026041503/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization)和[这里](https://web.archive.org/web/20221026041503/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-gradle)找到。