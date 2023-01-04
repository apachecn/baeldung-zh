# Spring Boot 应用程序中的日志属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-log-properties>

## 1.概观

属性是 Spring Boot 提供的最有用的机制之一。它们可以从各种地方提供，例如专用属性文件、环境变量等。因此，查找和记录特定的属性有时很有用，例如在调试时。

在这个简短的教程中，我们将看到在 Spring Boot 应用程序中查找和记录属性的几种不同方法。

首先，我们将创建一个简单的测试应用程序，我们将在 `. `上工作，然后，我们将尝试三种不同的方法来记录特定的属性。

## 2.创建测试应用程序

让我们创建一个具有三个自定义属性的简单应用程序。

我们可以使用 [Spring Initializr](https://web.archive.org/web/20221028202328/https://start.spring.io/) 来创建一个 Spring Boot 应用程序模板。我们将使用 Java 作为语言。我们可以自由选择其他选项，如 Java 版本、项目元数据等。

下一步是向我们的应用程序添加自定义属性。我们将这些属性添加到`src/main/resources`中的一个新的`application.properties`文件中:

```java
app.name=MyApp
app.description=${app.name} is a Spring Boot application
bael.property=stagingValue
```

## 3.使用上下文刷新事件记录属性

在 Spring Boot 应用程序中记录属性的第一种方式是使用 [Spring Events](/web/20221028202328/https://www.baeldung.com/spring-events) ，特别是`org.springframework.context.event.ContextRefreshedEvent`类和相应的`EventListener.`我们将展示如何记录所有可用的属性，以及一个更详细的版本，它只打印特定文件中的属性。

### 3.1.记录所有属性

让我们从创建 bean 和事件监听器方法开始:

```java
@Component
public class AppContextRefreshedEventPropertiesPrinter {

    @EventListener
    public void handleContextRefreshed(ContextRefreshedEvent event) {
        // event handling logic
    }
}
```

我们用`org.springframework.context.event.EventListener`注释来注释事件监听器方法。当`ContextRefreshedEvent`发生时，Spring 调用带注释的方法。

下一步是从触发的事件中获取一个`org.springframework.core.env.ConfigurableEnvironment`接口的实例。**`ConfigurableEnvironment` 接口提供了一个有用的方法`getPropertySources()`，我们将使用它来获取所有属性源**的列表，例如环境、JVM 或属性文件变量:

```java
ConfigurableEnvironment env = (ConfigurableEnvironment) event.getApplicationContext().getEnvironment();
```

现在让我们看看如何使用它来打印所有属性，不仅从`application.properties `文件，还从环境、JVM 变量等等:

```java
env.getPropertySources()
    .stream()
    .filter(ps -> ps instanceof MapPropertySource)
    .map(ps -> ((MapPropertySource) ps).getSource().keySet())
    .flatMap(Collection::stream)
    .distinct()
    .sorted()
    .forEach(key -> LOGGER.info("{}={}", key, env.getProperty(key)));
```

首先，我们从可用的属性源中创建一个 [`Stream`](/web/20221028202328/https://www.baeldung.com/java-streams) 。然后，我们使用它的 `filter()` 方法迭代属性源，这些属性源是 `org.springframework.core.env.MapPropertySource` 类的实例。

顾名思义，属性源中的属性存储在一个映射结构中。我们将在下一步中使用它，在下一步中，我们将使用流的`map()`方法来获取属性键集。

接下来，我们使用`Stream`的 `flatMap()` 方法，因为我们想要迭代单个属性键，而不是一组键。我们还希望有独特的，而不是重复的，按字母顺序打印的属性。

最后一步是记录属性键及其值。

当我们启动应用程序时，我们应该看到一个从各种来源获取的属性的大列表:

```java
COMMAND_MODE=unix2003
CONSOLE_LOG_CHARSET=UTF-8
...
bael.property=defaultValue
app.name=MyApp
app.description=MyApp is a Spring Boot application
...
java.class.version=52.0
ava.runtime.name=OpenJDK Runtime Environment
```

### 3.2.仅记录来自`application.properties`文件的属性

如果我们只想记录在`application.properties`文件中发现的属性，我们可以重用几乎所有以前的代码。我们只需要改变传递给`filter()`方法的 lambda 函数:

```java
env.getPropertySources()
    .stream()
    .filter(ps -> ps instanceof MapPropertySource && ps.getName().contains("application.properties"))
    ...
```

现在，当我们启动应用程序时，我们应该会看到以下日志:

```java
bael.property=defaultValue
app.name=MyApp
app.description=MyApp is a Spring Boot application
```

## 4.使用`Environment`界面记录属性

记录属性的另一种方法是使用 `org.springframework.core.env.Environment`接口:

```java
@Component
public class EnvironmentPropertiesPrinter {
    @Autowired
    private Environment env;

    @PostConstruct
    public void logApplicationProperties() {
        LOGGER.info("{}={}", "bael.property", env.getProperty("bael.property"));
        LOGGER.info("{}={}", "app.name", env.getProperty("app.name"));
        LOGGER.info("{}={}", "app.description", env.getProperty("app.description"));
    }
}
```

与上下文刷新事件方法相比，唯一的限制是**我们需要知道属性名来获得它的值**。环境接口没有提供列出所有属性的方法。另一方面，这绝对是一个更短更容易的技术。

当我们启动应用程序时，我们应该会看到与前面相同的输出:

```java
bael.property=defaultValue 
app.name=MyApp 
app.description=MyApp is a Spring Boot application
```

## 5.弹簧致动器的测井特性

[Spring Actuator](/web/20221028202328/https://www.baeldung.com/spring-boot-actuators) 是一个非常有用的库，它为我们的应用程序带来了生产就绪的特性。**`/env`REST 端点返回当前环境属性**。

首先，让我们将[弹簧执行器库](https://web.archive.org/web/20221028202328/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-actuator/2.7.3/jar)添加到我们的项目中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.7.3</version>
</dependency>
```

接下来，我们需要启用`/env`端点，因为默认情况下它是禁用的。让我们打开`application.properties`并添加以下条目:

```java
management.endpoints.web.exposure.include=env
```

现在，我们所要做的就是启动应用程序并转到`/env`端点。在我们的例子中，地址是`http://localhost:8080/actuator/env. `,我们应该看到一个包含所有环境变量的大型 JSON，包括我们的属性:

```java
{
  "activeProfiles": [],
  "propertySources": [
    ...
    {
      "name": "Config resource 'class path resource [application.properties]' via location 'optional:classpath:/' (document #0)",
      "properties": {
        "app.name": {
          "value": "MyApp",
          "origin": "class path resource [application.properties] - 10:10"
        },
        "app.description": {
          "value": "MyApp is a Spring Boot application",
          "origin": "class path resource [application.properties] - 11:17"
        },
        "bael.property": {
          "value": "defaultValue",
          "origin": "class path resource [application.properties] - 13:15"
        }
      }
    }
   ...
  ]
}
```

## 6.结论

在本文中，我们学习了如何在 Spring Boot 应用程序中记录属性。

首先，我们创建了一个具有三个定制属性的测试应用程序。然后，我们看到了检索和记录所需属性的三种不同方法。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221028202328/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-3)