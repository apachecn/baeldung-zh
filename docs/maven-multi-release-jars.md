# 使用 Maven 的多版本 JAR 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-multi-release-jars>

## 1。简介

Java 9 带给我们的一个新特性是构建[多版本 jar(MRJAR)](/web/20220823150607/https://www.baeldung.com/java-multi-release-jar)的能力。正如 [JDK 增强提案](https://web.archive.org/web/20220823150607/https://openjdk.java.net/jeps/238)所说，这允许我们在同一个 JAR 中拥有一个类的不同 Java 版本。

在本教程中，我们将探索如何使用 Maven 配置 MRJAR 文件。

## 2。肚子

[Maven](/web/20220823150607/https://www.baeldung.com/maven) 是 Java 生态系统中[最常用的构建工具](/web/20220823150607/https://www.baeldung.com/java-in-2019#build-tools-adoption)之一；它的功能之一是将项目打包到一个 JAR 中。

在接下来的小节中，我们将探索如何使用它来构建一个 MRJAR。

## 3。样本项目

先说一个基本的例子。

首先，我们将定义一个打印当前使用的 Java 版本的类；在 Java 9 之前，我们可以使用的方法之一是`System.getProperty`方法:

```java
public class DefaultVersion {
    public String version() {
        return System.getProperty("java.version");
    }
}
```

现在，从 Java 9 开始，我们可以使用来自`Runtime`类的新的`version`方法:

```java
public class DefaultVersion {
    public String version() {
        return Runtime.version().toString();
    }
}
```

用这种方法，我们可以得到一个 [`Runtime.Version`](https://web.archive.org/web/20220823150607/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Runtime.Version.html) 类实例，它给出了关于在[新版本中使用的 JVM 的信息——字符串模式格式](/web/20220823150607/https://www.baeldung.com/java-time-based-releases)。

另外，让我们添加一个`App`类来记录版本:

```java
public class App {

    private static final Logger logger = LoggerFactory.getLogger(App.class);

    public static void main(String[] args) {
        logger.info(String.format("Running on %s", new DefaultVersion().version()));
    }

}
```

最后，让我们将每个版本的`DefaultVersion` 放到它自己的`src/main`目录结构中:

```java
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── baeldung
│   │   │           └── multireleaseapp
│   │   │               ├── DefaultVersion.java
│   │   │               └── App.java
│   │   └── java9
│   │       └── com
│   │           └── baeldung
│   │               └── multireleaseapp
│   │                   └── DefaultVersion.java 
```

## 4。配置

为了从上面的类中配置 MRJAR，我们需要使用两个 Maven 插件:编译器插件和 JAR 插件。

### 4.1。Maven 编译器插件

在 Maven 编译器插件中，我们需要为将要打包的每个 Java 版本配置一个执行。

在这种情况下，我们添加两个:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <executions>
                <execution>
                    <id>compile-java-8</id>
                    <goals>
                        <goal>compile</goal>
                    </goals>
                    <configuration>
                        <source>1.8</source>
                        <target>1.8</target>
                    </configuration>
                </execution>
                <execution>
                    <id>compile-java-9</id>
                    <phase>compile</phase>
                    <goals>
                        <goal>compile</goal>
                    </goals>
                    <configuration>
                        <release>9</release>
                        <compileSourceRoots>
                            <compileSourceRoot>${project.basedir}/src/main/java9</compileSourceRoot>
                        </compileSourceRoots>
                        <outputDirectory>${project.build.outputDirectory}/META-INF/versions/9</outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

我们将使用第一次执行`compile-java-8`来编译我们的 Java 8 类，使用`compile-java-9`来编译我们的 Java 9 类。

我们可以看到**对于 Java 9 版本来说，有必要用各自的文件夹来配置`compileSourceRoot`和`outputDirectory`标签。**

但是，从[3 . 7 . 1](https://web.archive.org/web/20220823150607/https://maven.apache.org/plugins/maven-compiler-plugin/compile-mojo.html#multiReleaseOutput)开始，我们不需要手动设置输出目录。**相反，我们所要做的就是启用`multiReleaseOutput `属性**:

```java
<configuration> 
    <release>9</release> 
    <compileSourceRoots> 
        <compileSourceRoot>${project.basedir}/src/main/java9</compileSourceRoot> 
    </compileSourceRoots> 
    <multiReleaseOutput>true</multiReleaseOutput>
</configuration>
```

当设置为`true`时，编译器插件将所有特定于版本的类移动到`META-INF/versions/${release} `目录。**请注意，我们必须在这里将`release `** **标签设置为想要的 Java 版本，否则编译器插件失败**。

### 4.2。Maven JAR 插件

我们使用 JAR 插件将`MANIFEST`文件中的`Multi-Release`条目设置为`true`。有了这个配置，Java 运行时将在 JAR 文件的`META-INF/versions`文件夹中寻找特定于版本的类；否则，只使用基类。

让我们添加[`maven-jar-plugin` 配置](https://web.archive.org/web/20220823150607/https://search.maven.org/search?q=g:org.apache.maven.plugins%20AND%20a:maven-jar-plugin):

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <archive>
            <manifestEntries>
                <Multi-Release>true</Multi-Release>
            </manifestEntries>
        </archive>
    </configuration>
</plugin>
```

## 5。测试

是时候测试我们生成的 JAR 文件了。

当我们使用 Java 8 执行时，我们将看到以下输出:

```java
[main] INFO com.baeldung.multireleaseapp.App - Running on 1.8.0_252
```

但是如果我们用 Java 14 执行，我们会看到:

```java
[main] INFO com.baeldung.multireleaseapp.App - Running on 14.0.1+7
```

如我们所见，现在它使用了新的输出格式。**注意，尽管我们的 MRJAR 是用 Java 9 构建的，但它兼容多个主要的 Java 平台版本。**

## 6。结论

在这个简短的教程中，我们看到了如何配置 Maven 构建工具来生成一个简单的 MRJAR。

和往常一样，本教程中的完整代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220823150607/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-new-features)