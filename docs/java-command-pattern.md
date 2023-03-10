# Java 中的命令模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-command-pattern>

## 1。概述

命令模式是一种行为设计模式，是 [GoF](https://web.archive.org/web/20221024094653/https://en.wikipedia.org/wiki/Design_Patterns) 的正式设计模式列表的一部分。简单地说，该模式旨在**将执行给定动作(命令)所需的所有数据封装在一个对象中，**包括调用什么方法、方法的参数以及方法所属的对象。

这个模型允许我们**将产生命令的对象从它们的消费者**中分离出来，所以这就是为什么这个模式通常被称为生产者-消费者模式。

在本教程中，我们将学习如何通过使用面向对象和对象函数的方法在 Java 中实现 command 模式，并且我们将看到它在什么用例中是有用的。

## 2。面向对象的实现

在经典实现中，命令模式要求**实现四个组件:命令、接收者、调用者和客户端**。

为了理解模式如何工作以及每个组件扮演的角色，让我们创建一个基本的例子。

假设我们想要开发一个文本文件应用程序。在这种情况下，我们应该实现执行一些文本文件相关操作所需的所有功能，比如打开、写入、保存文本文件等等。

因此，我们应该将应用程序分解成上面提到的四个组件。

### 2.1。命令类别

命令是一个对象，其作用是**存储执行动作**所需的所有信息，包括要调用的方法、方法参数和实现该方法的对象(称为接收者)。

为了更准确地了解命令对象是如何工作的，让我们开始开发一个简单的命令层，它只包含一个接口和两个实现:

```java
@FunctionalInterface
public interface TextFileOperation {
    String execute();
}
```

```java
public class OpenTextFileOperation implements TextFileOperation {

    private TextFile textFile;

    // constructors

    @Override
    public String execute() {
        return textFile.open();
    }
}
```

```java
public class SaveTextFileOperation implements TextFileOperation {

    // same field and constructor as above

    @Override
    public String execute() {
        return textFile.save();
    }
} 
```

在这种情况下，`TextFileOperation`接口定义了命令对象的 API，而两个实现，`OpenTextFileOperation`和`SaveTextFileOperation,`执行具体的动作。前者打开文本文件，后者保存文本文件。

命令对象的功能显而易见:`TextFileOperation`命令**封装了打开和保存文本文件所需的所有信息**，包括接收者对象、要调用的方法和参数(在这种情况下，不需要参数，但它们可以是)。

值得强调的是**执行文件操作的组件是接收者(`TextFile`实例)**。

### 2.2。接收器类别

接收者是**执行一组内聚动作**的对象。当命令的 `execute()`方法被调用时，它是执行实际动作的组件。

在这种情况下，我们需要定义一个 receiver 类，它的作用是建模`TextFile`对象:

```java
public class TextFile {

    private String name;

    // constructor

    public String open() {
        return "Opening file " + name;
    }

    public String save() {  
        return "Saving file " + name;
    }

    // additional text file methods (editing, writing, copying, pasting)
} 
```

### 2.3。调用者类

invoker 是一个对象，**知道如何执行给定的命令，但不知道命令是如何实现的。**它只知道命令的接口。

在某些情况下，调用程序除了执行命令之外，还存储命令并对其进行排队。这对于实现一些附加功能很有用，比如宏记录或撤销和重做功能。

在我们的例子中，很明显，必须有一个额外的组件负责调用命令对象并通过命令的方法执行它们。**这正是 invoker 类发挥作用的地方**。

让我们看看调用者的一个基本实现:

```java
public class TextFileOperationExecutor {

    private final List<TextFileOperation> textFileOperations
     = new ArrayList<>();

    public String executeOperation(TextFileOperation textFileOperation) {
        textFileOperations.add(textFileOperation);
        return textFileOperation.execute();
    }
}
```

`TextFileOperationExecutor`类只是一个**抽象薄层，它将命令对象从它们的消费者**中分离出来，并调用封装在`TextFileOperation`命令对象中的方法。

在这种情况下，该类还将命令对象存储在一个`List`中。当然，这在模式实现中不是强制性的，除非我们需要向操作的执行过程添加一些进一步的控制。

### 2.4。客户端类

客户端是一个对象，**通过指定执行什么命令以及在过程的什么阶段执行它们来控制命令执行过程**。

因此，如果我们想要正统地使用模式的正式定义，我们必须使用典型的`main`方法创建一个客户端类:

```java
public static void main(String[] args) {
    TextFileOperationExecutor textFileOperationExecutor
      = new TextFileOperationExecutor();
    textFileOperationExecutor.executeOperation(
      new OpenTextFileOperation(new TextFile("file1.txt"))));
    textFileOperationExecutor.executeOperation(
      new SaveTextFileOperation(new TextFile("file2.txt"))));
} 
```

## 3。目标功能实现

到目前为止，我们已经使用了面向对象的方法来实现命令模式，这很好。

在 Java 8 中，我们可以使用一种基于 lambda 表达式和方法引用的对象函数方法，使代码更加紧凑，更加简洁。

### 3.1。使用λ表达式

由于`TextFileOperation`接口是一个[功能接口](https://web.archive.org/web/20221024094653/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/package-summary.html)，我们可以**以 lambda 表达式的形式将命令对象传递给调用者**，而不必显式创建`TextFileOperation`实例:

```java
TextFileOperationExecutor textFileOperationExecutor
 = new TextFileOperationExecutor();
textFileOperationExecutor.executeOperation(() -> "Opening file file1.txt");
textFileOperationExecutor.executeOperation(() -> "Saving file file1.txt"); 
```

实现现在看起来更加精简，因为我们已经减少了样板代码的数量。

即便如此，问题仍然存在:与面向对象的方法相比，这种方法更好吗？

嗯，那很棘手。如果我们假设在大多数情况下更紧凑的代码意味着更好的代码，那么事实确实如此。

作为一个经验法则，我们应该在每个用例的基础上评估何时求助于 lambda 表达式。

### 3.2。使用方法引用

类似地，我们可以使用**的方法引用将命令对象传递给调用者:**

```java
TextFileOperationExecutor textFileOperationExecutor
 = new TextFileOperationExecutor();
TextFile textFile = new TextFile("file1.txt");
textFileOperationExecutor.executeOperation(textFile::open);
textFileOperationExecutor.executeOperation(textFile::save); 
```

在这种情况下，实现**比使用 lambdas** 的实现稍微冗长一点，因为我们仍然必须创建`TextFile`实例。

## 4。结论

在本文中，我们学习了命令模式的关键概念，以及如何通过使用面向对象的方法以及 lambda 表达式和方法引用的组合在 Java 中实现该模式。

像往常一样，本教程中显示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221024094653/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-behavioral)