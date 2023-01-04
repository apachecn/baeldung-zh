# 在 Groovy 中使用 JSON

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-json>

## 1.介绍

在本文中，我们将描述并查看如何在一个 [Groovy](/web/20220807102518/https://www.baeldung.com/groovy-language) 应用程序中使用 JSON 的例子。

首先，要启动并运行本文的示例，我们需要设置我们的`pom.xml`:

```java
<build>
    <plugins>
        // ...
        <plugin>
            <groupId>org.codehaus.gmavenplus</groupId>
            <artifactId>gmavenplus-plugin</artifactId>
            <version>1.6</version>
        </plugin>
    </plugins>
</build>
<dependencies>
    // ...
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy-all</artifactId>
        <version>2.4.13</version>
    </dependency>
</dependencies>
```

最新的 Maven 插件可以在这里找到[，最新版本的`groovy-all`](https://web.archive.org/web/20220807102518/https://mvnrepository.com/artifact/org.codehaus.gmavenplus/gmavenplus-plugin) [可以在这里](https://web.archive.org/web/20220807102518/https://mvnrepository.com/artifact/org.codehaus.groovy/groovy-all)找到。

## 2.将 Groovy 对象解析为 JSON

在 Groovy 中将对象转换成 JSON 非常简单，假设我们有一个`Account`类:

```java
class Account {
    String id
    BigDecimal value
    Date createdAt
}
```

为了将该类的实例转换成 JSON `String,` ，我们需要使用`JsonOutput` 类并调用静态方法`toJson():`

```java
Account account = new Account(
    id: '123', 
    value: 15.6,
    createdAt: new SimpleDateFormat('MM/dd/yyyy').parse('01/01/2018')
) 
println JsonOutput.toJson(account)
```

结果，我们将得到解析后的 JSON `String:`

```java
{"value":15.6,"createdAt":"2018-01-01T02:00:00+0000","id":"123"}
```

### 2.1.定制 JSON 输出

正如我们所看到的，日期输出不是我们想要的。为此，从 2.5 版本开始，包`groovy.json` 附带了一组专用的工具。

使用`JsonGenerator`类，我们可以定义 JSON 输出的选项:

```java
JsonGenerator generator = new JsonGenerator.Options()
  .dateFormat('MM/dd/yyyy')
  .excludeFieldsByName('value')
  .build()

println generator.toJson(account)
```

因此，我们将得到格式化的 JSON，它没有我们排除的值字段，但带有格式化的日期:

```java
{"createdAt":"01/01/2018","id":"123"}
```

### 2.2.格式化 JSON 输出

通过上面的方法，我们看到 JSON 输出总是在一行中，如果要处理更复杂的对象，这可能会令人困惑。

然而，我们可以使用`prettyPrint`方法格式化我们的输出:

```java
String json = generator.toJson(account)
println JsonOutput.prettyPrint(json)
```

我们得到如下格式化的 JSON:

```java
{
    "value": 15.6,
    "createdAt": "01/01/2018",
    "id": "123"
}
```

## 3.将 JSON 解析为 Groovy 对象

我们将使用 Groovy 类`JsonSlurper` 从 JSON 转换到`Objects.`

同样，对于`JsonSlurper`，我们有一堆重载的`parse` 方法和一些特定的方法，比如 `parseText`、`parseFile,` 等等。

我们将使用`parseText` 将`String`解析为`Account class:`

```java
def jsonSlurper = new JsonSlurper()

def account = jsonSlurper.parseText('{"id":"123", "value":15.6 }') as Account
```

在上面的代码中，我们有一个接收 JSON `String`并返回一个`Account`对象的方法，这个对象可以是任何 Groovy 对象。

此外，我们可以将 JSON `String`解析为调用它的`Map,`,而不需要任何强制转换，使用 Groovy 动态类型，我们可以将它作为对象。

### 3.1.解析 JSON 输入

`JsonSlurper`的默认解析器实现是 `JsonParserType.CHAR_BUFFER`，但是在某些情况下，我们需要处理一个解析问题。

让我们看一个这样的例子:给定一个带有日期属性的 JSON`String`,`JsonSlurper`不会正确地创建对象，因为它会试图将日期解析为`String:`

```java
def jsonSlurper = new JsonSlurper()
def account 
  = jsonSlurper.parseText('{"id":"123","createdAt":"2018-01-01T02:00:00+0000"}') as Account
```

因此，上面的代码将返回一个包含所有包含`null`值的属性的`Account`对象。

为了解决这个问题，我们可以使用`JsonParserType.INDEX_OVERLAY.`

因此，它会尽可能避免创建`String`或 char 数组:

```java
def jsonSlurper = new JsonSlurper(type: JsonParserType.INDEX_OVERLAY)
def account 
  = jsonSlurper.parseText('{"id":"123","createdAt":"2018-01-01T02:00:00+0000"}') as Account
```

现在，上面的代码将返回一个适当创建的`Account`实例。

### 3.2。解析器变体

此外，在`JsonParserType,`中，我们有一些其他的实现:

*   `JsonParserType.LAX`将允许更宽松的 JSON 解析，有注释，没有引用字符串等。
*   `JsonParserType.CHARACTER_SOURCE`用于大型文件解析。

## 4.结论

我们已经用几个简单的例子介绍了 Groovy 应用程序中的许多 JSON 处理。

关于`groovy.json`包类的更多信息，我们可以看看 [Groovy 文档](https://web.archive.org/web/20220807102518/http://groovy-lang.org/json.html)。

在我们的 [GitHub 库](https://web.archive.org/web/20220807102518/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy)中检查本文中使用的类的源代码，以及一些单元测试。