# 用 Airline 解析命令行参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-airline>

## 1.介绍

在本教程中，**我们将介绍[航空公司](https://web.archive.org/web/20221128043700/https://rvesse.github.io/airline/)——一个用于构建命令行界面(CLI)的注释驱动的 Java 库。**

## 2.方案

在构建命令行应用程序时，很自然地会创建一个简单的界面，允许用户根据需要来塑造输出。几乎每个人都玩过 Git CLI，都知道它是多么强大而简单。唉，在构建这样的界面时，很少有工具能派上用场。

**航空公司** **旨在减少 Java** 中通常与 CLI 相关的样板代码，因为大多数常见行为都可以通过注释和零用户代码实现。

我们将实现一个小的 Java 程序，它将利用 Airline 的功能来模仿一个公共的 CLI。它将公开设置程序配置的用户命令，比如定义数据库 URL、凭证和日志详细程度。我们还将深入本库的表层，使用比基础更多的东西来探索它是否能处理一些复杂性。

## 3.设置

首先，让我们将[航空公司](https://web.archive.org/web/20221128043700/https://search.maven.org/search?q=g:com.github.rvesse%20a:airline)添加到我们的`pom.xm` l:

```
<dependency>
    <groupId>com.github.rvesse</groupId>
    <artifactId>airline</artifactId>
    <version>2.7.2</version>
</dependency> 
```

## 4.简单的命令行界面

让我们为应用程序创建入口点——`CommandLine`类:

```
@Cli(name = "baeldung-cli",
  description = "Baeldung Airline Tutorial",
  defaultCommand = Help.class)
public class CommandLine {
    public static void main(String[] args) {
        Cli<Runnable> cli = new Cli<>(CommandLine.class);
        Runnable cmd = cli.parse(args);
        cmd.run();
    }
}
```

通过一个简单的`@Cli`注释，我们已经定义了将在我们的应用程序上运行的默认命令—`Help`命令。

`Help`类是 Airline 库的一部分，使用`-h`或`–help`选项公开默认的帮助命令。

就这样，基本设置完成了。

## 5.我们的第一个命令

让我们实现我们的第一个命令，一个简单的`LoggingCommand`类，它将控制我们日志的详细程度。我们将使用`@Command`对该类进行注释，以确保当用户调用`setup-log`时应用了正确的命令:

```
@Command(name = "setup-log", description = "Setup our log")
public class LoggingCommand implements Runnable {

    @Inject
    private HelpOption<LoggingCommand> help;

    @Option(name = { "-v", "--verbose" }, 
      description = "Set log verbosity on/off")
    private boolean verbose = false;

    @Override
    public void run() {
        if (!help.showHelpIfRequested())
            System.out.println("Verbosity: " + verbose);
        }
    }
}
```

让我们仔细看看我们的示例命令。

首先，我们已经设置了一个描述，这样我们的助手，由于注入，将在请求时显示我们的命令选项。

然后我们声明了一个`boolean`变量`verbose`，并用`@Option`对其进行了注释，给它一个名称、描述以及一个别名`-v/–verbose`来表示我们控制详细程度的命令行选项。

最后，在`run`方法中，我们指示我们的命令在用户请求帮助时停止。

到目前为止，一切顺利。现在，我们需要通过修改@ `Cli`注释将新命令添加到主界面:

```
@Cli(name = "baeldung-cli",
description = "Baeldung Airline Tutorial",
defaultCommand = Help.class,
commands = { LoggingCommand.class, Help.class })
public class CommandLine {
    public static void main(String[] args) {
        Cli<Runnable> cli = new Cli<>(CommandLine.class);
        Runnable cmd = cli.parse(args);
        cmd.run();
    }
} 
```

现在，如果我们将`setup-log -v`传递给我们的程序，它将运行我们的逻辑。

## 6.约束和更多

我们已经看到了航空公司如何完美地生成 CLI，但是…还有更多！

我们可以为我们的参数指定约束(或限制),以处理允许的值、需求或依赖性等等。

我们将创建一个`DatabaseSetupCommand`类，它将响应`setup-db`命令；和我们之前做的一样，但是我们会加入一些香料。

首先，我们将请求数据库的类型，通过`@AllowedRawValues`只接受 3 个有效值:

```
@AllowedRawValues(allowedValues = { "mysql", "postgresql", "mongodb" })
@Option(type = OptionType.COMMAND,
  name = {"-d", "--database"},
  description = "Type of RDBMS.",
  title = "RDBMS type: mysql|postgresql|mongodb")
protected String rdbmsMode;
```

毫无疑问，在使用数据库连接时，用户应该提供一个端点和一些凭证来访问它。我们将让 CLI 通过一个(`URL mode)`或多个参数(`host mode`)来处理这个问题。为此，我们将使用`@MutuallyExclusiveWith`注释，用相同的标签标记每个参数:

```
@Option(type = OptionType.COMMAND,
  name = {"--rdbms:url", "--url"},
  description = "URL to use for connection to RDBMS.",
  title = "RDBMS URL")
@MutuallyExclusiveWith(tag="mode")
@Pattern(pattern="^(http://.*):(d*)(.*)u=(.*)&p;=(.*)")
protected String rdbmsUrl = "";

@Option(type = OptionType.COMMAND,
  name = {"--rdbms:host", "--host"},
  description = "Host to use for connection to RDBMS.",
  title = "RDBMS host")
@MutuallyExclusiveWith(tag="mode")
protected String rdbmsHost = ""; 
```

注意，我们使用了`@Pattern`装饰器，它帮助我们定义 URL 字符串格式。

如果我们查看项目文档，我们会发现其他**有价值的工具，用于处理需求、事件、允许值、特定案例等等，使我们能够定义我们的定制规则**。

最后，如果用户选择了主机模式，我们应该要求他们提供凭据。这样，一个选项依赖于另一个选项。我们可以用`@RequiredOnlyIf`注释实现这种行为:

```
@RequiredOnlyIf(names={"--rdbms:host", "--host"})
@Option(type = OptionType.COMMAND,
  name = {"--rdbms:user", "-u", "--user"},
  description = "User for login to RDBMS.",
  title = "RDBMS user")
protected String rdbmsUser;

@RequiredOnlyIf(names={"--rdbms:host", "--host"})
@Option(type = OptionType.COMMAND,
  name = {"--rdbms:password", "--password"},
  description = "Password for login to RDBMS.",
  title = "RDBMS password")
protected String rdbmsPassword; 
```

如果我们需要使用一些驱动程序来处理 DB 连接呢？另外，假设我们需要在单个参数中接收多个值。我们可以将选项类型更改为`OptionType.ARGUMENTS`,或者更好的是接受一个值列表:

```
@Option(type = OptionType.COMMAND,
  name = {"--driver", "--jars"},
  description = "List of drivers",
  title = "--driver <PATH_TO_YOUR_JAR> --driver <PATH_TO_YOUR_JAR>")
protected List<String> jars = new ArrayList<>();
```

现在，我们不要忘记将数据库设置命令添加到我们的主类中。否则，它将无法在 CLI 上使用。

## 7.奔跑

我们做到了！我们完成了我们的项目，现在我们可以运行它了。

不出所料，在没有传递任何参数的情况下，`Help`被调用:

```
$ baeldung-cli

usage: baeldung-cli <command> [ <args> ]

Commands are:
    help        Display help information
    setup-db    Setup our database
    setup-log   Setup our log

See 'baeldung-cli help <command>' for more information on a specific command.
```

如果我们改为执行`setup-log –help`，我们会得到:

```
$ baeldung-cli setup-log --help

NAME
        baeldung-cli setup-log - Setup our log

SYNOPSIS
        baeldung-cli setup-log [ {-h | --help} ] [ {-v | --verbose} ]

OPTIONS
        -h, --help
            Display help information

        -v, --verbose
            Set log verbosity on/off
```

最后，为这些命令提供参数将会运行正确的业务逻辑。

## 8.结论

在本文中，我们用很少的代码构建了一个简单而强大的命令行界面。

航空公司库凭借其强大的功能简化了 CLI，为我们提供了一个通用、干净且可重复使用的基础设施。它允许我们，开发人员，专注于我们的业务逻辑，而不是花时间去设计那些琐碎的东西。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128043700/https://github.com/eugenp/tutorials/tree/master/libraries-3)