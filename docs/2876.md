# 系统存根库指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-system-stubs>

## 1.概观

当我们的软件依赖于系统资源(如环境变量、系统属性)或使用进程级操作(如`System.exit`)时，测试它会很困难。

Java 没有提供设置环境变量的直接方法，我们冒着在一个测试中设置的值影响另一个测试的执行的风险。类似地，我们可能会发现自己避免为可能执行`System.exit`的代码编写 JUnit 测试，因为有可能会中止测试。

[系统规则和系统 Lambda 库](/web/20220627090235/https://www.baeldung.com/java-system-rules-junit)是这些问题的早期解决方案。在本教程中，我们将看到一个新的系统 Lambda 分支，名为 **[系统存根](https://web.archive.org/web/20220627090235/https://github.com/webcompere/system-stubs)，它提供了一个 [JUnit 5](/web/20220627090235/https://www.baeldung.com/tag/junit-5/) 替代。**

## 2.为什么是系统存根？

### 2.1.System Lambda 不是 JUnit 插件

最初的系统规则库只适用于 JUnit 4。在 JUnit 5 下，它仍然可以与 JUnit Vintage 一起使用，但是这需要继续创建 JUnit 4 测试。这个库的创建者开发了一个测试框架不可知的版本，叫做[系统λ](/web/20220627090235/https://www.baeldung.com/java-system-rules-junit)，它是为了在每个测试方法中使用:

```
@Test
void aSingleSystemLambda() throws Exception {
    restoreSystemProperties(() -> {
        System.setProperty("log_dir", "test/resources");
        assertEquals("test/resources", System.getProperty("log_dir"));
    });

    // more test code here
}
```

测试代码被表示为一个 lambda，传递给一个设置必要存根的方法。清理发生在控制返回到测试方法的其余部分之前。

虽然这在某些情况下很有效，但是这种方法也有一些缺点。

### 2.2.避免额外代码

System Lambda 方法的好处是，在其工厂类中有一些用于执行特定类型测试的通用方法。然而，当我们想要在许多测试用例中使用它时，这会导致一些代码膨胀。

首先，即使测试代码本身没有抛出一个检查过的异常，包装器方法会抛出，所以所有方法都会获得一个`throws Exception`。其次，在多个测试中设置相同的规则需要代码复制。每个测试都需要独立执行相同的配置。

然而，当我们试图一次设置多个工具时，这种方法最麻烦的地方就来了。假设我们想要设置一些环境变量和系统属性。在测试代码开始之前，我们最终需要两级嵌套:

```
@Test
void multipleSystemLambdas() throws Exception {
    restoreSystemProperties(() -> {
        withEnvironmentVariable("URL", "https://www.baeldung.com")
            .execute(() -> {
                System.setProperty("log_dir", "test/resources");
                assertEquals("test/resources", System.getProperty("log_dir"));
                assertEquals("https://www.baeldung.com", System.getenv("URL"));
            });
    });
}
```

这就是 JUnit 插件或扩展可以帮助我们减少测试中所需代码量的地方。

### 2.3.使用较少的样板文件

我们应该期望能够用最少的样板文件编写我们的测试:

```
@SystemStub
private EnvironmentVariables environmentVariables = ...;

@SystemStub
private SystemProperties restoreSystemProperties;

@Test
void multipleSystemStubs() {
    System.setProperty("log_dir", "test/resources");
    assertEquals("test/resources", System.getProperty("log_dir"));
    assertEquals("https://www.baeldung.com", System.getenv("ADDRESS"));
}
```

这种方法是由 JUnit 5 扩展提供的，它允许我们用更少的代码编写测试。

### 2.4.测试生命周期挂钩

当唯一可用的工具是[执行循环模式](https://web.archive.org/web/20220627090235/https://java-design-patterns.com/patterns/execute-around/)时，不可能将存根行为与测试生命周期的所有部分挂钩。当试图将它与其他 JUnit 扩展如`@SpringBootTest`结合起来时，这尤其具有挑战性。

如果我们想在 Spring Boot 测试中设置一些环境变量，那么我们不可能合理地将整个测试生态系统嵌入到一个测试方法中。我们需要一种方法来激活测试套件周围的测试设置。

这在 System Lambda 使用的方法中是不可能的，这也是创建系统存根的主要原因之一。

### 2.5.鼓励动态属性

其他用于设置系统属性的框架，比如 [JUnit Pioneer](https://web.archive.org/web/20220627090235/https://junit-pioneer.org/) ，强调编译时已知的配置。在现代测试中，我们可能会使用 [Testcontainers](/web/20220627090235/https://www.baeldung.com/docker-test-containers) 或 [Wiremock](/web/20220627090235/https://www.baeldung.com/introduction-to-wiremock) ，我们需要在这些工具启动后基于随机运行时设置来设置我们的系统属性。这最适合可以在整个测试生命周期中使用的测试库。

### 2.6.更多可配置性

拥有现成的测试方法是有益的，比如`catchSystemExit`，它包装测试代码来完成一项工作。然而，这依赖于测试库开发者来提供我们可能需要的配置选项的每一种变化。

通过组合进行配置更加灵活，并且是新系统存根实现的一大部分。

然而，**系统存根支持来自系统 Lambda** 的原始测试结构，以实现向后兼容性。此外，它还提供了一个新的 JUnit 5 扩展、一组 JUnit 4 规则和更多的配置选项。虽然基于原始代码，但它经过了大量的重构和模块化，以提供更丰富的功能。

下面我们来详细了解一下。

## 3.入门指南

### 3.1.属国

JUnit 5 扩展需要一个合理的最新版本的 [JUnit 5](https://web.archive.org/web/20220627090235/https://search.maven.org/artifact/org.junit.jupiter/junit-jupiter) :

```
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency> 
```

让我们将所有的[系统存根](https://web.archive.org/web/20220627090235/https://search.maven.org/search?q=system-stubs&g=uk.org.webcompere)库依赖项添加到我们的`pom.xml`:

```
<!-- for testing with only lambda pattern -->
<dependency>
    <groupId>uk.org.webcompere</groupId>
    <artifactId>system-stubs-core</artifactId>
    <version>1.1.0</version>
    <scope>test</scope>
</dependency>

<!-- for JUnit 4 -->
<dependency>
    <groupId>uk.org.webcompere</groupId>
    <artifactId>system-stubs-junit4</artifactId>
    <version>1.1.0</version>
    <scope>test</scope>
</dependency>

<!-- for JUnit 5 -->
<dependency>
    <groupId>uk.org.webcompere</groupId>
    <artifactId>system-stubs-jupiter</artifactId>
    <version>1.1.0</version>
    <scope>test</scope>
</dependency> 
```

我们应该注意，我们只需要导入我们正在使用的测试框架所需要的数量。事实上，后两者都包含了核心依赖性。

现在让我们编写第一个测试。

### 3.2.JUnit 4 环境变量

我们可以通过在类型为`EnvironmentVariablesRule`的测试类中声明一个 JUnit 4 `@Rule`注释字段来控制环境变量。这将在我们的测试运行时被 JUnit 4 激活，并允许我们在测试中设置环境变量:

```
@Rule
public EnvironmentVariablesRule environmentVariablesRule = new EnvironmentVariablesRule();

@Test
public void givenEnvironmentCanBeModified_whenSetEnvironment_thenItIsSet() {
    environmentVariablesRule.set("ENV", "value1");

    assertThat(System.getenv("ENV")).isEqualTo("value1");
}
```

在实践中，我们可能更喜欢在一个`@Before`方法中设置环境变量值，这样设置可以在所有测试中共享:

```
@Before
public void before() {
    environmentVariablesRule.set("ENV", "value1")
      .set("ENV2", "value2");
}
```

**这里我们要注意的是使用了流畅的`set`方法**，使得通过`method chaining`设置多个值变得很容易。

我们还可以使用`EnvironmentVariablesRule`对象的构造函数来提供构造值:

```
@Rule
public EnvironmentVariablesRule environmentVariablesRule =
  new EnvironmentVariablesRule("ENV", "value1",
    "ENV2", "value2");
```

构造函数有几种重载，允许以不同的形式提供变量。上例中的例子允许使用 [`varargs`](/web/20220627090235/https://www.baeldung.com/java-varargs) 提供任意数量的名称-值对。

每个系统存根 JUnit 4 规则都是一个核心存根对象的子类。通过在`static`字段上添加 [`@ClassRule`注释](/web/20220627090235/https://www.baeldung.com/junit-4-rules)，它们也可以在整个测试类的生命周期中使用，这将导致它们在第一次测试之前被激活，然后在最后一次测试之后被清除。

### 3.3.JUnit 5 环境变量

在 JUnit 5 测试中使用系统存根对象之前，我们必须将扩展添加到我们的测试类中:

```
@ExtendWith(SystemStubsExtension.class)
class EnvironmentVariablesJUnit5 {
    // tests
}
```

然后我们可以在测试类中为 JUnit 5 创建一个字段来管理我们。我们用`@SystemStub`对此进行注释，以便扩展知道激活它:

```
@SystemStub
private EnvironmentVariables environmentVariables;
```

该扩展将只管理标有`@SystemStub`的对象，如果我们愿意，这允许我们在测试中手动使用其他系统存根对象。

这里，我们没有提供存根对象的任何构造。扩展为我们构建了一个，就像 [Mockito 扩展](/web/20220627090235/https://www.baeldung.com/mockito-junit-5-extension)构建 mocks 一样。

我们现在可以使用对象来帮助我们在一个测试中设置环境变量:

```
@Test
void givenEnvironmentCanBeModified_whenSetEnvironment_thenItIsSet() {
    environmentVariables.set("ENV", "value1");

    assertThat(System.getenv("ENV")).isEqualTo("value1");
}
```

如果我们想从测试方法之外提供适用于所有测试的环境变量，我们可以在一个`@BeforeEach`方法内这样做，或者可以使用`EnvironmentVariables`的构造函数来设置我们的值:

```
@SystemStub
private EnvironmentVariables environmentVariables =
  new EnvironmentVariables("ENV", "value1");
```

和`EnvironmentVariablesRule`一样，构造函数有几种重载，允许我们以多种方式设置想要的变量。如果我们愿意，我们也可以流畅地使用`set`方法来设置值:

```
@SystemStub
private EnvironmentVariables environmentVariables =
  new EnvironmentVariables()
    .set("ENV", "value1")
    .set("ENV2", "value2");
```

我们还可以将我们的字段`static`作为`@BeforeAll` / `@AfterAll`生命周期的一部分进行管理。

### 3.4.JUnit 5 参数注入

虽然在所有测试中使用存根对象时，将它们放在字段中是有用的，但我们可能更喜欢仅在选定的测试中使用它们。这可以通过 JUnit 5 参数注入来实现:

```
@Test
void givenEnvironmentCanBeModified(EnvironmentVariables environmentVariables) {
    environmentVariables.set("ENV", "value1");

    assertThat(System.getenv("ENV")).isEqualTo("value1");
}
```

在这种情况下，`EnvironmentVariables`对象是用它的默认构造函数为我们构造的，允许我们在一次测试中使用它。该对象也被激活，以便它在运行时环境中运行。测试结束后会整理好的。

所有的系统存根对象都有一个默认的构造函数，并且能够在运行时重新配置。我们可以根据需要在测试中注入任意多的量。

### 3.5.执行环境变量

用于创建存根的原始 System Lambda facade 方法也可以通过`SystemStubs` 类获得。在内部，它们通过创建存根对象的实例来实现。有时从配方返回的对象是存根对象，供进一步配置和使用:

```
withEnvironmentVariable("ENV3", "val")
    .execute(() -> {
        assertThat(System.getenv("ENV3")).isEqualTo("val");
    });
```

在幕后，`withEnvironmentVariable`正在做着相当于:

```
return new EnvironmentVariables().set("ENV3", "val");
```

**`execute`方法是所有`SystemStub`对象共有的。**它设置由对象定义的存根，然后执行传入的 lambda。之后，它整理并将控制返回给周围的测试。

如果测试代码返回一个值，那么该值可以由`execute`返回:

```
String extracted = new EnvironmentVariables("PROXY", "none")
  .execute(() -> System.getenv("PROXY"));

assertThat(extracted).isEqualTo("none");
```

当我们测试的代码需要访问环境设置来构建某些东西时，这可能是有用的。它通常在测试 AWS Lambda 处理程序时使用，这些处理程序通常通过环境变量进行配置。

这种模式对于偶尔测试的好处是，我们必须只在需要的地方显式地设置存根。因此，它可以更精确和可见。然而，它不允许我们在测试之间共享设置，并且可能更冗长。

### 3.6.多个系统存根

我们已经看到了 JUnit 4 和 JUnit 5 插件如何为我们构造和激活存根对象。如果有多个存根，它们由框架代码适当地设置和删除。

然而，当我们为执行环绕模式构造存根对象时，我们需要测试代码在它们内部运行。

这可以通过使用`with` / `execute`方法来实现。这些通过使用单个`execute`从多个存根对象创建一个组合来工作:

```
with(new EnvironmentVariables("FOO", "bar"), new SystemProperties("prop", "val"))
  .execute(() -> {
      assertThat(System.getenv("FOO")).isEqualTo("bar");
      assertThat(System.getProperty("prop")).isEqualTo("val");
  });
```

现在我们已经看到了使用系统存根对象的一般形式，无论有没有 JUnit 框架支持，让我们看看库的其他功能。

## 4.系统属性

我们可以用 Java 随时调用`System.setProperty`。然而，这有将设置从一个测试泄露到另一个测试的风险。`SystemProperties` stubbing 的主要目的是在测试完成后将系统属性恢复到原始设置。然而，在测试开始之前，公共设置代码定义应该使用哪些系统属性也是有用的。

### 4.1.JUnit 4 系统属性

通过将规则添加到 JUnit 4 测试类中，我们可以将每个测试与其他测试方法中的任何`System.setProperty`调用隔离开来。我们还可以通过构造函数提供一些前期属性:

```
@Rule
public SystemPropertiesRule systemProperties =
  new SystemPropertiesRule("db.connection", "false");
```

使用这个对象，我们还可以在 JUnit `@Before`方法中设置一些额外的属性:

```
@Before
public void before() {
    systemProperties.set("before.prop", "before");
}
```

如果我们愿意，我们也可以在测试体中使用`set`方法或者使用`System.setProperty`。我们必须只在创建`SystemPropertiesRule`或`@Before`方法时使用`set`，因为它将设置存储在规则中，以备后用。

### 4.2.JUnit 5 系统属性

我们有两个使用`SystemProperties`对象的主要用例。我们可能希望在每个测试用例之后重置系统属性，或者我们可能希望在一个中心位置准备一些公共的系统属性供每个测试用例使用。

恢复系统属性需要我们向测试类添加 JUnit 5 扩展和一个`SystemProperties`字段:

```
@ExtendWith(SystemStubsExtension.class)
class RestoreSystemProperties {
    @SystemStub
    private SystemProperties systemProperties;

}
```

现在，每个测试都将清除它所更改的任何系统属性。

我们也可以通过参数注入对选定的测试执行此操作:

```
@Test
void willRestorePropertiesAfter(SystemProperties systemProperties) {

} 
```

如果我们希望在测试中设置属性，那么我们可以在构造我们的`SystemProperties `对象时分配这些属性，或者使用一个`@BeforeEach`方法:

```
@ExtendWith(SystemStubsExtension.class)
class SetSomeSystemProperties {
    @SystemStub
    private SystemProperties systemProperties;

    @BeforeEach
    void before() {
        systemProperties.set("beforeProperty", "before");
    }
}
```

同样，我们注意到 JUnit 5 测试需要用`@ExtendWith(SystemStubsExtension.class).` 进行注释。如果我们不在初始化列表中提供`new`语句，扩展将创建系统存根对象。

### 4.3.执行时的系统属性

`SystemStubs`类提供了一个`restoreSystemProperties`方法，允许我们运行恢复了属性的测试代码:

```
restoreSystemProperties(() -> {
    // test code
    System.setProperty("unrestored", "true");
});

assertThat(System.getProperty("unrestored")).isNull();
```

这需要一个不返回任何值的 lambda。如果我们希望使用一个公共的设置函数来创建属性，从测试方法中获得一个返回值，或者通过`with` / `execute`将`SystemProperties`与其他存根结合起来，那么我们可以显式地创建对象:

```
String result = new SystemProperties()
  .execute(() -> {
      System.setProperty("unrestored", "true");
      return "it works";
  });

assertThat(result).isEqualTo("it works");
assertThat(System.getProperty("unrestored")).isNull();
```

### 4.4.文件中的属性

`SystemProperties`和`EnvironmentVariables`对象都可以由`Map`构造。这允许 Java 的`Properties`对象作为系统属性或环境变量的源提供。

在`PropertySource`类中有 helper 方法，用于从文件或资源中加载 Java 属性。这些属性文件是名称/值对:

```
name=baeldung
version=1.0
```

我们可以通过使用`fromResource`函数从资源`test.properties`加载:

```
SystemProperties systemProperties =
  new SystemProperties(PropertySource.fromResource("test.properties")); 
```

对于其他来源，如`fromFile`或`fromInputStream`，在`PropertySource`中也有类似的方便方法。

## 5.系统输出和系统错误

当我们的应用程序写入`System.out,`时，可能很难测试。这有时通过使用接口作为输出的目标并在测试时模仿它来解决:

```
interface LogOutput {
   void write(String line);
}

class Component {
    private LogOutput log;

    public void method() {
        log.write("Some output");
    }
}
```

像这样的技术对`Mockito`模仿很有效，但是如果**我们可以直接困住`System.out`本身，就没有必要了。**

### 5.1.JUnit 4 `SystemOutRule`和`SystemErrRule`

为了在 JUnit 4 测试中将输出捕获到`System.out`，我们添加了`SystemOutRule`:

```
@Rule
public SystemOutRule systemOutRule = new SystemOutRule();
```

之后，可以在测试中读取`System.out`的任何输出:

```
System.out.println("line1");
System.out.println("line2");

assertThat(systemOutRule.getLines())
  .containsExactly("line1", "line2");
```

我们有多种文本格式可供选择。上面的例子使用了`getLines`提供的`Stream<String>`。我们也可以选择获取整个文本块:

```
assertThat(systemOutRule.getText())
  .startsWith("line1");
```

但是，我们应该注意，这个文本将有换行符，这些换行符因平台而异。通过使用规范化形式，我们可以在每个平台上用`\n`替换换行符:

```
assertThat(systemOutRule.getLinesNormalized())
  .isEqualTo("line1\nline2\n");
```

`SystemErrRule`的工作方式与`System.err`的`System.out`相同:

```
@Rule
public SystemErrRule systemErrRule = new SystemErrRule();

@Test
public void whenCodeWritesToSystemErr_itCanBeRead() {
    System.err.println("line1");
    System.err.println("line2");

    assertThat(systemErrRule.getLines())
      .containsExactly("line1", "line2");
}
```

还有一个`SystemErrAndOutRule`类，它将`System.out`和`System.err`同时接入一个缓冲区。

### 5.2.JUnit 5 示例

与其他系统存根对象一样，我们只需要声明一个类型为`SystemOut`或`SystemErr`的字段或参数。这将为我们提供输出的捕获:

```
@SystemStub
private SystemOut systemOut;

@SystemStub
private SystemErr systemErr;

@Test
void whenWriteToOutput_thenItCanBeAsserted() {
    System.out.println("to out");
    System.err.println("to err");

    assertThat(systemOut.getLines()).containsExactly("to out");
    assertThat(systemErr.getLines()).containsExactly("to err");
}
```

我们也可以使用`SystemErrAndOut`类将两组输出导入同一个缓冲区。

### 5.3.执行循环示例

`SystemStubs` facade 提供了一些函数来获取输出并将其作为`String`返回:

```
@Test
void givenTapOutput_thenGetOutput() throws Exception {
    String output = tapSystemOutNormalized(() -> {
        System.out.println("a");
        System.out.println("b");
    });

    assertThat(output).isEqualTo("a\nb\n");
}
```

我们应该注意到，这些方法不像原始对象本身那样提供丰富的接口。输出的捕获不容易与其他存根结合，比如设置环境变量。

但是，可以直接使用`SystemOut`、 `SystemErr,`和`SystemErrAndOut `对象。例如，我们可以将它们与一些`SystemProperties`结合起来:

```
SystemOut systemOut = new SystemOut();
SystemProperties systemProperties = new SystemProperties("a", "!");
with(systemOut, systemProperties)
  .execute(()  -> {
    System.out.println("a: " + System.getProperty("a"));
});

assertThat(systemOut.getLines()).containsExactly("a: !");
```

### 5.4.叛变

有时我们的目的不是捕获输出，而是防止它扰乱我们的测试运行日志。我们可以使用`muteSystemOut`或`muteSystemErr`函数来实现这一点:

```
muteSystemOut(() -> {
    System.out.println("nothing is output");
});
```

我们可以通过 JUnit 4 `SystemOutRule`在所有测试中实现同样的事情:

```
@Rule
public SystemOutRule systemOutRule = new SystemOutRule(new NoopStream());
```

在 JUnit 5 中，我们可以使用相同的技术:

```
@SystemStub
private SystemOut systemOut = new SystemOut(new NoopStream());
```

### 5.5.用户化

正如我们所看到的，截取输出有几种不同的方式。它们都共享库中的一个公共基类。为了方便起见，几个助手方法和类型，如`SystemErrAndOut,`帮助做一些常见的事情。然而，库本身很容易定制。

我们可以提供自己的目标来捕获输出，作为`Output`的实现。我们已经看到了第一个例子中使用的`Output`类`TapStream`。`NoopStream`用于静音。我们也有`DisallowWriteStream`,如果有东西写入，它会抛出一个错误:

```
// throws an exception:
new SystemOut(new DisallowWriteStream())
  .execute(() -> System.out.println("boo"));
```

## 6.模拟系统在

我们可能有一个在`stdin`上读取输入的应用程序。测试可能包括将算法提取到一个从任何`InputStream`读取的函数中，然后用一个预先准备好的输入流给它。一般来说，模块化代码更好，所以这是一个很好的模式。

然而，**如果我们只测试核心功能，我们将失去对提供`System.in`** 作为源代码的代码的测试覆盖。

无论如何，构造我们自己的流是不方便的。幸运的是，系统存根有所有这些问题的解决方案。

### 6.1.测试输入流

系统存根提供了一系列的`AltInputStream`类作为从`InputStream` 读取的任何代码的**可选输入:**

```
LinesAltStream testInput = new LinesAltStream("line1", "line2");

Scanner scanner = new Scanner(testInput);
assertThat(scanner.nextLine()).isEqualTo("line1");
```

在这个例子中，我们使用了一个字符串数组来构造`LinesAltStream`，但是**我们可以提供来自`Stream<String>`的输入，这样就可以使用任何来源的文本数据**，而不必一次将它们全部加载到内存中。

### 6.2.JUnit 4 示例

我们可以使用`SystemInRule`在 JUnit 4 测试中提供输入行:

```
@Rule
public SystemInRule systemInRule =
  new SystemInRule("line1", "line2", "line3");
```

然后，测试代码可以从`System.in`读取该输入:

```
@Test
public void givenInput_canReadFirstLine() {
    assertThat(new Scanner(System.in).nextLine())
      .isEqualTo("line1");
}
```

### 6.3.JUnit 5 示例

对于 JUnit 5 测试，我们创建一个`SystemIn`字段:

```
@SystemStub
private SystemIn systemIn = new SystemIn("line1", "line2", "line3");
```

然后我们的测试将使用`System.in`运行，提供这些行作为输入。

### 6.4.执行循环示例

`SystemStubs` facade 提供了作为工厂方法的`withTextFromSystemIn`,它创建了一个`SystemIn`对象，用于它的`execute`方法:

```
withTextFromSystemIn("line1", "line2", "line3")
  .execute(() -> {
      assertThat(new Scanner(System.in).nextLine())
        .isEqualTo("line1");
  });
```

### 6.5.用户化

更多的特性可以在构建时或者在测试中运行时添加到`SystemIn`对象中。

我们可以调用`andExceptionThrownOnInputEnd`，这将导致从`System.in`中读取的内容在文本用尽时抛出异常。这可以模拟对文件的中断读取。

我们也可以通过使用`setInputStream`将输入流设置为来自任何`InputStream`，比如`FileInputStream`。我们还有`LinesAltStream`和`TextAltStream`，它们对输入的文本进行操作。

## 7.嘲讽系统。出口

如前所述，如果我们的代码可以调用`System.exit`，它会导致危险和难以调试的测试错误。我们 stub`System.exit`的一个目的是把一个偶然的调用变成一个可追踪的错误。另一个动机是测试软件的有意退出。

### 7.1.JUnit 4 示例

让我们将`SystemExitRule`添加到一个测试类中，作为防止任何`System.exit`停止 JVM 的安全措施:

```
@Rule
public SystemExitRule systemExitRule = new SystemExitRule();
```

然而，**我们可能也想看看是否使用了正确的退出代码**。为此，我们需要断言代码抛出了`AbortExecutionException`，这是调用了`System.exit`的系统存根信号。

```
@Test
public void whenExit_thenExitCodeIsAvailable() {
    assertThatThrownBy(() -> {
        System.exit(123);
    }).isInstanceOf(AbortExecutionException.class);

    assertThat(systemExitRule.getExitCode()).isEqualTo(123);
}
```

在这个例子中，我们使用了来自`AssertJ`的`assertThatThrownBy`来捕捉和检查异常信号 exit 的发生。然后我们查看来自`SystemExitRule`的`getExitCode`来断言退出代码。

### 7.2.JUnit 5 示例

对于 JUnit 5 测试，我们声明了`@SystemStub`字段:

```
@SystemStub
private SystemExit systemExit;
```

然后我们以与 JUnit 4 中的`SystemExitRule`相同的方式使用`SystemExit`类。鉴于`SystemExitRule`类是`SystemExit`的子类，它们有相同的接口。

### 7.3.执行循环示例

`SystemStubs`类提供内部使用`SystemExit`的`execute`函数的`catchSystemExit,`:

```
int exitCode = catchSystemExit(() -> {
    System.exit(123);
});
assertThat(exitCode).isEqualTo(123);
```

与 JUnit 插件示例相比，这段代码不会抛出异常来指示系统退出。相反，它捕捉错误并记录退出代码。对于 facade 方法，它返回退出代码。

当我们直接使用`execute`方法时，退出被捕获，退出代码设置在`SystemExit`对象内部。然后我们可以调用`getExitCode`来获得退出代码，或者如果没有退出代码，就调用`null`。

## 8.JUnit 5 中的定制测试资源

JUnit 4 已经为创建测试规则提供了一个简单的结构，就像系统存根中使用的那些一样。如果我们想为一些资源创建一个新的测试规则，通过设置和拆卸，我们可以子类化`ExternalResource`并提供`before`和`after`方法的覆盖。

JUnit 5 有一个更复杂的资源管理模式。对于简单的用例，可以使用系统存根库作为起点。`SystemStubsExtension`对满足`TestResource`接口的任何东西进行操作。

### 8.1.创建一个`TestResource`

我们可以创建一个`TestResource`的子类，然后像使用系统存根一样使用我们的自定义对象。我们应该注意，如果我们想要使用字段和参数的自动创建，我们需要提供一个默认的构造函数。

假设我们想为一些测试打开一个到数据库的连接，然后关闭它:

```
public class FakeDatabaseTestResource implements TestResource {
    // let's pretend this is a database connection
    private String databaseConnection = "closed";

    @Override
    public void setup() throws Exception {
        databaseConnection = "open";
    }

    @Override
    public void teardown() throws Exception {
        databaseConnection = "closed";
    }

    public String getDatabaseConnection() {
        return databaseConnection;
    }
}
```

我们使用`databaseConnection`字符串作为资源的例子，比如数据库连接。**我们在`setup`和`teardown`方法中修改资源的状态。**

### 8.2.执行循环是内置的

现在让我们尝试将它与执行循环模式一起使用:

```
FakeDatabaseTestResource fake = new FakeDatabaseTestResource();
assertThat(fake.getDatabaseConnection()).isEqualTo("closed");

fake.execute(() -> {
    assertThat(fake.getDatabaseConnection()).isEqualTo("open");
});
```

正如我们所见，`TestResource`接口赋予了它执行其他对象的能力。

### 8.3.JUnit 5 测试中的自定义`TestResource`

我们也可以在 JUnit 5 测试中使用它:

```
@ExtendWith(SystemStubsExtension.class)
class FakeDatabaseJUnit5UnitTest {

    @Test
    void useFakeDatabase(FakeDatabaseTestResource fakeDatabase) {
        assertThat(fakeDatabase.getDatabaseConnection()).isEqualTo("open");
    }
}
```

因此，**很容易创建遵循系统存根设计的额外测试对象**。

## 9.JUnit 5 Spring 测试的环境和属性覆盖

为 Spring 测试设置环境变量可能很困难。我们可以[为集成测试编写一个定制规则](/web/20220627090235/https://www.baeldung.com/spring-boot-testcontainers-integration-test),为 Spring 设置一些系统属性。

我们也可以使用一个 [`ApplicationContextInitializer`类插入到我们的 Spring 上下文](/web/20220627090235/https://www.baeldung.com/spring-tests-override-properties#testPropertySourceUtils)中，为测试提供额外的属性。

由于许多 Spring 应用程序是由系统属性或环境变量覆盖来控制的，所以使用系统存根在外部测试中设置它们可能更容易，而 Spring 测试作为内部类运行。

在系统存根文档中提供了一个完整的示例。我们首先创建一个外部类:

```
@ExtendWith(SystemStubsExtension.class)
public class SpringAppWithDynamicPropertiesTest {

    // sets the environment before Spring even starts
    @SystemStub
    private static EnvironmentVariables environmentVariables;
}
```

在这种情况下,@ `SystemStub `字段是`static`并且在`@BeforeAll`方法中被初始化:

```
@BeforeAll
static void beforeAll() {
     String baseUrl = ...;

     environmentVariables.set("SERVER_URL", baseUrl);
}
```

测试生命周期中的这一点允许在 Spring 测试运行之前创建一些全局资源并应用到运行环境中。

然后，我们可以把春考放到一个`@Nested`类中。这导致它仅在设置了父类时运行:

```
@Nested
@SpringBootTest(classes = {RestApi.class, App.class},
    webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class InnerSpringTest {
    @LocalServerPort
    private int serverPort;

    // Test methods
}
```

Spring 上下文是根据外部类中@ `SystemStub`对象设置的环境状态创建的。

这种技术还允许我们控制任何其他库的配置，这些库依赖于可能在 Spring Beans 后面运行的系统属性或环境变量的状态。

这可以让我们**在 Spring 测试运行之前，挂钩到测试生命周期来修改诸如代理设置或 HTTP 连接池参数**之类的东西。

## 10.结论

在本文中，我们已经了解了能够模拟系统资源的重要性，以及 System Stubs 如何通过其 JUnit 4 和 JUnit 5 插件以最少的代码重复实现复杂的存根配置。

我们看到了如何在测试中提供和隔离环境变量和系统属性。然后，我们看了如何在标准流上捕获输出和控制输入。我们还看了捕获和断言对`System.exit`的调用。

最后，我们看了如何创建定制的测试资源，以及如何在 Spring 中使用系统存根。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220627090235/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-libraries-2)