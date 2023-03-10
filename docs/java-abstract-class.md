# Java 中的抽象类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-abstract-class>

## 1。概述

在许多情况下，当实现一个合同时，我们希望将实现的某些部分推迟到以后完成。我们可以通过抽象类在 Java 中轻松实现这一点。

在本教程中，我们将学习 Java 中抽象类的基础知识，以及在什么情况下它们会有所帮助。

## 2。抽象类的关键概念

在开始讨论什么时候使用抽象类之前，**让我们看看它们最相关的特征**:

*   **我们用`class`关键字**前面的`abstract`修饰符定义一个抽象类
*   抽象类可以被子类化，但是不能被实例化
*   如果一个类定义了一个或多个`abstract`方法，那么这个类本身必须声明为`abstract`
*   **抽象类可以声明抽象和具体方法**
*   从抽象类派生的子类必须要么实现基类的所有抽象方法，要么本身就是抽象的

为了更好地理解这些概念，我们将创建一个简单的例子。

让我们的基本抽象类定义一个棋盘游戏的抽象 API:

```java
public abstract class BoardGame {

    //... field declarations, constructors

    public abstract void play();

    //... concrete methods
}
```

然后，我们可以创建一个实现`play `方法的子类:

```java
public class Checkers extends BoardGame {

    public void play() {
        //... implementation
    }
}
```

## 3.何时使用抽象类

现在，**让我们分析几个典型的场景，在这些场景中，我们应该更喜欢抽象类而不是接口**和具体类:

*   我们希望将一些公共功能封装在一个地方(代码重用),供多个相关的子类共享
*   我们需要部分定义一个 API，我们的子类可以很容易地扩展和改进它
*   子类需要继承一个或多个带有受保护访问修饰符的公共方法或字段

让我们记住，所有这些场景都是完全的、基于继承的遵守[开放/封闭原则](https://web.archive.org/web/20221029133240/https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)的好例子。

此外，由于抽象类的使用隐含地处理了基本类型和子类型，我们还利用了[多态性](/web/20221029133240/https://www.baeldung.com/java-polymorphism)。

请注意，代码重用是使用抽象类的一个非常有说服力的理由，只要保留类层次结构中的“是-a”关系。

**和 [Java 8 用默认方法](/web/20221029133240/https://www.baeldung.com/java-static-default-methods)增加了另一个难题，它有时可以代替创建一个抽象类。**

## 4。文件读取器的示例层次结构

为了更清楚地理解抽象类给表带来的功能，让我们看另一个例子。

### 4.1。定义一个基础抽象类

因此，如果我们想要有几种类型的文件读取器，我们可以创建一个抽象类来封装文件读取的常见内容:

```java
public abstract class BaseFileReader {

    protected Path filePath;

    protected BaseFileReader(Path filePath) {
        this.filePath = filePath;
    }

    public Path getFilePath() {
        return filePath;
    }

    public List<String> readFile() throws IOException {
        return Files.lines(filePath)
          .map(this::mapFileLine).collect(Collectors.toList());
    }

    protected abstract String mapFileLine(String line);
}
```

请注意，我们已经将`filePath `设为受保护，以便子类可以在需要时访问它。更重要的是，**我们还有一些事情没有做:如何从文件内容中解析一行文本**。

我们的计划很简单:虽然我们的具体类没有各自存储文件路径或遍历文件的特殊方法，但它们都有转换每一行的特殊方法。

乍一看，`BaseFileReader`似乎没有必要。然而，它是一个干净的、易于扩展的设计的基础。通过它，**我们可以很容易地实现不同版本的文件阅读器，这些阅读器可以专注于它们独特的业务逻辑**。

### 4.2。定义子类

一个自然的实现可能是将文件内容转换成小写:

```java
public class LowercaseFileReader extends BaseFileReader {

    public LowercaseFileReader(Path filePath) {
        super(filePath);
    }

    @Override
    public String mapFileLine(String line) {
        return line.toLowerCase();
    }   
}
```

或者另一个可能是将文件的内容转换为大写:

```java
public class UppercaseFileReader extends BaseFileReader {

    public UppercaseFileReader(Path filePath) {
        super(filePath);
    }

    @Override
    public String mapFileLine(String line) {
        return line.toUpperCase();
    }
}
```

从这个简单的例子中我们可以看到，**每个子类都可以专注于其独特的行为**，而不需要指定文件读取的其他方面。

### 4.3。使用子类

最后，使用从抽象类继承的类与任何其他具体类没有什么不同:

```java
@Test
public void givenLowercaseFileReaderInstance_whenCalledreadFile_thenCorrect() throws Exception {
    URL location = getClass().getClassLoader().getResource("files/test.txt")
    Path path = Paths.get(location.toURI());
    BaseFileReader lowercaseFileReader = new LowercaseFileReader(path);

    assertThat(lowercaseFileReader.readFile()).isInstanceOf(List.class);
}
```

为了简单起见，目标文件位于 `src/main/resources/files`文件夹下。因此，我们使用应用程序类加载器来获取示例文件的路径。请随意查看我们的 Java 类加载器教程。

## 5。结论

在这篇简短的文章中，**我们学习了 Java 中抽象类的基础知识，以及何时使用它们来实现抽象和将公共实现封装在一个地方**。

像往常一样，本教程中显示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221029133240/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-inheritance)