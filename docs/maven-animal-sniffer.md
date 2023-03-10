# 动物嗅探器 Maven 插件简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-animal-sniffer>

## 1。简介

使用 Java 时，有时我们需要同时使用多种语言版本。

通常需要我们的 Java 程序在编译时与一个 Java 版本(比如说 Java 6)兼容，但是需要在我们的开发工具中使用一个不同的版本(比如说 Java 8)和一个可能不同的版本来运行应用程序。

在这篇简短的文章中，我们将展示添加基于 Java 版本的不兼容保护措施是多么容易，以及如何使用 Animal Sniffer 插件，通过对照先前生成的签名检查我们的项目，在构建时标记这些问题。

## 2。设置 Java 编译器的`-source`和`-target`

让我们从一个`hello world` Maven 项目开始——我们在本地机器上使用 Java 7，但是我们希望将该项目部署到仍然使用 Java 6 的生产环境中。

在这种情况下，我们可以用指向 Java 6 的`source`和`target`字段来配置 Maven 编译器插件。

`“source”`字段用于指定与 Java 语言变化的兼容性，而`“target”`字段用于指定与 JVM 变化的兼容性。

现在让我们来看看`pom.xml:`的 Maven 编译器配置

```java
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.7.0</version>
	<configuration>
            <source>1.6</source>
            <target>1.6</target>
	</configuration>
    </plugin>
</plugins>
```

有了本地机器上的 Java 7 和控制台上的 Java 代码输出“hello world ”,如果我们继续使用 Maven 构建这个项目，它将在运行 Java 6 的生产机器上正确地构建和工作。

## 3。API 不兼容性介绍

现在让我们看看偶然引入 API 不兼容是多么容易。

假设我们开始处理一些新的需求，我们使用了 Java 7 的一些 API 特性，这些特性在 Java 6 中是没有的。

让我们看看更新后的源代码:

```java
public static void main(String[] args) {
    System.out.println("Hello World!");
    System.out.println(StandardCharsets.UTF_8.name());
}
```

`java.nio.charset.StandardCharsets`在 Java 7 中推出。

如果我们现在继续执行 Maven 构建，它仍然可以成功编译，但是在运行时会失败，并在安装了 Java 6 的生产机器上出现链接错误。

专家文档提到了这个缺陷，并推荐使用动物嗅探器插件作为选项之一。

## 4。报告 API 兼容性

动物嗅探器插件提供了两个核心功能:

1.  生成 Java 运行时的签名
2.  对照 API 签名检查项目

现在让我们修改`pom.xml`来包含插件:

```java
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>animal-sniffer-maven-plugin</artifactId>
    <version>1.16</version>
    <configuration>
        <signature>
            <groupId>org.codehaus.mojo.signature</groupId>
            <artifactId>java16</artifactId>
            <version>1.0</version>
        </signature>
    </configuration>
    <executions>
        <execution>
            <id>animal-sniffer</id>
            <phase>verify</phase>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

这里，Animal Sniffer 的配置部分指的是现有的 Java 6 运行时签名。此外，如果发现任何问题，执行部分将根据给定的签名和标志检查和验证项目源代码。

如果我们继续构建 Maven 项目，构建将会失败，插件会像预期的那样报告签名验证错误:

```java
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.codehaus.mojo:animal-sniffer-maven-plugin:1.16:check 
(animal-sniffer) on project example-animal-sniffer-mvn-plugin: Signature errors found.
Verify them and ignore them with the proper annotation if needed.
```

## 5。结论

在本教程中，我们探索了 Maven Animal Sniffer 插件，以及如何在构建时使用它来报告 API 相关的不兼容性。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221105201417/https://github.com/eugenp/tutorials/tree/master/maven-modules/animal-sniffer-mvn-plugin)