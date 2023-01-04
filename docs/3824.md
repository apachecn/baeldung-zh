# 在 Maven 中运行 Java Main 方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-java-main-method>

## 1.概观

在这个简短的教程中，我们将看到如何使用 Maven 从任何 Java 类运行任意的 [main 方法](/web/20221111013907/https://www.baeldung.com/java-main-method)。

## 2.`exec-maven-plugin`

假设我们有下面的类:

```
public class Exec {

    private static final Logger LOGGER = LoggerFactory.getLogger(Exec.class);

    public static void main(String[] args) {
        LOGGER.info("Running the main method");
        if (args.length > 0) {
            LOGGER.info("List of arguments: {}", Arrays.toString(args));
        }
    }
}
```

我们想通过 Maven 从命令行执行它的 main 方法。

**为了做到这一点，我们可以使用 [`exec-maven-plugin`](https://web.archive.org/web/20221111013907/https://www.mojohaus.org/exec-maven-plugin/) 。更具体地说，这个插件中的`exec:java ` [目标](/web/20221111013907/https://www.baeldung.com/maven-goals-phases)执行所提供的 Java 类，将封闭项目的依赖项作为类路径。**

为了执行`Exec`类的 main 方法，我们必须将类的完全限定名传递给插件:

```
$ mvn compile exec:java -Dexec.mainClass="com.baeldung.main.Exec"
02:26:45.112 INFO com.baeldung.main.Exec - Running the main method
```

如上所示，我们使用`exec.mainClass `系统属性来传递完全限定的类名。

此外，我们必须确保在运行 main 方法之前，类路径已经准备好。这就是我们在执行 main 方法之前编译源代码的原因。

我们可以用普通的`java `和`javac. `实现同样的事情，但是，当我们使用一个相当大的类路径时，这可能会很麻烦。相反， **使用这个插件时，Maven 会自动负责填充类路径。**

## 3.传递参数

也可以从命令行向 main 方法传递参数。为了做到这一点，我们可以使用`exec.args `系统属性:

```
$ mvn compile exec:java -Dexec.mainClass="com.baeldung.main.Exec" \
  -Dexec.args="First Second"
02:31:08.235 INFO com.baeldung.main.Exec - Running the main method
02:31:08.236 INFO com.baeldung.main.Exec - List of arguments: [First, Second]
```

如上所示，我们正在传递一个空格分隔的参数列表。此外，我们可以通过`exec.arguments `系统属性使用逗号分隔的参数列表:

```
$ mvn compile exec:java -Dexec.mainClass="com.baeldung.main.Exec" \ 
  -Dexec.arguments="Hello World,Bye"
02:32:25.616 INFO com.baeldung.main.Exec - Running the main method
02:32:25.618 INFO com.baeldung.main.Exec - List of arguments: [Hello World, Bye]
```

当我们想在参数本身中使用分隔符(空格或逗号)时，这两个选项会很有用。

## 4.自定义配置

我们也可以在我们的`pom.xml`中显式声明插件依赖。这样，我们可以使用定制和默认配置。

例如，我们可以在插件的配置中指定一个默认的主类:

```
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <version>3.0.0</version>
            <configuration>
                <mainClass>com.baeldung.main.Exec</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>
```

现在，如果我们不指定所需类的完全限定名，将使用`com.baeldung.main.Exec `:

```
$ mvn compile exec:java
02:33:14.197 INFO com.baeldung.main.Exec - Running the main method
```

然而，仍然可以通过显式的`exec`来覆盖这个默认配置。`mainClass `系统属性。

此外，我们还可以在配置中指定默认的程序参数:

```
<configuration>
    <mainClass>com.baeldung.main.Exec</mainClass>
    <arguments>
        <argument>First</argument>
        <argument>Second</argument>
    </arguments>
</configuration>
```

这样我们就不需要在命令行上传递这些参数了:

```
$ mvn clean compile exec:java
02:34:24.448 INFO com.baeldung.main.Exec - Running the main method
02:34:24.450 INFO com.baeldung.main.Exec - List of arguments: [First, Second]
```

除了这些配置之外，还有更多可用的配置包含在[官方文档](https://web.archive.org/web/20221111013907/https://www.mojohaus.org/exec-maven-plugin/java-mojo.html)中。

## 5.结论

在这篇短文中，我们看到了如何通过`exec-maven-plugin`从命令行运行 main 方法。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221111013907/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-exec-plugin)