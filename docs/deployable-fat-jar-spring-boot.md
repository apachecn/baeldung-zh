# 用 Spring Boot 创建一个胖罐子应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/deployable-fat-jar-spring-boot>

## 1。简介

近年来最令人振奋的发展之一是 web 应用程序部署方式的不断简化。

跳过所有无聊的中间历史步骤，我们到达了今天——我们不仅可以省去繁琐的 servlets 和 XML 样板文件，而且主要是服务器本身。

本文将专注于**从 Spring Boot 应用**中创建一个`fat jar”` ——基本上是创建一个易于部署和运行的工件。 

Boot 提供了开箱即用的无容器部署功能:我们需要做的只是在`pom.xml:`中添加一些配置

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.4.0</version>
    </dependency>
</dependencies>

<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>2.4.0</version>
    </plugin>
</plugins>
```

## 2。构建并运行

有了这个配置，我们现在可以简单地用标准的`mvn clean install`构建项目——这没什么不寻常的。

我们用下面的命令运行它:`java -jar <artifact-name>`–非常简单和直观。

正确的进程管理超出了本文的范围，但是一个简单的保持进程运行的方法是使用`nohup` 命令:`nohup java -jar <artifact-name>.`

停止`spring-boot` 项目也与停止常规过程没有什么不同，无论我们只是简单地`cntrl+c` 或`kill <pid>.`

## 3。胖罐子/胖战争

在幕后，`spring-boot`将所有的项目依赖项打包到最终工件中，并附带项目类(因此有了“胖”罐子)。嵌入式 Tomcat 服务器也是内置的。

因此，最终的工件是完全独立的，易于使用标准的 Unix 工具(scp、sftp 等)进行部署，并且可以在任何具有 JVM 的服务器上运行。

默认情况下，Boot 会创建一个`jar` 文件——但是如果我们将`pom.xml` 中的`packaging` 属性改为`war`，Maven 会自然地转而**构建一个 war** 。

这当然既可以独立执行，也可以部署到 web 容器中。

## 4。进一步配置

大多数时候不需要额外的配置，一切都“正常工作”，但是在一些特定的情况下，我们可能需要明确地告诉`spring-boot` 主类是什么。一种方法是添加一个属性:

```
<properties>
    <start-class>org.baeldung.boot.Application</start-class>
</properties>
```

如果我们继承了 spring-boot-starter-parent，我们需要在 Maven 插件中这样做:

```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>2.4.0</version>
    <configuration>
        <mainClass>org.baeldung.boot.Application</mainClass>
        <layout>ZIP</layout>
    </configuration>
</plugin>
```

在一些罕见的情况下，我们可能需要做的另一件事是指示 Maven`unpack` 一些依赖关系:

```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <requiresUnpack>
            <dependency>
                <groupId>org.jruby</groupId>
                <artifactId>jruby-complete</artifactId>
            </dependency>
        </requiresUnpack>
    </configuration>
</plugin>
```

## 5。结论

在本文中，我们研究了使用由`spring-boot.`构建的“胖”jar 的无服务器部署

和往常一样，这篇文章中的代码都可以在 Github 的[上找到。](https://web.archive.org/web/20220813073109/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-artifacts)