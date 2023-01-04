# 带有 ANTLR 的 Java

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-antlr>

## 1。概述

在本教程中，我们将快速概述一下 [ANTLR](https://web.archive.org/web/20220625235751/http://www.antlr.org/) 解析器生成器，并展示一些实际应用。

## 2。ANTLR

ANTLR(另一种语言识别工具)是一种处理结构化文本的工具。

它让我们能够访问语言处理原语，如词法分析器、语法分析器和运行时来处理文本。

它通常用于构建工具和框架。例如，Hibernate 使用 ANTLR 来解析和处理 HQL 查询，而 Elasticsearch 使用它来实现无痛查询。

而 Java 只是一个绑定。ANTLR 还提供 C#、Python、JavaScript、Go、C++和 Swift 的绑定。

## 3。配置

首先，让我们从添加 [antlr-runtime](https://web.archive.org/web/20220625235751/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.antlr%22%20AND%20a%3A%22antlr4-runtime%22) 到我们的`pom.xml`开始:

```java
<dependency>
    <groupId>org.antlr</groupId>
    <artifactId>antlr4-runtime</artifactId>
    <version>4.7.1</version>
</dependency>
```

还有 [antlr-maven 插件](https://web.archive.org/web/20220625235751/https://search.maven.org/classic/#search%7Cga%7C1%7Corg.antlr%20antlr-maven-plugin):

```java
<plugin>
    <groupId>org.antlr</groupId>
    <artifactId>antlr4-maven-plugin</artifactId>
    <version>4.7.1</version>
    <executions>
        <execution>
            <goals>
                <goal>antlr4</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

插件的工作是从我们指定的语法中生成代码。

## 4。它是如何工作的？

基本上，当我们想要使用 [ANTLR Maven 插件](https://web.archive.org/web/20220625235751/http://www.antlr.org/api/maven-plugin/latest/)创建解析器时，我们需要遵循三个简单的步骤:

*   **准备一个语法文件**
*   **生成源**
*   **创建监听器**

那么，让我们来看看这些步骤的实际操作。

## 5。使用现有语法

让我们首先使用 ANTLR 来分析大小写错误的方法的代码:

```java
public class SampleClass {

    public void DoSomethingElse() {
        //...
    }
}
```

简单地说，我们将验证代码中的所有方法名称都以小写字母开头。

### 5.1。准备一个语法文件

好在已经有几个语法文件可以满足我们的目的。

让我们使用[在](https://web.archive.org/web/20220625235751/https://github.com/antlr/codebuff/blob/master/grammars/org/antlr/codebuff/Java8.g4) [ANTLR 的 Github 语法报告](https://web.archive.org/web/20220625235751/https://github.com/antlr/grammars-v4/tree/master/java/java8)中找到的 Java8.g4 语法文件。

我们可以创建`src/main/antlr4`目录并在那里下载。

### 5.2。生成源

ANTLR 通过生成对应于我们给它的语法文件的 Java 代码来工作，maven 插件使它变得简单:

```java
mvn package
```

默认情况下，这将在`target/generated-sources/antlr4`目录下生成几个文件:

*   `Java8.interp`
*   `Java8Listener.java`
*   `Java8BaseListener.java`
*   `Java8Lexer.java`
*   `Java8Lexer.interp`
*   `Java8Parser.java`
*   `Java8.tokens`
*   `Java8Lexer.tokens`

注意，这些文件的名称**是基于语法文件**的名称。

我们稍后测试时将需要`Java8Lexer `和`Java8Parser`文件。不过现在，我们需要`Java8BaseListener`来创建我们的`MethodUppercaseListener`。

### 5.3。创造`MethodUppercaseListener`

基于我们使用的 Java8 语法，`Java8BaseListener` 有几个我们可以覆盖的方法，每个方法对应于语法文件中的一个标题。

例如，语法定义了方法名、参数列表和 throws 子句，如下所示:

```java
methodDeclarator
	:	Identifier '(' formalParameterList? ')' dims?
	;
```

因此`Java8BaseListener `有一个方法`enterMethodDeclarator `,每次遇到这种模式时都会被调用。

因此，让我们忽略`enterMethodDeclarator`，拔出`Identifier`，并执行我们的检查:

```java
public class UppercaseMethodListener extends Java8BaseListener {

    private List<String> errors = new ArrayList<>();

    // ... getter for errors

    @Override
    public void enterMethodDeclarator(Java8Parser.MethodDeclaratorContext ctx) {
        TerminalNode node = ctx.Identifier();
        String methodName = node.getText();

        if (Character.isUpperCase(methodName.charAt(0))) {
            String error = String.format("Method %s is uppercased!", methodName);
            errors.add(error);
        }
    }
}
```

### 5.4。测试

现在，让我们做一些测试。首先，我们构建 lexer:

```java
String javaClassContent = "public class SampleClass { void DoSomething(){} }";
Java8Lexer java8Lexer = new Java8Lexer(CharStreams.fromString(javaClassContent));
```

然后，我们实例化解析器:

```java
CommonTokenStream tokens = new CommonTokenStream(lexer);
Java8Parser parser = new Java8Parser(tokens);
ParseTree tree = parser.compilationUnit();
```

然后，行者和听者:

```java
ParseTreeWalker walker = new ParseTreeWalker();
UppercaseMethodListener listener= new UppercaseMethodListener();
```

最后，我们告诉 ANTLR 遍历我们的示例类`:`

```java
walker.walk(listener, tree);

assertThat(listener.getErrors().size(), is(1));
assertThat(listener.getErrors().get(0),
  is("Method DoSomething is uppercased!"));
```

## 6。构建我们的语法

现在，让我们尝试一些稍微复杂一点的东西，比如解析日志文件:

```java
2018-May-05 14:20:18 INFO some error occurred
2018-May-05 14:20:19 INFO yet another error
2018-May-05 14:20:20 INFO some method started
2018-May-05 14:20:21 DEBUG another method started
2018-May-05 14:20:21 DEBUG entering awesome method
2018-May-05 14:20:24 ERROR Bad thing happened
```

因为我们有一个自定义的日志格式，所以我们首先需要创建自己的语法。

### 6.1。准备一个语法文件

首先，让我们看看是否可以为文件中的每个日志行创建一个心理地图。

`<datetime> <level> <message>`

或者如果我们再深入一层，我们可能会说:

`<datetime> := <year><dash><month><dash><day> …`

诸如此类。考虑这一点很重要，这样我们就可以决定以什么样的粒度来解析文本。

语法文件基本上是一组词法分析器和语法分析器规则。简单地说，词法分析器规则描述语法的句法，而语法分析器规则描述语义。

让我们从定义片段开始，这些片段是 lexer 规则的可重用构件。

```java
fragment DIGIT : [0-9];
fragment TWODIGIT : DIGIT DIGIT;
fragment LETTER : [A-Za-z];
```

接下来，让我们定义剩余的 lexer 规则:

```java
DATE : TWODIGIT TWODIGIT '-' LETTER LETTER LETTER '-' TWODIGIT;
TIME : TWODIGIT ':' TWODIGIT ':' TWODIGIT;
TEXT   : LETTER+ ;
CRLF : '\r'? '\n' | '\r';
```

有了这些构建块，我们就可以为基本结构构建解析器规则了:

```java
log : entry+;
entry : timestamp ' ' level ' ' message CRLF;
```

然后我们将添加`timestamp`的细节:

```java
timestamp : DATE ' ' TIME;
```

对于`level`:

```java
level : 'ERROR' | 'INFO' | 'DEBUG';
```

对于`message`:

```java
message : (TEXT | ' ')+;
```

就是这样！我们的语法已经可以使用了。我们会像以前一样把它放在`src/main/antlr4` 目录下。

### 6.2。 **生成来源**

回想一下，这只是一个快速的`mvn package`，这将基于我们的语法名称创建几个文件，如`LogBaseListener`、`LogParser`等等。

### 6.3。创建我们的日志监听器

现在，我们准备实现我们的侦听器，我们最终将使用它将日志文件解析成 Java 对象。

因此，让我们从一个简单的日志条目模型类开始:

```java
public class LogEntry {

    private LogLevel level;
    private String message;
    private LocalDateTime timestamp;

    // getters and setters
}
```

现在，我们需要像以前一样子类化`LogBaseListener`:

```java
public class LogListener extends LogBaseListener {

    private List<LogEntry> entries = new ArrayList<>();
    private LogEntry current;
```

`current `将保留当前的日志行，我们可以根据我们的语法在每次再次输入`logEntry, `时重新初始化它:

```java
 @Override
    public void enterEntry(LogParser.EntryContext ctx) {
        this.current = new LogEntry();
    }
```

接下来，我们将使用`enterTimestamp`、`enterLevel,`和`enterMessage`来设置适当的`LogEntry` 属性:

```java
 @Override
    public void enterTimestamp(LogParser.TimestampContext ctx) {
        this.current.setTimestamp(
          LocalDateTime.parse(ctx.getText(), DEFAULT_DATETIME_FORMATTER));
    }

    @Override
    public void enterMessage(LogParser.MessageContext ctx) {
        this.current.setMessage(ctx.getText());
    }

    @Override
    public void enterLevel(LogParser.LevelContext ctx) {
        this.current.setLevel(LogLevel.valueOf(ctx.getText()));
    }
```

最后，让我们使用`exitEntry `方法来创建和添加新的`LogEntry`:

```java
 @Override
    public void exitLogEntry(LogParser.EntryContext ctx) {
        this.entries.add(this.current);
    }
```

**顺便说一下，注意我们的`LogListener `不是线程安全的！**

### 6.4。测试

现在我们可以像上次一样再次测试:

```java
@Test
public void whenLogContainsOneErrorLogEntry_thenOneErrorIsReturned()
  throws Exception {

    String logLine ="2018-May-05 14:20:24 ERROR Bad thing happened";

    // instantiate the lexer, the parser, and the walker
    LogListener listener = new LogListener();
    walker.walk(listener, logParser.log());
    LogEntry entry = listener.getEntries().get(0);

    assertThat(entry.getLevel(), is(LogLevel.ERROR));
    assertThat(entry.getMessage(), is("Bad thing happened"));
    assertThat(entry.getTimestamp(), is(LocalDateTime.of(2018,5,5,14,20,24)));
}
```

## 7。结论

在本文中，我们关注如何使用 ANTLR 为自己的语言创建定制的解析器。

我们还看到了如何使用现有的语法文件，并将它们应用于非常简单的任务，如林挺代码。

和往常一样，这里使用的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220625235751/https://github.com/eugenp/tutorials/tree/master/antlr)