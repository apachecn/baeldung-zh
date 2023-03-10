# Maven 编译器插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-compiler-plugin>

[This article is part of a series:](javascript:void(0);)[• Maven Resources Plugin](/web/20220626085525/https://www.baeldung.com/maven-resources-plugin)
• Maven Compiler Plugin (current article)[• Quick Guide to the Maven Install Plugin](/web/20220626085525/https://www.baeldung.com/maven-install-plugin)
[• The Maven Failsafe Plugin](/web/20220626085525/https://www.baeldung.com/maven-failsafe-plugin)
[• Quick Guide to the Maven Surefire Plugin](/web/20220626085525/https://www.baeldung.com/maven-surefire-plugin)
[• The Maven Deploy Plugin](/web/20220626085525/https://www.baeldung.com/maven-deploy-plugin)
[• The Maven Clean Plugin](/web/20220626085525/https://www.baeldung.com/maven-clean-plugin)
[• The Maven Verifier Plugin](/web/20220626085525/https://www.baeldung.com/maven-verifier-plugin)
[• The Maven Site Plugin](/web/20220626085525/https://www.baeldung.com/maven-site-plugin)
[• Guide to the Core Maven Plugins](/web/20220626085525/https://www.baeldung.com/core-maven-plugins)

## 1。概述

这个快速教程介绍了`compiler`插件，Maven 构建工具的核心插件之一。

关于其他核心插件的概述，请参考本文。

## 2。插件目标

**`compiler`插件用于编译一个 Maven 项目**的源代码。这个插件有两个目标，已经绑定到默认生命周期的特定阶段:

*   `compile`**–**编译主源文件
*   `testCompile`**–**编译测试源文件

这是 POM 中的`compiler`插件:

```java
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.7.0</version>
    <configuration>
        ...
    </configuration>
</plugin>
```

我们可以在这里找到这个插件的最新版本。

## 3。配置

**默认情况下，`compiler`插件编译与 Java 5 兼容的源代码，生成的类也能与 Java 5 一起工作，而不管使用的是什么 JDK。**我们可以在`configuration`元素中修改这些设置:

```java
<configuration>
    <source>1.8</source>
    <target>1.8</target>
    <-- other customizations -->
</configuration>
```

为了方便起见，我们可以将 Java 版本设置为 POM 的属性:

```java
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

有时我们想把参数传递给 [`javac`](/web/20220626085525/https://www.baeldung.com/javac) 编译器。这就是`compilerArgs`参数派上用场的地方。

例如，我们可以为编译器指定以下配置，以警告未检查的操作:

```java
<configuration>
    <!-- other configuration -->
    <compilerArgs>
        <arg>-Xlint:unchecked</arg>
    </compilerArgs>
</configuration>
```

编译该类时:

```java
public class Data {
    List<String> textList = new ArrayList();

    public void addText(String text) {
        textList.add(text);
    }

    public List getTextList() {
        return this.textList;
    }
}
```

我们将在控制台上看到一个未选中的警告:

```java
[WARNING] ... Data.java:[7,29] unchecked conversion
  required: java.util.List<java.lang.String>
  found:    java.util.ArrayList
```

由于`compiler`插件的两个目标都自动绑定到 Maven 默认生命周期中的阶段，我们可以用命令`mvn compile`和`mvn test-compile`来执行这些目标。

## 4.Java 9 更新

### 4.1.配置

直到 Java 8，我们都是用版本号为 1。`x`其中`x`代表 Java 的版本，比如 Java 8 的 1.8。

对于 Java 9 及以上版本，我们可以直接使用版本号:

```java
<configuration>
    <source>9</source>
    <target>9</target>
</configuration>
```

类似地，我们可以使用`properties`将版本定义为:

```java
<properties>
    <maven.compiler.source>9</maven.compiler.source>
    <maven.compiler.target>9</maven.compiler.target>
</properties>
```

Maven 在`3.5.0,` 中增加了对 Java 9 的支持，所以我们至少需要那个版本。我们至少还需要[中的`3.8.0` `maven-compiler-plugin`](https://web.archive.org/web/20220626085525/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-compiler-plugin%22):

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.0</version>
            <configuration>
                <source>9</source>
                <target>9</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 4.2.建设

现在是时候测试我们的配置了。

首先，让我们创建一个 `MavenCompilerPlugin` 类，在其中我们从另一个模块导入一个包[。](/web/20220626085525/https://www.baeldung.com/java-9-modularity)

简单的一个就是 `javax.xml.XMLConstants.XML_NS_PREFIX:`

```java
public class MavenCompilerPlugin {
    public static void main(String[] args) {
        System.out.println("The XML namespace prefix is: "
          + XML_NS_PREFIX);
    }
}
```

接下来，我们来编译一下:

```java
mvn -q clean compile exec:java
  -Dexec.mainClass="com.baeldung.maven.java9.MavenCompilerPlugin"
```

但是，当使用 Java 9 默认值时，我们会得到一个错误:

```java
[ERROR] COMPILATION ERROR :
[ERROR] .../MavenCompilerPlugin.java:[3,20]
  package javax.xml is not visible
  (package javax.xml is declared in module java.xml,
  but module com.baeldung.maven.java9 does not read it)
[ERROR] .../MavenCompilerPlugin.java:[3,1]
  static import only from classes and interfaces
[ERROR] .../MavenCompilerPlugin.java:[7,62]
  cannot find symbol
symbol:   variable XML_NS_PREFIX
location: class com.baeldung.maven.java9.MavenCompilerPlugin
```

错误来自于这个包在一个单独的模块中，我们还没有把它包含在我们的构建中。

解决这个问题最简单的方法是创建`a module-info.java`类，并指出我们需要`java.xml`模块:

```java
module com.baeldung.maven.java9 {
    requires java.xml;
}
```

现在我们可以再试一次:

```java
mvn -q clean compile exec:java
  -Dexec.mainClass="com.baeldung.maven.java9.MavenCompilerPlugin"
```

我们的输出将是:

```java
The XML namespace prefix is: xml
```

## 5。结论

在本文中，我们回顾了`compiler`插件并描述了如何使用它。我们还了解了 Maven 对 Java 9 的支持。

本教程的完整源代码可以在 GitHub 上的[找到。](https://web.archive.org/web/20220626085525/https://github.com/eugenp/tutorials/tree/master/maven-modules/compiler-plugin-java-9)

Next **»**[Quick Guide to the Maven Install Plugin](/web/20220626085525/https://www.baeldung.com/maven-install-plugin)**«** Previous[Maven Resources Plugin](/web/20220626085525/https://www.baeldung.com/maven-resources-plugin)