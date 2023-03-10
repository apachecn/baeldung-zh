# java 文档链接

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-doclint>

## 1.概观

使用 [Javadoc](/web/20220628122833/https://www.baeldung.com/javadoc) 是一个好主意的原因有很多。例如，我们可以从 Java 代码中生成 HTML，遍历它们的定义，并发现与它们相关的各种属性。

此外，**方便了开发者之间的交流，** **提高了可维护性**。Java [DocLint](https://web.archive.org/web/20220628122833/https://openjdk.java.net/jeps/172) 是一个分析我们 Javadoc 的工具。每当发现错误的语法时，它就会抛出警告和错误。

在本教程中，我们将重点讨论如何使用它。稍后，我们将研究它在某些情况下可能产生的问题，以及如何避免这些问题的一些指导原则。

## 2.如何使用 DocLint

假设我们有一个名为`Sample.java`的类文件:

```java
/**
 * This sample file creates a class that
 * just displays sample string on standard output.
 *
 * @autho  Baeldung
 * @version 0.9
 * @since   2020-06-13 
 */
public class Sample {

    public static void main(String[] args) {
        // Prints Sample! on standard output.
        System.out.println("Sample!");
    }
}
```

这里故意打错了，`@author`参数写成了`@autho`。让我们看看，如果我们尝试在没有 DocLint `:`的情况下制作 Javadoc 会发生什么

[![](img/7f79b6d2e53cc84963fa88c5669816ea.png)](/web/20220628122833/https://www.baeldung.com/wp-content/uploads/2021/06/jdoc.png)

正如我们从控制台输出中看到的，Javadoc 引擎无法找出我们文档中的错误，也没有返回任何错误或警告。

为了让 Java DocLint 返回这种类型的警告，我们必须执行带有–`Xdoclint` 选项的`javadoc`命令。(我们稍后会详细解释这一点):

[![](img/422a13a98a50211c60fab584f06ec1f3.png)](/web/20220628122833/https://www.baeldung.com/wp-content/uploads/2021/06/jdoc-error.png)

正如我们在输出中看到的，它直接提到了 Java 文件第 5 行中的错误:

```java
Sample.java:5: error: unknown tag: autho
 * @autho  Baeldung
   ^
```

## 3.`-Xdoclint`

`-Xdoclint`参数有三个不同用途的选项。我们将快速浏览每一个。

### 3.1.`none`

`none`选项禁用`-Xdoclint`选项:

```java
javadoc -Xdoclint:none Sample.java
```

### 3.2.`group`

当我们想要应用与某些组相关的某些检查时，此选项很有用，例如:

```java
javadoc -Xdoclint:syntax Sample.java
```

有几种类型的组变量:

*   `accessibility`–检查可访问性检查器要检测的问题(例如，在`table`标签中没有指定标题或摘要属性)
*   `html`–检测高级 HTML 问题，比如将块元素放入行内元素中，或者不关闭需要结束标记的元素
*   `missing`–检查缺少的 Javadoc 注释或标记(例如，缺少的注释或类，或者缺少的`@return`标记或方法上的类似标记)
*   `reference`–检查与从 Javadoc 标签引用 Java API 元素相关的问题(例如，在`@see`中找不到项目，或者在`@param`后有错误的名称)
*   `syntax`–检查低级问题，如未转义的尖括号(<和>)、&和无效的 Javadoc 标签

可以一次应用多个组:

```java
javadoc -Xdoclint:html,syntax,accessibility Sample.java
```

### 3.3.`all`

这个选项一次启用所有组，但是如果我们想排除其中一些组呢？

我们可以使用`-group`语法:

```java
javadoc -Xdoclint:all,-missing
```

## 4.如何禁用 DocLint

由于 Java DocLint **在 Java 8** 之前并不存在，这可能会造成不必要的麻烦，尤其是在旧的第三方库中。

我们已经在前面的章节中看到了带有`javadoc`命令的`none`选项。此外，还有一个选项是**从 Maven、Gradle、Ant 等构建系统**中禁用 DocLint。我们将在接下来的几个小节中看到这些。

### 4.1 版。肚子

对于`maven-javadoc-plugin`，从版本 3.0.0 开始，添加了新的 [doclint](https://web.archive.org/web/20220628122833/https://maven.apache.org/plugins/maven-javadoc-plugin/javadoc-mojo.html#doclint) 配置。让我们看看如何配置它来禁用 DocLint:

```java
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-javadoc-plugin</artifactId>
        <version>3.1.1</version>
        <configuration>
            <doclint>none</doclint> <!-- Turn off all checks -->
        </configuration>
        <executions>
            <execution>
                <id>attach-javadocs</id>
                <goals>
                    <goal>jar</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
</plugins>
```

但是**一般来说，不推荐使用`none`选项**，因为它会跳过所有类型的检查。我们应该用`<doclint>all,-missing</doclint>`来代替。

使用早期版本(v3.0.0 之前)时，我们需要使用不同的设置:

```java
<plugins>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <configuration>
      <additionalparam>-Xdoclint:none</additionalparam>
    </configuration>
  </plugin>
</plugins>
```

### 4.2 版。度

我们可以用一个简单的脚本在 Gradle 项目中停用 DocLint:

```java
if (JavaVersion.current().isJava8Compatible()) {
    allprojects {
        tasks.withType(Javadoc) {
            options.addStringOption('Xdoclint:none', '-quiet')
        }
    }
}
```

不幸的是，Gradle 不像上面例子中的 Maven 那样支持`additionalparam`，所以我们需要手动完成。

### 4.3。蚂蚁

Ant 像 Maven 一样使用`additionalparam`，所以我们可以像上面演示的那样设置`-Xdoclint:none`。

## 5.结论

在本文中，我们研究了使用 Java DocLint 的各种方法。当我们想要一个干净的、容易出错的 Javadoc 时，它可以帮助我们。

要获得更多的深入信息，最好看一下官方的[文档](https://web.archive.org/web/20220628122833/https://docs.oracle.com/en/java/javase/11/tools/javadoc.html)。