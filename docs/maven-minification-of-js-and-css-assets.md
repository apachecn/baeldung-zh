# 用 Maven 缩小 JS 和 CSS 资产

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-minification-of-js-and-css-assets>

## 1。概述

本文展示了如何在构建步骤中缩小 Javascript 和 CSS 资产，并使用 Spring MVC 提供结果文件。

我们将使用 [YUI 压缩器](https://web.archive.org/web/20211127201804/https://yui.github.io/yuicompressor/)作为底层缩小库，使用 [YUI 压缩器 Maven 插件](https://web.archive.org/web/20211127201804/https://davidb.github.io/yuicompressor-maven-plugin/)将其集成到我们的构建过程中。

## 2。Maven 插件配置

首先，我们需要声明我们将在我们的`pom.xml`文件中使用 compressor 插件并执行`compress` 目标。这将压缩`src/main/webapp` 下的所有`.js`和`.css`文件，这样`foo.js`将被缩小为`foo-min.js`,`myCss.css`将被缩小为`myCss-min.css`:

```java
<plugin>
   <groupId>net.alchim31.maven</groupId>
    <artifactId>yuicompressor-maven-plugin</artifactId>
    <version>1.5.1</version>
    <executions>
        <execution>
            <goals>
                <goal>compress</goal>
            </goals>
        </execution>
    </executions>
</plugin> 
```

我们的`src/main/webapp directory`包含以下文件:

```java
js/
├── foo.js
├── jquery-1.11.1.min.js
resources/
└── myCss.css 
```

执行`mvn clean package`后，生成的 WAR 文件将包含以下文件:

```java
js/
├── foo.js
├── foo-min.js
├── jquery-1.11.1.min.js
├── jquery-1.11.1.min-min.js
resources/
├── myCss.css
└── myCss-min.css
```

## 3。保持文件名不变

在这个阶段，当我们执行`mvn clean package`，`foo-min.js`和`myCss-min.css`是由插件创建的。由于我们最初在引用文件时使用了`foo.js`和`myCss.css`，我们的页面将仍然使用原始的未缩小文件，因为缩小文件的名称与原始文件不同。

为了防止同时拥有`foo.js/foo-min.js`和`myCss.css/myCss-min.css`并且在不改变文件名的情况下缩小文件，我们需要配置带有`nosuffix`选项的插件，如下所示:

```java
<plugin>
    <groupId>net.alchim31.maven</groupId>
    <artifactId>yuicompressor-maven-plugin</artifactId>
    <version>1.5.1</version>
    <executions>
        <execution>
            <goals>
                <goal>compress</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <nosuffix>true</nosuffix>
    </configuration>
</plugin>
```

现在，当我们执行`mvn clean package`时，我们将在生成的 WAR 文件中拥有以下文件:

```java
js/
├── foo.js
├── jquery-1.11.1.min.js
resources/
└── myCss.css
```

## 4。WAR 插件配置

保持文件名不变有一个副作用。它导致 WAR 插件用原始文件覆盖缩小的`foo.js`和`myCss.css`文件，所以我们在最终输出中没有文件的缩小版本。`foo.js`文件在缩小前包含以下几行:

```java
function testing() {
    alert("Testing");
}
```

当我们检查生成的 WAR 文件中的`foo.js`文件的内容时，我们看到它具有原始内容，而不是缩小的内容。为了解决这个问题，我们需要为 compressor 插件指定一个`webappDirectory` ,并在 WAR 插件配置中引用它。

```java
<plugin>
    <groupId>net.alchim31.maven</groupId>
    <artifactId>yuicompressor-maven-plugin</artifactId>
    <version>1.5.1</version>
    <executions>
        <execution>
            <goals>
                <goal>compress</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <nosuffix>true</nosuffix>
        <webappDirectory>${project.build.directory}/min</webappDirectory>
    </configuration>
</plugin>
<plugin>
<artifactId>maven-war-plugin</artifactId>
<configuration>
    <webResources>
        <resource>
            <directory>${project.build.directory}/min</directory>
        </resource>
    </webResources>
</configuration>
</plugin>
```

在这里，我们指定了`min`目录作为缩小文件的输出目录，并配置了 WAR 插件，使其包含在最终输出中。

现在我们在生成的 WAR 文件中有了缩小的文件，它们的原始文件名是`foo.js`和`myCss.css.` ,我们可以检查`foo.js`,看看它现在有以下缩小的内容:

```java
function testing(){alert("Testing")};
```

## 5。排除已经缩小的文件

第三方 Javascript 和 CSS 库可能有缩小版可供下载。如果您碰巧在您的项目中使用了其中的一个，您不需要再次处理它们。

包含已经缩小的文件会在生成项目时产生警告消息。

例如，`jquery-1.11.1.min.js`是一个已经缩小的 Javascript 文件，它会在构建过程中产生类似如下的警告消息:

```java
[WARNING] .../src/main/webapp/js/jquery-1.11.1.min.js [-1:-1]: 
Using 'eval' is not recommended. Moreover, using 'eval' reduces the level of compression!
execScript||function(b){a. ---> eval <--- .call(a,b);})
[WARNING] ...jquery-1.11.1.min.js:line -1:column -1: 
Using 'eval' is not recommended. Moreover, using 'eval' reduces the level of compression!
execScript||function(b){a. ---> eval <--- .call(a,b);})
```

要从进程中排除已经缩小的文件，请使用`excludes` 选项配置 compressor 插件，如下所示:

```java
<plugin>
    <groupId>net.alchim31.maven</groupId>
    <artifactId>yuicompressor-maven-plugin</artifactId>
    <version>1.5.1</version>
    <executions>
        <execution>
            <goals>
                <goal>compress</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <nosuffix>true</nosuffix>
        <webappDirectory>${project.build.directory}/min</webappDirectory>
        <excludes>
            <exclude>**/*.min.js</exclude>
        </excludes>
    </configuration>
</plugin>
```

这将排除文件名以`min.js`结尾的所有目录下的所有文件。执行`mvn clean package` 现在不会产生警告消息，构建也不会试图缩小已经缩小的文件。

## 6。结论

在本文中，我们描述了一种将 Javascript 和 CSS 文件的精简集成到 Maven 工作流中的好方法。要用 Spring MVC 应用程序服务这些静态资产，请参阅我们的文章[用 Spring](/web/20211127201804/https://www.baeldung.com/spring-mvc-static-resources) 服务静态资源。

你可以在 [GitHub](https://web.archive.org/web/20211127201804/https://github.com/eugenp/tutorials/tree/master/spring-static-resources) 上找到这篇文章的代码。