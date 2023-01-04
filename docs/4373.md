# 用 Picocli 创建 Java 命令行程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-picocli-create-command-line-program>

## 1.介绍

在本教程中，我们将接近 [`picocli`库](https://web.archive.org/web/20220917215308/https://picocli.info/)，它允许我们轻松地用 Java 创建命令行程序。

我们将首先创建一个 Hello World 命令。然后，我们将通过部分复制`git `命令来深入探究该库的关键特性。

## 2.你好世界指挥部

让我们从简单的开始:Hello World 命令！

首先，我们需要将[依赖项添加到`picocli`项目](https://web.archive.org/web/20220917215308/https://search.maven.org/search?q=g:info.picocli%20AND%20a:picocli)中:

```
<dependency>
    <groupId>info.picocli</groupId>
    <artifactId>picocli</artifactId>
    <version>3.9.6</version>
</dependency>
```

正如我们所见，我们将使用库的`3.9.6`版本，尽管`4.0.0`版本正在构建中(目前在 alpha 测试中可用)。

既然已经设置了依赖项，那么让我们创建 Hello World 命令。为了做到这一点，**我们将使用来自库**的`@Command`注释:

```
@Command(
  name = "hello",
  description = "Says hello"
)
public class HelloWorldCommand {
}
```

正如我们看到的，注释可以带参数。我们只使用其中的两个。它们的目的是为自动帮助消息提供有关当前命令和文本的信息。

目前，我们对这个命令无能为力。为了让它做点什么，我们需要添加一个调用**的`main`方法，方便的`CommandLine.run(Runnable, String[])`方法**。这需要两个参数:一个是我们命令的实例，它必须实现 *Runnable* 接口，另一个是代表命令参数(选项、参数和子命令)的`String` 数组:

```
public class HelloWorldCommand implements Runnable {
    public static void main(String[] args) {
        CommandLine.run(new HelloWorldCommand(), args);
    }

    @Override
    public void run() {
        System.out.println("Hello World!");
    }
}
```

现在，当我们运行`main`方法时，我们将看到控制台输出`“Hello World!”`

[当打包到 jar](/web/20220917215308/https://www.baeldung.com/java-create-jar) 中时，我们可以使用`java`命令运行 Hello World 命令:

```
java -cp "pathToPicocliJar;pathToCommandJar" com.baeldung.picoli.helloworld.HelloWorldCommand
```

毫无疑问，它也将`“Hello World!”`字符串输出到控制台。

## 3.一个具体的用例

现在我们已经看到了基础知识，我们将深入研究`picocli`库。为了做到这一点，我们将部分复制一个流行的命令:`git`。

当然，目的不是实现`git`命令的行为，而是再现`git`命令的可能性——存在哪些子命令，对于一个特殊的子命令有哪些选项。

首先，我们必须创建一个`GitCommand`类，就像我们为 Hello World 命令所做的那样:

```
@Command
public class GitCommand implements Runnable {
    public static void main(String[] args) {
        CommandLine.run(new GitCommand(), args);
    }

    @Override
    public void run() {
        System.out.println("The popular git command");
    }
}
```

## 4.添加子命令

`git `命令提供了许多[子命令](https://web.archive.org/web/20220917215308/https://picocli.info/#_subcommands) — `add, commit, remote`，等等。这里我们将重点关注`add`和`commit`。

因此，我们的目标是向主命令声明这两个子命令。`Picocli`提供了三种方法来实现这一点。

### 4.1.在类上使用`@Command`注释

**`@Command`注释提供了通过`subcommands`参数**注册子命令的可能性:

```
@Command(
  subcommands = {
      GitAddCommand.class,
      GitCommitCommand.class
  }
)
```

在我们的例子中，我们添加了两个新的类:`GitAddCommand`和`GitCommitCommand`。两者都标注了`@Command`并实现了`Runnable`。**给它们一个名字很重要，因为这些名字将被`picocli`用来识别要执行的子命令:**

```
@Command(
  name = "add"
)
public class GitAddCommand implements Runnable {
    @Override
    public void run() {
        System.out.println("Adding some files to the staging area");
    }
}
```

```
@Command(
  name = "commit"
)
public class GitCommitCommand implements Runnable {
    @Override
    public void run() {
        System.out.println("Committing files in the staging area, how wonderful?");
    }
}
```

因此，如果我们使用`add`作为参数运行我们的主命令，控制台将输出`“Adding some files to the staging area”`。

### 4.2.在方法上使用`@Command`注释

声明子命令的另一种方式是**创建`@Command`带注释的方法，表示`GitCommand`类**中的那些命令:

```
@Command(name = "add")
public void addCommand() {
    System.out.println("Adding some files to the staging area");
}

@Command(name = "commit")
public void commitCommand() {
    System.out.println("Committing files in the staging area, how wonderful?");
}
```

这样，我们可以直接将业务逻辑实现到方法中，而不用创建单独的类来处理它。

### 4.3.以编程方式添加子命令

**最后，`picocli`为我们提供了以编程方式注册子命令的可能性。**这个有点复杂，因为我们必须创建一个`CommandLine`对象来包装我们的命令，然后向其中添加子命令:

```
CommandLine commandLine = new CommandLine(new GitCommand());
commandLine.addSubcommand("add", new GitAddCommand());
commandLine.addSubcommand("commit", new GitCommitCommand());
```

之后，我们仍然必须运行我们的命令，但是**我们不能再使用`CommandLine.run()`方法**。现在，我们必须在新创建的 C `ommandLine`对象上调用`parseWithHandler()`方法:

```
commandLine.parseWithHandler(new RunLast(), args);
```

我们应该注意到`RunLast`类的使用，它告诉 `picocli`运行最具体的子命令。`picocli`还提供了另外两个命令处理程序:`RunFirst`和`RunAll`。前者运行最顶层的命令，而后者运行所有命令。

当使用便利方法`CommandLine.run()`时，默认使用`RunLast`处理程序。

## 5.使用`@Option`注释管理选项

### 5.1.不带参数的选项

现在让我们看看如何给我们的命令添加一些选项。事实上，我们想告诉我们的`add`命令，它应该添加所有修改过的文件。为了实现这一点，**我们将向我们的`GitAddCommand`类添加一个用 [`@Option`](https://web.archive.org/web/20220917215308/https://picocli.info/#_options) 注释**注释的字段:

```
@Option(names = {"-A", "--all"})
private boolean allFiles;

@Override
public void run() {
    if (allFiles) {
        System.out.println("Adding all files to the staging area");
    } else {
        System.out.println("Adding some files to the staging area");
    }
}
```

正如我们看到的，注释带有一个`names`参数，它给出了选项的不同名称。因此，用`-A`或`–all`调用`add`命令会将`allFiles`字段设置为`true`。因此，如果我们运行带有选项的命令，控制台将显示`“Adding all files to the staging area”`。

### 5.2.带参数的选项

正如我们刚刚看到的，对于没有参数的选项，它们的存在与否总是被评估为一个`boolean`值。

然而，可以注册带参数的选项。**我们可以简单地通过声明我们的字段为不同的类型来做到这一点。**让我们给`commit`命令添加一个`message`选项:

```
@Option(names = {"-m", "--message"})
private String message;

@Override
public void run() {
    System.out.println("Committing files in the staging area, how wonderful?");
    if (message != null) {
        System.out.println("The commit message is " + message);
    }
}
```

不出所料，当给定`message`选项时，该命令将在控制台上显示提交消息。在本文的后面，我们将讨论哪些类型由库处理，以及如何处理其他类型。

### 5.3.具有多个参数的选项

但是现在，如果我们希望我们的命令接受多个消息，就像真正的 [`git commit`](https://web.archive.org/web/20220917215308/https://git-scm.com/docs/git-commit) 命令所做的那样，该怎么办呢？别担心，**让我们把我们的领域变成一个`array`或者一个`Collection`** ，我们已经基本完成了:

```
@Option(names = {"-m", "--message"})
private String[] messages;

@Override
public void run() {
    System.out.println("Committing files in the staging area, how wonderful?");
    if (messages != null) {
        System.out.println("The commit message is");
        for (String message : messages) {
            System.out.println(message);
        }
    }
}
```

现在，我们可以多次使用`message`选项:

```
commit -m "My commit is great" -m "My commit is beautiful"
```

但是，我们也可能希望只给选项一次，并用正则表达式分隔符分隔不同的参数。因此，我们可以使用`@Option`注释的`split`参数:

```
@Option(names = {"-m", "--message"}, split = ",")
private String[] messages;
```

现在，我们可以通过`-m “My commit is great”,”My commit is beautiful”`达到与上面相同的结果。

### 5.4.必需选项

**有时候，我们可能有一个必需的选项。默认为`false`的`required`参数允许我们这样做:**

```
@Option(names = {"-m", "--message"}, required = true)
private String[] messages;
```

现在不指定`message`选项就无法调用`commit`命令。如果我们尝试这样做，`picocli`将打印一个错误:

```
Missing required option '--message=<messages>'
Usage: git commit -m=<messages> [-m=<messages>]...
  -m, --message=<messages>
```

## 6.管理位置参数

### 6.1.捕获位置参数

现在，让我们专注于我们的`add`命令，因为它还不是很强大。我们只能决定添加所有文件，但是如果我们想添加特定的文件呢？

我们可以使用另一个选项来做到这一点，但这里更好的选择是使用位置参数。实际上，**位置参数意味着捕获占据特定位置的命令参数，它既不是子命令也不是选项。**

在我们的示例中，这将使我们能够做类似于以下的事情:

```
add file1 file2
```

为了捕捉**的位置参数，我们将利用 [`@Parameters`](https://web.archive.org/web/20220917215308/https://picocli.info/#_positional_parameters) 标注**:

```
@Parameters
private List<Path> files;

@Override
public void run() {
    if (allFiles) {
        System.out.println("Adding all files to the staging area");
    }

    if (files != null) {
        files.forEach(path -> System.out.println("Adding " + path + " to the staging area"));
    }
}
```

现在，我们之前的命令会打印出来:

```
Adding file1 to the staging area
Adding file2 to the staging area
```

### 6.2.捕获位置参数的子集

由于注释的`index` 参数，可以更细粒度地确定要捕获哪些位置参数。该索引是从零开始的。因此，如果我们定义:

```
@Parameters(index="2..*")
```

这将捕获不匹配选项或子命令的参数，从第三个到最后。

索引可以是代表单个位置的范围或单个数字。

## 7.关于类型转换的一句话

正如我们在本教程前面看到的，`picocli`自己处理一些类型转换。例如，它将多个值映射到`arrays`或`Collections`，但是它也可以将参数映射到特定的类型，比如当我们将`Path` 类用于`add`命令时。

**事实上，`picocli`自带[一堆预处理类型](https://web.archive.org/web/20220917215308/https://picocli.info/#_built_in_types)。这意味着我们可以直接使用这些类型，而不必考虑自己转换它们。**

然而，我们可能需要将我们的命令参数映射到那些已经被处理的类型之外的类型。对我们来说幸运的是，**多亏了 [`ITypeConverter`](https://web.archive.org/web/20220917215308/https://picocli.info/#_custom_type_converters) 接口和`CommandLine#registerConverter`方法，它将一个类型关联到一个转换器**。

假设我们想要将`config`子命令添加到`git`命令中，但是我们不希望用户更改不存在的配置元素。因此，我们决定将这些元素映射到一个枚举:

```
public enum ConfigElement {
    USERNAME("user.name"),
    EMAIL("user.email");

    private final String value;

    ConfigElement(String value) {
        this.value = value;
    }

    public String value() {
        return value;
    }

    public static ConfigElement from(String value) {
        return Arrays.stream(values())
          .filter(element -> element.value.equals(value))
          .findFirst()
          .orElseThrow(() -> new IllegalArgumentException("The argument " 
          + value + " doesn't match any ConfigElement"));
    }
}
```

另外，在我们新创建的`GitConfigCommand`类中，让我们添加两个位置参数:

```
@Parameters(index = "0")
private ConfigElement element;

@Parameters(index = "1")
private String value;

@Override
public void run() {
    System.out.println("Setting " + element.value() + " to " + value);
}
```

这样，我们可以确保用户不能更改不存在的配置元素。

最后，我们必须注册我们的转换器。美妙的是，如果使用 Java 8 或更高版本，我们甚至不必创建实现`ITypeConverter`接口的类。**我们可以将一个 lambda 或方法引用传递给`registerConverter()`方法:**

```
CommandLine commandLine = new CommandLine(new GitCommand());
commandLine.registerConverter(ConfigElement.class, ConfigElement::from);

commandLine.parseWithHandler(new RunLast(), args);
```

这发生在`GitCommand`的`main()` 方法中。注意，我们不得不放弃便利的`CommandLine.run()`方法。

当与未处理的配置元素一起使用时，该命令将显示帮助消息和一条信息，告诉我们无法将参数转换为`ConfigElement`:

```
Invalid value for positional parameter at index 0 (<element>): 
cannot convert 'user.phone' to ConfigElement 
(java.lang.IllegalArgumentException: The argument user.phone doesn't match any ConfigElement)
Usage: git config <element> <value>
      <element>
      <value>
```

## 8.与 Spring Boot 融合

最后，让我们来看看如何使这一切变得有弹性！

事实上，我们可能在 Spring Boot 环境中工作，并且希望在我们的命令行程序中受益于它。为了做到这一点，**我们必须创建一个`SpringBootApplication` 来实现`CommandLineRunner`接口**:

```
@SpringBootApplication
public class Application implements CommandLineRunner {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) {
    }
}
```

另外，**让我们用 Spring `@Component`注释**来注释我们所有的命令和子命令，并在我们的`Application`中自动连接它们:

```
private GitCommand gitCommand;
private GitAddCommand addCommand;
private GitCommitCommand commitCommand;
private GitConfigCommand configCommand;

public Application(GitCommand gitCommand, GitAddCommand addCommand, 
  GitCommitCommand commitCommand, GitConfigCommand configCommand) {
    this.gitCommand = gitCommand;
    this.addCommand = addCommand;
    this.commitCommand = commitCommand;
    this.configCommand = configCommand;
}
```

注意，我们必须自动连接每个子命令。不幸的是，这是因为，就目前而言，`picocli`还不能从声明性声明的 Spring 上下文中检索子命令(带注释)。因此，我们必须以编程的方式自己进行连接:

```
@Override
public void run(String... args) {
    CommandLine commandLine = new CommandLine(gitCommand);
    commandLine.addSubcommand("add", addCommand);
    commandLine.addSubcommand("commit", commitCommand);
    commandLine.addSubcommand("config", configCommand);

    commandLine.parseWithHandler(new CommandLine.RunLast(), args);
}
```

现在，我们的命令行程序在 Spring 组件上运行得非常好。因此，我们可以创建一些服务类，并在命令中使用它们，让 Spring 负责依赖注入。

## 9.结论

在本文中，我们已经看到了`picocli `库的一些关键特性。我们已经学习了如何创建一个新命令并向它添加一些子命令。我们已经看到了许多处理选项和位置参数的方法。此外，我们还学习了如何实现我们自己的类型转换器，使我们的命令具有强类型。最后，我们看到了如何将 Spring Boot 纳入我们的指挥范围。

当然，关于它还有很多东西需要发现。图书馆提供[完整的文档](https://web.archive.org/web/20220917215308/https://picocli.info/)。

至于这篇文章的完整代码，可以在[我们的 GitHub](https://web.archive.org/web/20220917215308/https://github.com/eugenp/tutorials/tree/master/libraries-2) 上找到。