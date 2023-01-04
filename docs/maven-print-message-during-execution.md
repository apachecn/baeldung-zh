# 如何在 Maven 中显示消息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-print-message-during-execution>

## 1.概观

有时候，我们可能想在 Maven 执行期间打印一些额外的信息。然而，在 Maven 构建生命周期中，没有内置的方法将值输出到控制台。

在本教程中，我们将探索在 Maven 执行期间**支持打印消息的插件。我们将讨论三个不同的插件，每个插件都可以绑定到我们选择的特定 Maven 阶段。**

## 2。AntRun 插件

首先，我们将讨论 [`AntRun`插件](/web/20221127022221/https://www.baeldung.com/maven-ant-task)。它提供了从 Maven 内部运行 Ant 任务的能力。为了在我们的项目中使用这个插件，我们需要将 [`maven-antrun-plugin`](https://web.archive.org/web/20221127022221/https://search.maven.org/search?q=g:org.apache.maven.plugins%20a:maven-antrun-plugin) 添加到我们的`pom.xml`中:

```java
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-antrun-plugin</artifactId>
        <version>3.0.0</version>
    </plugin>
</plugins>
```

让我们在`execution`标签中定义目标和阶段。此外，我们将添加保存带有`echo`消息的`target` 的`configuration`标签:

```java
<executions>
    <execution>
        <id>antrun-plugin</id>
        <phase>validate</phase>
        <goals>
            <goal>run</goal>
        </goals>
        <configuration>
            <target>
                <echo message="Hello, world"/>
                <echo message="Embed a line break: ${line.separator}"/>
                <echo message="Build dir: ${project.build.directory}" level="info"/>
                <echo file="${basedir}/logs/log-ant-run.txt" append="true" message="Save to file!"/>
            </target>
        </configuration>
    </execution>
</executions>
```

我们可以**打印常规字符串以及属性值**。`echo`标签向当前的记录器和监听器发送消息，除非被覆盖，否则它们对应于`System.out`。**我们还可以指定一个`level`** ，它告诉插件应该在哪个日志级别过滤`message`。

该任务也可以**到 `echo` 文件中。**我们可以通过将`append`属性分别设置为`true`或`false`来追加或覆盖一个文件。如果我们选择记录到一个文件中，我们应该省略记录`level`。只有标有`file`标记的消息才会被记录到文件中。

## 3.Echo Maven 插件

如果我们不想使用基于 Ant `,` 的插件，我们可以将 [`echo-maven-plugin`](https://web.archive.org/web/20221127022221/https://search.maven.org/search?q=g:com.github.ekryd.echo-maven-plugin%20a:echo-maven-plugin) 依赖项添加到我们的`pom.xml`中:

```java
<plugin>
    <groupId>com.github.ekryd.echo-maven-plugin</groupId>
    <artifactId>echo-maven-plugin</artifactId>
    <version>1.3.2</version>
</plugin>
```

就像我们在前面的插件例子中看到的，我们将在`execution`标签中声明目标和阶段。接下来，我们将填充`configuration`标签:

```java
<executions>
    <execution>
        <id>echo-maven-plugin-1</id>
        <phase>package</phase>
        <goals>
            <goal>echo</goal>
        </goals>
        <configuration>
            <message>
                Hello, world
                Embed a line break: ${line.separator}
                ArtifactId is ${project.artifactId}
            </message>
            <level>INFO</level>
            <toFile>/logs/log-echo.txt</toFile>
            <append>true</append>
        </configuration>
    </execution>
</executions>
```

类似地，我们可以打印简单的字符串和属性。我们还可以使用`level`标签来设置日志级别。使用`toFile` 标签，我们可以指出日志将被保存到的文件的路径。最后，如果我们想要打印多条消息，我们应该为每条消息添加一个单独的`execution`标签。

## 4.Groovy Maven 插件

要使用`[groovy-maven-plugin](https://web.archive.org/web/20221127022221/https://search.maven.org/search?q=g:org.codehaus.gmaven%20a:groovy-maven-plugin),`，我们必须将依赖关系放在我们的`pom.xml`中:

```java
<plugin>
    <groupId>org.codehaus.gmaven</groupId>
    <artifactId>groovy-maven-plugin</artifactId>
    <version>2.1.1</version>
</plugin>
```

此外，让我们在`execution`标签中添加阶段和目标。接下来，我们将把`source`标签放在`configuration`部分。它包含 Groovy 代码:

```java
<executions>
    <execution>
        <phase>validate</phase>
        <goals>
            <goal>execute</goal>
        </goals>
        <configuration>
            <source>
                log.info('Test message: {}', 'Hello, World!')
                log.info('Embed a line break {}', System.lineSeparator())
                log.info('ArtifactId is: ${project.artifactId}')
                log.warn('Message only in debug mode')
            </source>
        </configuration>
    </execution>
</executions>
```

与前面的解决方案类似，Groovy logger 允许我们设置日志记录级别。从代码层面，我们也可以很容易地访问 Maven 属性。此外，我们可以使用 Groovy 脚本将消息写入文件。

多亏了 Groovy 脚本，我们可以**给消息**添加更复杂的逻辑。Groovy 脚本**也可以从文件**中加载，所以我们不必用冗长的内联脚本来搞乱我们的`pom.xml`。

## 5.结论

在这个快速教程中，我们看到了如何使用各种插件进行打印。我们描述了如何使用`maven-antrun-plugin`、`echo-maven-plugin`和`groovy-maven-plugin`进行打印。此外，我们还讨论了几个用例。

最后，这篇文章的所有源代码都可以在 GitHub 的[上找到。](https://web.archive.org/web/20221127022221/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-printing-plugins)