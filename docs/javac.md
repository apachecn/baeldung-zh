# 编译 Java *。使用 javac 的类文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javac>

## 1。概述

本教程将介绍`javac`工具，并描述如何使用它将 Java 源文件编译成类文件。

我们将从对`javac`命令的简短描述开始，然后通过查看它的各种选项来更深入地研究这个工具。

## 2。`javac`命令

当执行`javac`工具时，我们可以指定选项和源文件:

```java
javac [options] [source-files]
```

其中`[options]`表示控制工具操作的选项，而`[source-files]`表示要编译的一个或多个源文件。

所有选项实际上都是可选的。源文件可以直接指定为`javac`命令的参数，或者保存在一个引用的参数文件中，如下所述。**注意，源文件应该按照它们包含的类型的完全限定名**排列在一个目录层次结构中。

`javac`的选项分为三组:标准、交叉编译和额外。在本文中，我们将关注标准选项和额外选项。

交叉编译选项用于在不同于编译器环境的 JVM 实现上编译类型定义的不太常见的用例，这里不做介绍。

## 3。类型定义

让我们从介绍我们将用来演示`javac`选项的类开始:

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

源代码放在文件`com/baeldung/javac/Data.java`中。

注意，我们在本文中使用*nix 文件分隔符；在 Windows 机器上，我们必须使用反斜杠(`\'`)，而不是正斜杠(`/'`)。

## 4。标准选项

`javac`命令最常用的标准选项之一是`-d`，**为生成的类文件**指定目标目录。如果一个类型不是缺省包的一部分，则创建一个反映包名的目录结构来保存该类型的类文件。

让我们在包含上一节中提供的结构的目录中执行以下命令:

```java
javac -d javac-target com/baeldung/javac/Data.java
```

`javac`编译器将生成类文件`javac-target/com/baeldung/javac/Data.class`。注意，在某些系统上，`javac`不会自动创建目标目录，在本例中是`javac-target`。因此，我们可能需要手动操作。

以下是一些其他常用的选项:

*   **`-cp`(或`-classpath`，`–class-path`)–**指定编译我们的源文件所需的类型可以在哪里找到。如果缺少该选项并且没有设置`CLASSPATH`环境变量，则使用当前的工作目录(如上例所示)。
*   **`-p`(或`–module-path`)–**表示必要应用模块的位置。该选项仅适用于 Java 9 及以上版本——请参考[本教程](/web/20220926194546/https://www.baeldung.com/project-jigsaw-java-modularity)获取 Java 9 模块系统指南。

如果我们想知道编译过程中发生了什么，例如哪些类被加载了，哪些被编译了，我们可以使用`-verbose`选项。

我们要讨论的最后一个标准选项是参数文件。**我们可以将参数存储在参数文件**中，而不是直接传递给`javac`工具。这些文件的名称以' @【T1]字符为前缀，然后被用作命令参数。

当`javac`命令遇到以“@ `‘`开头的参数时，它会将以下字符解释为文件的路径，并将文件的内容扩展到参数列表中。空格和换行符可以用来分隔这样的参数文件中包含的参数。

假设我们在`javac-args`目录中有两个文件，名为`options`和`types`，内容如下:

`options`文件:

```java
-d javac-target
-verbose
```

`types`文件:

```java
com/baeldung/javac/Data.java
```

我们可以像以前一样编译`Data`类型，并通过执行以下命令在控制台上打印详细消息:

```java
javac @javac-args/options @javac-args/types
```

除了将参数保存在单独的文件中，**我们还可以将它们全部保存在一个文件中**。

假设在`javac-args`目录中有一个名为`arguments`的文件:

```java
-d javac-target -verbose
com/baeldung/javac/Data.java
```

让我们将这个文件输入到`javac` 中，以获得与`:`之前的两个独立文件相同的结果

```java
javac @javac-args/arguments
```

请注意，我们在本节中讨论的选项只是最常见的选项。如需标准`javac`选项的完整列表，请查看[本参考](https://web.archive.org/web/20220926194546/https://docs.oracle.com/javase/9/tools/javac.htm#GUID-AEEC9F07-CB49-4E96-8BC7-BCC2C7F725C9__STANDARDOPTIONSFORJAVAC-7D3D9CC2)。

## 5。额外选项

`javac`的额外选项是非标准选项，是针对当前编译器实现的，将来可能会更改。因此，我们不会详细讨论这些选项。

然而，有一个选项非常有用，值得一提，`-Xlint`。关于其他`javac`额外选项的完整描述，请点击[此链接](https://web.archive.org/web/20220926194546/https://docs.oracle.com/javase/9/tools/javac.htm#GUID-AEEC9F07-CB49-4E96-8BC7-BCC2C7F725C9__NONSTANDARDOPTIONSFORJAVAC-7D3DAA9D)。

**`-Xlint`选项允许我们在编译**期间启用警告。有两种方法可以在命令行上指定该选项:

*   **`-Xlint –`** 触发所有推荐警告
*   **`-Xlint:key[,key]* –`** 启用特定警告

以下是一些最方便的钥匙:

*   **`rawtypes –`** 警告不要使用原始类型
*   **`unchecked –`** 对未经检查的操作发出警告
*   **`static –`** 警告从实例成员访问静态成员
*   **`cast –`** 警告不必要的施法
*   **`serial –`** 警告没有`serialversionUID`的可序列化类
*   **`fallthrough –`** 在一个`switch`语句中警告关于穿越的坠落

现在，在`javac-args`目录下创建一个名为`xlint-ops`的文件，内容如下:

```java
-d javac-target
-Xlint:rawtypes,unchecked
com/baeldung/javac/Data.java
```

运行此命令时:

```java
javac @javac-args/xlint-ops
```

我们应该会看到`rawtypes` 和`unchecked`警告:

```java
com/baeldung/javac/Data.java:7: warning: [rawtypes] found raw type: ArrayList
    List<String> textList = new ArrayList();
                                ^
  missing type arguments for generic class ArrayList<E>
  where E is a type-variable:
    E extends Object declared in class ArrayList
com/baeldung/javac/Data.java:7: warning: [unchecked] unchecked conversion
    List<String> textList = new ArrayList();
                            ^
  required: List<String>
  found:    ArrayList
...
```

## 6。结论

本教程介绍了`javac`工具，展示了如何使用选项来管理典型的编译过程。

在现实中，我们通常使用 IDE 或构建工具来编译程序，而不是直接依赖于`javac`。然而，对这个工具的扎实理解将允许我们在高级用例中定制编译。

一如既往，本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926194546/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java)