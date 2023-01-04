# Log4j 2 的编程配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/log4j2-programmatic-config>

## 1。简介

在本教程中，我们将看看以编程方式配置 Apache Log4j 2 的不同方法。

## 2。初始设置

要开始使用 Log4j 2，我们只需要在我们的`pom.xml`中包含 [log4j-core](https://web.archive.org/web/20220730031635/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-core%22) 和 [log4j-slf4j-impl](https://web.archive.org/web/20220730031635/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-slf4j-impl%22) 依赖项:

```
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.11.0</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.11.0</version>
</dependency>
```

## 3。`ConfigurationBuilder`

一旦我们配置了 Maven，那么我们需要创建一个`ConfigurationBuilder`，这个类让我们配置`appenders, filters, layouts,`和`loggers.`

Log4j 2 提供了几种获取`ConfigurationBuilder`的方法。

让我们从最直接的方式开始:

```
ConfigurationBuilder<BuiltConfiguration> builder
 = ConfigurationBuilderFactory.newConfigurationBuilder();
```

为了开始配置组件，`ConfigurationBuilder`为每个组件配备了相应的`new`方法，如`newAppender `或`newLayout`。

一些组件有不同的子类型，如`FileAppender`或`ConsoleAppender,`，这些在 API 中被称为`plugins`。

### 3.1。配置附加器

让我们通过配置一个`appender`来告诉`builder`将每个日志行发送到哪里:

```
AppenderComponentBuilder console 
  = builder.newAppender("stdout", "Console"); 

builder.add(console);

AppenderComponentBuilder file 
  = builder.newAppender("log", "File"); 
file.addAttribute("fileName", "target/logging.log");

builder.add(file);
```

虽然大多数`new`方法不支持这一点，但是`newAppender(name, plugin)`允许我们给 appender 起一个名字，这在以后会变得很重要。这些附加器，我们称之为`stdout`和`log, `，尽管我们可以给它们取任何名字。

我们还告诉了`builder`使用哪个追加器`plugin `(或者更简单地说，哪种追加器)。`Console`和`File`分别指 Log4j 2 的写入标准输出和文件系统的附加器。

虽然 [Log4j 2 支持几个 appender](https://web.archive.org/web/20220730031635/https://logging.apache.org/log4j/2.x/manual/appenders.html)，**使用 Java 配置它们可能有点棘手，因为`AppenderComponentBuilder `是所有 appender 类型的通用类。**

这使得它有类似于`addAttribute` 和`addComponent `的方法，而不是`setFileName`和`addTriggeringPolicy`:

```
AppenderComponentBuilder rollingFile 
  = builder.newAppender("rolling", "RollingFile");
rollingFile.addAttribute("fileName", "rolling.log");
rollingFile.addAttribute("filePattern", "rolling-%d{MM-dd-yy}.log.gz");

builder.add(rollingFile); 
```

最后，**不要忘记调用`builder.add`将其追加到主配置中！**

### 3.2。配置过滤器

我们可以在每个 appender 中添加过滤器，它们决定是否应该在每个日志行中添加内容。

让我们使用控制台附加器上的`MarkerFilter`插件:

```
FilterComponentBuilder flow = builder.newFilter(
  "MarkerFilter", 
  Filter.Result.ACCEPT,
  Filter.Result.DENY);  
flow.addAttribute("marker", "FLOW");

console.add(flow);
```

注意，这个`new`方法不允许我们命名过滤器，但是它要求我们指出如果过滤器通过或失败该做什么。

在这种情况下，我们保持简单，声明如果`MarkerFilter`通过，那么`ACCEPT`通过 logline。否则，`DENY `言之凿凿。

**注意，在这种情况下，我们没有将它附加到`builder`中，而是附加到我们希望使用该过滤器的附加器中。**

### 3.3。配置布局

接下来，让我们定义每个日志行的布局。在这种情况下，我们将使用`PatternLayout`插件:

```
LayoutComponentBuilder standard 
  = builder.newLayout("PatternLayout");
standard.addAttribute("pattern", "%d [%t] %-5level: %msg%n%throwable");

console.add(standard);
file.add(standard);
rolling.add(standard);
```

同样，我们将它们直接添加到适当的 appenders 中，而不是直接添加到`builder`中。

### 3.4。配置根日志记录器

现在我们知道了日志将被运送到哪里，我们想要配置哪些日志将到达每个目的地。

根日志记录器是最高的日志记录器，有点像 Java 中的`Object`。除非被覆盖，否则默认情况下将使用该记录器。

因此，让我们使用一个根日志记录器将默认日志记录级别设置为`ERROR`并将默认 appender 设置为上面的`stdout` appender:

```
RootLoggerComponentBuilder rootLogger 
  = builder.newRootLogger(Level.ERROR);
rootLogger.add(builder.newAppenderRef("stdout"));

builder.add(rootLogger);
```

为了将我们的记录器指向一个特定的 appender，我们没有给它一个构建器的实例。相反，**我们用之前给出的`name`来指代它。**

### 3.5。配置附加记录器

子记录器可用于定位特定的包或记录器名称。

让我们为应用程序中的`com`包添加一个日志记录器，将日志记录级别设置为`DEBUG`,并让它们进入我们的`log `附加器:

```
LoggerComponentBuilder logger = builder.newLogger("com", Level.DEBUG);
logger.add(builder.newAppenderRef("log"));
logger.addAttribute("additivity", false);

builder.add(logger);
```

注意，我们可以用我们的记录器设置`additivity `,它指示这个记录器是否应该从它的祖先继承像日志记录级别和 appender 类型这样的属性。

### 3.6。配置其他组件

不是所有的组件在`ConfigurationBuilder`上都有专用的`new`方法。

所以，在这种情况下，我们称之为`newComponent.`

例如，因为没有`TriggeringPolicyComponentBuilder`，我们需要使用`newComponent `来指定滚动文件附加器的触发策略:

```
ComponentBuilder triggeringPolicies = builder.newComponent("Policies")
  .addComponent(builder.newComponent("CronTriggeringPolicy")
    .addAttribute("schedule", "0 0 0 * * ?"))
  .addComponent(builder.newComponent("SizeBasedTriggeringPolicy")
    .addAttribute("size", "100M"));

rolling.addComponent(triggeringPolicies);
```

### 3.7。XML 等价物

`ConfigurationBuilder `提供了一种简便的方法来打印出等价的 XML:

```
builder.writeXmlConfiguration(System.out);
```

运行上面的行打印出:

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
   <Appenders>
      <Console name="stdout">
         <PatternLayout pattern="%d [%t] %-5level: %msg%n%throwable" />
         <MarkerFilter onMatch="ACCEPT" onMisMatch="DENY" marker="FLOW" />
      </Console>
      <RollingFile name="rolling" 
        fileName="target/rolling.log" 
        filePattern="target/archive/rolling-%d{MM-dd-yy}.log.gz">
         <PatternLayout pattern="%d [%t] %-5level: %msg%n%throwable" />
         <Policies>
            <CronTriggeringPolicy schedule="0 0 0 * * ?" />
            <SizeBasedTriggeringPolicy size="100M" />
         </Policies>
      </RollingFile>
      <File name="FileSystem" fileName="target/logging.log">
         <PatternLayout pattern="%d [%t] %-5level: %msg%n%throwable" />
      </File>
   </Appenders>
   <Loggers>
      <Logger name="com" level="DEBUG" additivity="false">
         <AppenderRef ref="log" />
      </Logger>
      <Root level="ERROR" additivity="true">
         <AppenderRef ref="stdout" />
      </Root>
   </Loggers>
</Configuration>
```

当我们想要仔细检查我们的配置或者如果我们想要持久化我们的配置，比如说，到文件系统，这就很方便了。

### 3.8。将所有这些放在一起

现在我们已经完全配置好了，让我们告诉 Log4j 2 使用我们的配置:

```
Configurator.initialize(builder.build());
```

在这个被调用之后，**未来对 Log4j 2 的调用将使用我们的配置**。

**注意，这意味着我们需要在调用`LogManager.getLogger`之前调用`Configurator.initialize`。**

## 4。`ConfigurationFactory`

现在我们已经看到了一种获取和应用`ConfigurationBuilder`的方法，让我们再来看看另一种方法:

```
public class CustomConfigFactory
  extends ConfigurationFactory {

    public Configuration createConfiguration(
      LoggerContext context, 
      ConfigurationSource src) {

        ConfigurationBuilder<BuiltConfiguration> builder = super
          .newConfigurationBuilder();

        // ... configure appenders, filters, etc.

        return builder.build();
    }

    public String[] getSupportedTypes() { 
        return new String[] { "*" };
    }
}
```

在这种情况下，我们没有使用`ConfigurationBuilderFactory`，而是子类化了`ConfigurationFactory`，这是一个抽象类，旨在创建`Configuration`的实例。

然后，我们只需要让 Log4j 2 知道我们的新配置工厂，而不是像第一次那样调用`Configurator.initialize`。

有三种方法可以做到这一点:

*   **静态初始化**
*   **一个运行时属性，或**
*   **`@Plugin `注解**

### 4.1。使用静态初始化

Log4j 2 支持静态初始化时调用`setConfigurationFactory `:

```
static {
    ConfigurationFactory custom = new CustomConfigFactory();
    ConfigurationFactory.setConfigurationFactory(custom);
}
```

这种方法和我们看到的上一种方法有相同的限制，那就是我们需要在任何对`LogManager.getLogger` `.`的调用之前调用它

 **### 4.2。使用运行时属性

如果我们可以访问 Java 启动命令，那么 Log4j 2 也支持通过一个`-D`参数指定要使用的`ConfigurationFactory `:

```
-Dlog4j2.configurationFactory=com.baeldung.log4j2.CustomConfigFactory
```

这种方法的主要好处是我们不必像前两种方法那样担心初始化顺序。

### 4.3。使用`@Plugin `标注

最后，在我们不想通过添加`-D`来篡改 Java 启动命令的情况下，我们可以简单地用 Log4j 2 `@Plugin `注释来注释我们的`CustomConfigurationFactory`:

```
@Plugin(
  name = "CustomConfigurationFactory", 
  category = ConfigurationFactory.CATEGORY)
@Order(50)
public class CustomConfigFactory
  extends ConfigurationFactory {

  // ... rest of implementation
}
```

Log4j 2 将在类路径中扫描带有`@Plugin `注释的类，如果在`ConfigurationFactory `类别中找到这个类，就会使用它。

### 4.4。结合静态配置

使用`ConfigurationFactory`扩展的另一个好处是，我们可以轻松地将我们的定制配置与其他配置源(如 XML)结合起来:

```
public Configuration createConfiguration(
  LoggerContext context, 
  ConfigurationSource src) {
    return new WithXmlConfiguration(context, src);
} 
```

`source`参数表示 Log4j 2 找到的静态 XML 或 JSON 配置文件。

我们可以将配置文件发送给我们的定制实现`XmlConfiguration`,在那里我们可以放置我们需要的任何覆盖配置:

```
public class WithXmlConfiguration extends XmlConfiguration {

    @Override
    protected void doConfigure() {
        super.doConfigure(); // parse xml document

        // ... add our custom configuration
    }
}
```

## 5。结论

在本文中，我们研究了如何使用 Log4j 2 中新的`ConfigurationBuilder` API。

我们还研究了如何定制与`ConfigurationBuilder`结合的`ConfigurationFactory `来实现更高级的用例。

不要忘记在 GitHub 上查看我的完整示例[。](https://web.archive.org/web/20220730031635/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j2)**