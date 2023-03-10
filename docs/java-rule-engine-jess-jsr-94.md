# 杰斯规则引擎和 JSR 94

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-rule-engine-jess-jsr-94>

## 1.概观

使用规则引擎是将业务逻辑从样板代码中分离出来并保护我们的应用程序代码不受业务变化影响的一个好方法。

在之前关于 Java 规则引擎的文章中，我们提到了 JSR 94 规范。**作为 JSR 94** 的参考规则驱动实现，Jess 规则引擎对 **尤为重要，所以让我们来看看。**

## 2.Jess 规则引擎

[Jess](https://web.archive.org/web/20220628131443/http://alvarestech.com/temp/fuzzyjess/Jess60/Jess70b7/docs/index.html) 是最早与 Java 轻松集成的规则引擎之一。Jess 使用了高效的 [Rete 算法](https://web.archive.org/web/20220628131443/https://en.wikipedia.org/wiki/Rete_algorithm)的增强实现，使其在大多数情况下比简单的 Java 循环快得多。

**规则可以从用本地 Jess 规则语言(一种扩展的基于 Lisp 的语法)编写的规则集中执行，**或者从更详细的 XML 格式中执行。我们将使用本地格式。

有一个用于开发的[基于 Eclipse 的 IDE](https://web.archive.org/web/20220628131443/http://alvarestech.com/temp/fuzzyjess/Jess60/Jess70b7/docs/release_notes.html) (用于 Eclipse 的旧版本)和一些关于使用和集成 Jess 与 Java 的优秀的[文档](https://web.archive.org/web/20220628131443/http://alvarestech.com/temp/fuzzyjess/Jess60/Jess70b7/docs/basics.html)。甚至有一个 REPL 命令行界面，我们可以在创建规则文件之前尝试我们的想法。

作为 JSR 94 的参考规则引擎，Jess 从定义上来说是符合 JSR 94 的，尽管它已经不再处于开发阶段。

### 2.1.简单介绍一下 JSR 94

JSR 94 提供了一个 API，我们可以用它来独立于我们选择的规则引擎。我们可以将任何符合 JSR 94 的规则引擎插入到我们的代码中，并运行一些规则，而无需改变我们与应用程序中的规则引擎的交互方式。

这并不意味着规则引擎的底层规则看起来是一样的——如果我们改变规则引擎，我们可能必须重写这些规则，但这确实意味着我们不需要重写我们的应用程序的一部分来使用新的规则引擎。我们需要的唯一代码更改是更新驱动程序的名称和一些规则文件名。

### 2.2.杰斯·JSR 94 年的车手

尽管 JSR 94 包含了 Jess 的参考规则引擎`driver` ,但 Jess 本身并不包含在内，因为它是一个许可的商业产品。参考驱动程序在`org.jcp.jsr94.jess`包中，但是当我们[下载杰斯](https://web.archive.org/web/20220628131443/http://alvarestech.com/temp/fuzzyjess/Jess60/Jess70b7/docs/basics.html)时，一个更新的驱动程序在`jess.jsr94`包中可用。

在继续了解 JSR 94 层如何改变这一点之前，让我们先来看看 Jess 的原生 Java 集成。

## 3.提供示例

在开始将 Jess 集成到我们的代码中之前，让我们确保已经下载了它，并在我们的类路径中提供了它。我们需要注册 30 天免费试用下载，除非我们已经有许可证。

所以，让我们[下载 Jess](https://web.archive.org/web/20220628131443/https://jess.sandia.gov/jess/download.shtml) ，解包下载的`Jess71p2.jar`，并运行它的一个例子，以确保我们有一个工作版本。

### 3.1.独立 Jess

让我们看看`Jess71p2/examples`目录，其中`jess`目录保存了一些示例规则集。`pricing_engine`目录显示了一个可以通过 [ant](/web/20220628131443/https://www.baeldung.com/ant-maven-gradle) `build.xml` 脚本执行的集成。让我们将目录更改为定价引擎示例，并通过`ant test`运行程序:

```java
cd Jess71p2/examples/pricing_engine
ant test
```

这将构建并运行一个示例定价规则集:

```java
Buildfile: Jess71p2\examples\pricing_engine\build.xml
...
test:
[java] Items for order 123:
[java] 1 CD Writer: 199.99
...
[java] Items for order 666:
[java] 1 Incredibles DVD: 29.99
[java] Offers for order 666:
[java] BUILD SUCCESSFUL
Total time: 1 second
```

### 3.2.杰斯和 JSR 94

现在我们已经让 Jess 工作了，让我们[下载 JSR 94](https://web.archive.org/web/20220628131443/https://jcp.org/aboutJava/communityprocess/final/jsr094/index.html) ，然后解压缩它以创建一个 jsr94-1.0 目录，其中包含 ant、doc、lib 和 src 目录。

```java
unzip jreng-1_0a-fr-spec-api.zip
```

这为我们提供了 JSR 94 API 和 Jess 参考驱动程序，但它没有附带许可的 Jess 实现，所以如果我们现在尝试运行一个示例，我们会得到以下错误:

```java
Error: The reference implementation Jess could not be found.
```

因此，让我们添加 Jess 参考实现`jess.jar`，它是我们之前下载的 Jess71p2 的一部分，并将其复制到 JSR 94 lib 目录，然后运行示例:

```java
cp Jess71p2/lib/jess.jar jsr94-1.0/lib/
java -jar jsr94-1.0/lib/jsr94-example.jar
```

该示例运行一些规则来确定客户在支付发票时的剩余信用:

```java
Administration API Acquired RuleAdministrator: [[email protected]](/web/20220628131443/https://www.baeldung.com/cdn-cgi/l/email-protection)
...
Runtime API Acquired RuleRuntime: [[email protected]](/web/20220628131443/https://www.baeldung.com/cdn-cgi/l/email-protection)
Customer credit limit result: 3000
...
Invoice 2 amount: 1750 status: paid
Released Stateful Rule Session.
```

## 4.将 Jess 与 Java 集成

现在，我们已经下载了杰斯和 JSR 94，并通过 JSR 本地运行了一些规则，让我们看看如何将杰斯规则集集成到 Java 程序中。

在我们的例子中，我们将从 Java 代码中执行一个简单的 Jess 规则文件`hellojess.clp,`开始，然后查看另一个规则文件`bonus.clp`，它将使用和修改我们的一些对象。

### 4.1.Maven 依赖性

Jess 没有可用的 Maven 依赖项，所以如果我们还没有这样做，让我们下载并解压 Jess jar ( `jess.jar`)和`[mvn install](https://web.archive.org/web/20220628131443/https://maven.apache.org/guides/mini/guide-3rd-party-jars-local.html)`到我们的本地 Maven 存储库:

```java
mvn install:install-file -Dfile=jess.jar -DgroupId=gov.sandia -DartifactId=jess -Dversion=7.1p2 -Dpackaging=jar -DgeneratePom=true
```

然后，我们可以以通常的方式将它添加为依赖项:

```java
<dependency>
    <groupId>gov.sandia</groupId>
    <artifactId>jess</artifactId>
    <version>7.1p2</version>
</dependency>
```

### 4.2.你好杰斯规则文件

接下来，让我们创建最简单的规则文件来打印消息。我们将规则文件保存为`hellojess.clp`:

```java
(printout t "Hello from Jess!" crlf)
```

### 4.3.Jess 规则引擎

现在，让我们创建一个 Jess `Rete` 规则引擎的实例，`reset()`将其初始化，在`hellojess.clp`中加载规则，并运行它们:

```java
public class HelloJess {
    public static void main(String[] args) throws JessException {
    Rete engine = new Rete();
    engine.reset();
    engine.batch("hellojess.clp");
    engine.run();
}
```

对于这个简单的例子，我们刚刚将潜在的`JessException`添加到我们的`main` 方法的`throws`子句中。

当我们运行程序时，我们将看到输出:

```java
Hello from Jess!
```

## 5.用数据将 Jess 集成到 Java

现在一切都已正确安装，我们可以运行规则了，**让我们看看如何添加数据供规则引擎处理，以及如何检索结果**。

首先，我们需要一些 Java 类，然后是一个使用它们的新规则集。

### 5.1.模型

让我们创建一些简单的`Question`和`Answer`类:

```java
public class Question {
    private String question;
    private int balance;
    <i><span style="font-weight: 400">// getters and setters</span></i>

    public Question(String question, int balance) {
        this.question = question;
        this.balance = balance;
    }
}

public class Answer {
    private String answer;
    private int newBalance;
    <i><span style="font-weight: 400">// getters and setters</span></i>

    public Answer(String answer, int newBalance) {
        this.answer = answer;
        this.newBalance = newBalance;
    }
}
```

### 5.2。有输入和输出的杰斯法则

现在，让我们创建一个简单的名为`bonus.clp`的 Jess 规则集，我们将向其传递一个`Question`并从其接收一个`Answer`。

首先，我们`import`我们的`Question`和`Answer`类，然后使用 Jess 的`deftemplate`函数使它们对规则引擎可用:

```java
(import com.baeldung.rules.jsr94.jess.model.*)
(deftemplate Question     (declare (from-class Question)))
(deftemplate Answer       (declare (from-class Answer)))
```

注意圆括号的使用，它表示 Jess 函数调用。

现在，让我们使用`defrule`在 Jess 的扩展 Lisp 格式中添加一个规则`avoid-overdraft`,如果我们的`Question`中的余额低于零，它会给我们 50 美元的奖金:

```java
(defrule avoid-overdraft "Give $50 to anyone overdrawn"
    ?q <- (Question { balance < 0 })
    =>
    (add (new Answer "Overdrawn bonus" (+ ?q.balance 50))))
```

这里，当“`<-“`右边的条件匹配时，“`?”`将一个对象绑定到一个变量`q`。在这种情况下，当规则引擎发现一个`Question`的`balance`小于 0 时。

当它这样做时，那么"`=>”`右边的动作被触发，因此引擎`add`是工作存储器的一个`new Answer`对象。我们给它两个必需的构造函数参数:参数`answer`的“透支奖金”和计算参数`newAmount` 的`(+)`函数。

### 5.3.使用 Jess 规则引擎操作数据

我们可以使用`add()`一次向规则引擎的工作内存添加一个对象，或者使用`addAll()`添加一组数据。让我们用`add()`来增加一个问题:

```java
Question question = new Question("Can I have a bonus?", -5);
engine.add(data);
```

有了所有的数据，让我们执行我们的规则:

```java
engine.run();
```

Jess `Rete`引擎将发挥其魔力，并在所有相关规则执行完毕后返回。在我们的例子中，我们将有一个`Answer`来检查。

让我们使用一个`jess.Filter`将规则引擎中的`Answer`提取到一个`Iterable`结果对象`:`

```java
Iterator results = engine.getObjects(new jess.Filter.ByClass(Answer.class));
while (results.hasNext()) {
    Answer answer = (Answer) results.next();
    // process our Answer
}
```

在我们的简单示例中，我们没有任何引用数据，但是当我们有引用数据时，我们可以使用一个`WorkingMemoryMarker`和`engine.mark()`来标记添加数据后规则引擎的工作内存的状态。然后我们可以叫`engine`。`resetToMark`使用我们的标记将工作内存重置为“已加载”状态，并针对不同的对象集高效地重用规则引擎:

```java
WorkingMemoryMarker marker;
// load reference data
marker = engine.mark();
// load specific data and run rules
engine.resetToMark(marker);
```

现在，让我们看看如何使用 JSR 94 运行相同的规则集。

## 6.使用 JSR 94 集成 Jess 规则引擎

JSR 94 标准化了我们的代码如何与规则引擎交互。如果出现更好的替代方案，这使得在不显著改变应用程序的情况下改变我们的规则引擎变得更加容易。

JSR 94 API 有两个主要的包:

*   `javax.rules.admin`–用于加载驱动程序和规则
*   `javax.rules`–运行规则并提取结果

我们将看看如何使用这两个中的类。

### 6.1.Maven 依赖性

首先，让我们为`jsr94`添加一个 Maven 依赖项:

```java
<dependency>
    <groupId>jsr94</groupId>
    <artifactId>jsr94</artifactId>
    <version>1.1</version>
</dependency>
```

### 6.2.管理 API

要开始使用 JSR 94，我们需要实例化一个`RuleServiceProvider`。让我们创建一个，传递给我们的 Jess 规则驱动程序:

```java
String RULE_SERVICE_PROVIDER="jess.jsr94";
Class.forName(RULE_SERVICE_PROVIDER + ".RuleServiceProviderImpl");
RuleServiceProvider ruleServiceProvider = RuleServiceProviderManager.getRuleServiceProvider(RULE_SERVICE_PROVIDER);
```

现在，让我们获取 Jess 的 JSR 94 `RuleAdministrator`，将我们的示例规则集加载到 JSR 94 `RuleExecutionSet,`中，并注册它以便用我们选择的 URI 执行:

```java
RuleAdministrator ruleAdministrator = serviceProvider.getRuleAdministrator();

InputStream ruleInput = JessRunner.class.getResourceAsStream(rulesFile);
HashMap vendorProperties = new HashMap();

RuleExecutionSet ruleExecutionSet = ruleAdministrator
  .getLocalRuleExecutionSetProvider(vendorProperties)
  .createRuleExecutionSet(ruleInput, vendorProperties);

String rulesURI = "rules://com/baeldung/rules/bonus";
ruleAdministrator.registerRuleExecutionSet(rulesURI, ruleExecutionSet, vendorProperties);
```

Jess 驱动不需要我们提供给`RuleAdministrator`的`vendorProperties`地图，但它是接口的一部分，其他厂商可能需要。

现在，我们的规则引擎提供者 Jess 已经初始化，我们的规则集也已经注册，我们几乎可以运行我们的规则了。

在运行它们之前，我们需要一个运行时实例和一个会话来运行它们。让我们也添加一个占位符，`calculateResults(),`来表示魔法将在哪里发生，并释放会话:

```java
RuleRuntime ruleRuntime = ruleServiceProvider.getRuleRuntime();
StatelessRuleSession statelessRuleSession
  = (StatelessRuleSession) ruleRuntime.createRuleSession(rulesURI, new HashMap(), RuleRuntime.STATELESS_SESSION_TYPE);
calculateResults(statelessRuleSession);
statelessRuleSession.release();
```

### 6.3.执行 API

现在一切就绪，让我们实现`calculateResults`来提供初始数据，在无状态会话中执行我们的规则，并提取结果:

```java
List data = new ArrayList();
data.add(new Question("Can I have a bonus?", -5));
List results = statelessRuleSession.executeRules(data);
```

因为 JSR 94 是在 JDK 5 出现之前写的，API 不使用泛型，所以让我们只使用一个`Iterator`来看看结果:

```java
Iterator itr = results.iterator();
while (itr.hasNext()) {
    Object obj = itr.next();
    if (obj instanceof Answer) {
        int answerBalance = ((Answer) obj).getCalculatedBalance());
    }
}
```

我们在示例中使用了无状态会话，但是如果我们想在调用之间维护状态，我们也可以创建一个`StatefuleRuleSession`。

## 7.结论

在本文中，我们学习了如何通过使用 Jess 的本地类将 Jess 规则引擎集成到我们的应用程序中，并通过使用 JSR 94 做了一些努力。我们已经看到了如何将业务规则分成单独的文件，当我们的应用程序运行时，规则引擎会处理这些文件。

如果我们有为另一个 JSR 94 兼容规则引擎编写的相同业务逻辑的规则，那么我们可以简单地添加替代规则引擎的驱动程序，并更新我们的应用程序应该使用的驱动程序名称，并且不需要进一步的代码更改。

在 jess.sandia.gov 有更多关于在 Java 应用程序中嵌入 Jess 的细节，Oracle 有一个有用的指南帮助 T2 开始使用 Java 规则引擎 API(JSR 94)T3。

像往常一样，我们在本文中看到的代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628131443/https://github.com/eugenp/tutorials/tree/master/rule-engines/jess)