# Java 9 编译器中的–release 选项是什么？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compiler-release-option>

## 1.概观

在本教程中，我们将了解 Java 9 的新命令行选项`–release.` 。使用`–release N`选项运行的 Java 编译器会自动生成与 Java 版本 N 兼容的类文件`*.*` 我们将讨论该选项与现有编译器命令行选项`-source`和`-target.`的关系

## 2.需要— `release`选项

为了理解对一个— `release`选项的需求，让我们考虑一个场景，我们需要用 Java 8 编译我们的代码，并且希望编译后的类与 Java 7 兼容。

在 Java 9 之前，可以通过使用— `source`和— `target`选项来实现这一点，其中

*   `-source:`指定编译器接受的 Java 版本
*   `-target:`指定要产生的类文件的 Java 版本

假设编译后的程序使用了当前版本平台(在我们的例子中是 Java 8)中独有的 API。在这种情况下，不管传递给–`source`和–`target`选项的值是多少，编译后的程序都无法在 Java 7 等早期版本上运行。

此外，我们需要添加–`bootclasspath`选项以及–`source`和–`target`,以便在 Java 版本 8 和更低版本中工作。

**为了简化这种交叉编译问题，Java 9 引入了新的选项— `release` 来简化这个过程。**

## 3.与-s `ource`和-t `arget`选项的关系

根据[JDK](https://web.archive.org/web/20220525124803/https://openjdk.java.net/jeps/247)的定义，`–release` N 可以展开为:

*   对于 N < 9，`-source`N`-target`N`-bootclasspath`documented-APIs-from-N>
*   对于 N >= 9，`-source`N`-target`N`–system`documented-APIs-from-N>

Here are a few details about these internal options:

*   用于搜索引导类文件的目录、JAR 档案和 ZIP 档案的分号分隔列表
*   — `system`:覆盖 Java 9 和更高版本的[系统模块](/web/20220525124803/https://www.baeldung.com/java-9-modularity)的位置

Also, the documented APIs are located in `$JDK_ROOT/lib/ct.sym`, which is a ZIP file containing class files stripped down according to the Java version.

对于 Java 版本 N< 9，这些 API 包括从位于`jre/lib/rt.jar`的 jar 和其他相关 jar 中检索的引导类。

对于 Java 版本 N >= 9，这些 API 包括从位于`jdkpath/jmods/`目录的 Java 模块中检索的引导类。

## 4.命令行的用法

首先，让我们创建一个示例类，并使用被覆盖的`ByteBuffer`的`flip`方法，它是在 Java 9:

```
import java.nio.ByteBuffer;

public class TestForRelease {

    public static void main(String[] args) {
        ByteBuffer bb = ByteBuffer.allocate(16);
        bb.flip();
        System.out.println("Baeldung: --release option test is successful");
    }
}
```

### 4.1.使用现有的源和目标选项

让我们用 Java 9 编译代码，将`-source`和`-target`选项值设为 8:

```
/jdk9path/bin/javac TestForRelease.java -source 8 -target 8 
```

这样做的结果是成功的，但是有一个警告:

```
warning: [options] bootstrap class path not set in conjunction with -source 8

1 warning
```

现在，让我们在 Java 8 上运行代码:

```
/jdk8path/bin/java TestForRelease
```

我们看到这失败了:

```
Exception in thread "main" java.lang.NoSuchMethodError: java.nio.ByteBuffer.flip()Ljava/nio/ByteBuffer;
at com.corejava.TestForRelease.main(TestForRelease.java:9)
```

正如我们所看到的，这不是我们期望在我们的`-release`和`-target`选项中看到的给定值 8。所以虽然编译器应该考虑，但事实并非如此。

让我们更详细地理解这一点。

在 Java 9 之前的版本中，`Buffer`类包含了`flip`方法:

```
public Buffer flip() {
    ...
 }
```

在 Java 9 中， *ByteBuffer，*扩展了*缓冲区，*覆盖了 *flip* 方法:

```
@Override
public ByteBuffer flip() {
    ...
}
```

当这个新方法在 Java 9 上编译并在 Java 8 上运行时，我们会得到错误，因为两个方法有不同的返回类型，并且使用描述符的方法查找在运行时会失败:

```
Exception in thread "main" java.lang.NoSuchMethodError: java.nio.ByteBuffer.flip()Ljava/nio/ByteBuffer;
at com.corejava.TestForRelease.main(TestForRelease.java:9)
```

在编译过程中，我们得到了之前忽略的警告。这是因为默认情况下， **Java 编译器使用最新的 API**进行编译。换句话说，编译器使用了 Java 9 类，尽管我们将`–source`和–target 指定为 8，所以我们的程序无法在 Java 8 上运行。

因此，我们必须将另一个名为–`bootclasspath`的命令行选项传递给 Java 编译器，以选择正确的版本。

现在，让我们用–`bootclasspath` 选项`:`重新编译相同的代码

```
/jdk9path/bin/javac TestForRelease.java -source 8 -target 8 -Xbootclasspath ${jdk8path}/jre/lib/rt.jar
```

同样，这样做的结果是成功的，这一次我们没有任何警告。

现在，让我们在 Java 8 上运行我们的代码，我们看到这是成功的:

```
/jdk8path/bin/java TestForRelease 
Baeldung: --release option test is successful 
```

虽然交叉编译现在可以工作了，但是我们必须提供三个命令行选项。

### 4.2.带–释放选项

现在，让我们用`–release`选项编译相同的代码:

```
/jdk9path/bin/javac TestForRelease.java —-release 8
```

这次编译再次成功，没有任何警告。

最后，当我们在 Java 8 上运行代码时，我们看到它是成功的:

```
/jdk8path/bin/java TestForRelease
Baeldung: --release option test is successful
```

我们看到，**使用— `release`选项很简单，因为`javac`在内部为`-source, -target,`和—`bootclasspath.`T5 设置了正确的值**

## 5.Maven 编译器插件的用法

通常，我们使用像 Maven 或 Gradle 这样的构建工具，而不是命令行`javac`工具。因此在这一节中，我们将看到如何在 maven 编译器插件中应用`–release`选项。

让我们首先看看如何使用现有的`-source`和`-target`选项:

```
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
 </plugins>
```

下面是我们如何使用`–release`选项:

```
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
        <configuration>
            <release>1.8</release>
        </configuration>
    </plugin>
 </plugins>
```

尽管行为与我们之前描述的一样，但是我们将这些值传递给 Java 编译器的方式是不同的。

## 6.结论

在本文中，我们了解了`–release`选项及其与现有的`-source`和`-target`选项的关系。然后，我们看到了如何在命令行和 Maven 编译器插件中使用该选项。

最后，我们看到新的— `release`选项需要更少的输入选项来进行交叉编译。因此，建议尽可能使用它来代替`-target, -source,`和`-bootclasspath` 选项`.`