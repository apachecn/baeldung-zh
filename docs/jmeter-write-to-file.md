# 使用 JMeter 将提取的数据写入文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jmeter-write-to-file>

## 1.概观

在本教程中，让我们探索两种从 [Apache JMeter](/web/20221128050843/https://www.baeldung.com/jmeter) 中提取数据并将其写入外部文件的方法。

## 2.设置基本 JMeter 脚本

现在让我们从创建一个基本的 JMeter 脚本开始。让我们用单线程创建一个`Thread Group`(这是创建`Thread Group`时的默认设置):

[![JMeter create thread group](img/5104acfbe6111115940c4714c3bb1cb3.png)](/web/20221128050843/https://www.baeldung.com/wp-content/uploads/2021/02/JMeter-create-thread-group.png)

在这个`Thread Group`中，我们现在创建一个`HTTP Sampler`:

[![JMeter create Http sampler](img/5dc58f65c4a39d48a0099021f46629fb.png)](https://web.archive.org/web/20221128050843/https://baeldung.com/wp-content/uploads/2021/02/JMeter-create-http-sampler.png)

让我们设置我们的`HTTP Sampler`来调用运行在`localhost.` 上的 API，我们可以从用一个简单的 [REST 控制器](/web/20221128050843/https://www.baeldung.com/spring-controller-vs-restcontroller)定义 API 开始:

```java
@RestController
public class RetrieveUuidController {

    @GetMapping("/api/uuid")
    public Response uuid() {
        return new Response(format("Test message... %s.", UUID.randomUUID()));
    }
}
```

此外，让我们也定义由我们的控制器返回的`Response` 实例，如上所述:

```java
public class Response {
    private Instant timestamp;
    private UUID uuid;
    private String message;

    // getters, setters, and constructor omitted
}
```

现在让我们用它来测试我们的 JMeter 脚本。默认情况下，这将在端口 8080 上运行。如果我们不能使用端口 8080，那么我们需要相应地更新`HTTP Sampler` 中的`Port Number` 字段`.`

`HTTP Sampler`请求应该是这样的:

[![JMeter http sampler details](img/9154d7c3c7c5ade260d2fa7e5090e7fb.png)](https://web.archive.org/web/20221128050843/https://baeldung.com/wp-content/uploads/2021/02/JMeter-http-sampler-details.png)

## 3.使用监听器写入提取的输出

接下来，让我们使用一个类型为`Save Responses to a file`的监听器将我们想要的数据提取到一个文件中:

[![JMeter write listener](img/15780269b65fe9f9f70d30e23bc67e52.png)](https://web.archive.org/web/20221128050843/https://baeldung.com/wp-content/uploads/2021/02/JMeter-WriteListener.png)

**使用这个监听器很方便，但是在我们提取到一个文件**的内容上没有太大的灵活性。对于我们的例子，这将生成一个 JSON 文件，保存到 JMeter 当前运行的位置(尽管路径可以在`Filename Prefix`字段中配置)。

## 4.使用`PostProcessor`写入提取的输出

我们可以将数据提取到文件中的另一种方法是创建一个 [`BeanShell`](https://web.archive.org/web/20221128050843/https://jmeter.apache.org/usermanual/component_reference.html#BeanShell_Sampler) `PostProcessor`。 **`BeanShell`是一个非常灵活的脚本处理器，它允许我们使用 Java 代码编写脚本，并利用 JMeter** 提供的一些内置变量。

`BeanShell`可用于各种不同的用例。在这种情况下，让我们创建一个`BeanShell`后处理器，并添加一个脚本来帮助我们将一些数据提取到一个文件中:

[![JMeter BeanShell PostProcessor](img/eb5d0f7f2a4c2a4ebfd38b3278136ea8.png)](https://web.archive.org/web/20221128050843/https://baeldung.com/wp-content/uploads/2021/02/JMeter-BeanShell-PostProcessor.png)

现在让我们将以下脚本添加到`Script`部分:

```java
FileWriter fWriter = new FileWriter("/<path>/result.txt", true);
BufferedWriter buff = new BufferedWriter(fWriter);

buff.write("data");

buff.close();
fWriter.close();
```

我们现在有一个简单的脚本，它将字符串`data`输出到一个名为 result 的文件中。这里需要注意的重要一点是`FileWriter` 构造函数的第二个参数。**这必须设置为`true`，这样我们的`BeanShell`将追加到文件中，而不是覆盖它。** **在 JMeter 中使用多线程时这一点非常重要。**

接下来，我们想要提取对我们的用例更有意义的东西。让我们利用 JMeter 提供的 **`[ctx](https://web.archive.org/web/20221128050843/https://jmeter.apache.org/api/org/apache/jmeter/threads/JMeterContext.html)`** 变量。这将允许我们访问运行 HTTP 请求的单个线程所拥有的上下文。

从`ctx`中，我们获取响应代码、响应头和响应体，并将它们提取到我们的文件中:

```java
buff.write("Response Code : " + ctx.getPreviousResult().getResponseCode());
buff.write(System.getProperty("line.separator"));
buff.write("Response Headers : " + ctx.getPreviousResult().getResponseHeaders());
buff.write(System.getProperty("line.separator"));
buff.write("Response Body : " + new String(ctx.getPreviousResult().getResponseData()));
```

如果我们想收集特定的字段数据并将其写入我们的文件，我们可以利用`[**vars**](https://web.archive.org/web/20221128050843/https://jmeter.apache.org/api/org/apache/jmeter/threads/JMeterVariables.html)`变量。这是一个我们可以在`PostProcessors`中用来存储和检索字符串数据的映射。

对于这个更复杂的例子，让我们在文件提取器之前创建另一个`PostProcessor`。这将在 HTTP 请求的 JSON 响应中进行搜索:

[![JMeter JSON Exctractor](img/1d9c9acaa7e2b5e50e4dbfcc85f03313.png)](https://web.archive.org/web/20221128050843/https://baeldung.com/wp-content/uploads/2021/02/JMeter-JSON-Exctractor.png)

这个提取器将创建一个名为`message`的变量。剩下要做的就是在我们的文件提取器中引用这个变量，将其输出到我们的文件中:

```java
buff.write("More complex extraction : " + vars.get("message"));
```

注意:我们可以将这种方法与其他后处理器(如“正则表达式提取器”)结合使用，以更定制的方式收集信息。

## 5.结论

在本教程中，我们介绍了如何使用 BeanShell 后处理器和写监听器从 JMeter 提取数据到外部文件。我们使用的 JMeter 脚本和 Spring REST 应用程序可以在 GitHub 上找到[。](https://web.archive.org/web/20221128050843/https://github.com/eugenp/tutorials/tree/master/jmeter)