# 一个用 Java 编写的基本 AWS Lambda 示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-aws-lambda>

## 1。简介

[AWS Lambda](https://web.archive.org/web/20220630124205/https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) 是亚马逊提供的无服务器计算服务，用于减少服务器、OS、可扩展性等的配置。AWS Lambda 能够在 AWS Cloud 上执行代码。

它运行以响应不同 AWS 资源上的事件，这触发了 AWS Lambda 函数。定价是现收现付，这意味着我们不会把钱花在闲置的 lambda 函数上。

本教程需要有效的 AWS 帐户；你可以在这里创建一个。

## 2。Maven 依赖关系

要启用 AWS lambda，我们的项目中需要以下依赖项:

```java
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-lambda-java-core</artifactId>
    <version>1.1.0</version>
</dependency>
```

这种依赖性可以在 [Maven 仓库](https://web.archive.org/web/20220630124205/https://search.maven.org/classic/#search%7Cga%7C1%7Caws-lambda-java-core)中找到。

我们还需要 [Maven Shade 插件](https://web.archive.org/web/20220630124205/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22maven-shade-plugin%22)来构建 lambda 应用程序:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>2.4.3</version>
    <configuration>
        <createDependencyReducedPom>false</createDependencyReducedPom>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
	    <goals>
                <goal>shade</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## 3。创建处理程序

简单地说，要调用一个 lambda 函数，我们需要指定一个处理程序；创建处理程序有三种方式:

1.  创建自定义`MethodHandler`
2.  实现`RequestHandler`接口
3.  实现`RequestStreamHandler`接口

让我们使用代码示例来看看如何做到这一点。

### 3.1。 `MethodHandler`风俗

我们将创建一个 handler 方法，作为传入请求的入口点。我们可以使用 JSON 格式或原始数据类型作为输入值。

另外，可选的`Context`对象将允许我们访问 Lambda 执行环境中可用的有用信息:

```java
public class LambdaMethodHandler {
    public String handleRequest(String input, Context context) {
        context.getLogger().log("Input: " + input);
        return "Hello World - " + input;
    }
}
```

### 3.2。`RequestHandler` 界面

我们还可以将`RequestHandler`实现到我们的类中，并覆盖`handleRequest`方法，这将是我们请求的入口点:

```java
public class LambdaRequestHandler
  implements RequestHandler<String, String> {
    public String handleRequest(String input, Context context) {
        context.getLogger().log("Input: " + input);
        return "Hello World - " + input;
    }
}
```

在这种情况下，输入将与第一个示例相同。

### 3.3。`RequestStreamHandler` 界面

我们也可以在我们的类中实现`RequestStreamHandler`并简单地覆盖`handleRequest`方法。

不同之处在于，`InputStream`、`ObjectStream`和`Context`对象是作为参数传递的:

```java
public class LambdaRequestStreamHandler
  implements RequestStreamHandler {
    public void handleRequest(InputStream inputStream, 
      OutputStream outputStream, Context context) {
        String input = IOUtils.toString(inputStream, "UTF-8");
        outputStream.write(("Hello World - " + input).getBytes());
    }
}
```

## 4。构建部署文件

完成所有配置后，我们可以通过运行以下命令来创建部署文件:

```java
mvn clean package shade:shade
```

将在`target`文件夹下创建`jar`文件。

## 5。通过管理控制台创建 Lambda 函数

登录 [AWS 亚马逊](https://web.archive.org/web/20220630124205/https://aws.amazon.com/)，然后点击服务下的 Lambda。该页面将显示已经创建的 lambda 函数列表。

下面是创建我们的 lambda 所需的步骤:

1.  **“选择蓝图”**然后选择“**空白功能”**
2.  **“配置触发器”**(在我们的例子中，我们没有任何触发器或事件)
3.  **“配置功能”:**
    *   名称:提供**方法 HandlerLambda** ，
    *   描述:描述我们的 lambda 函数的任何东西
    *   运行时:选择 **java8**
    *   代码入口类型和功能包:选择“**上传一个. ZIP 和 Jar 文件”**，点击“**上传”**按钮。选择包含 lambda 代码的文件。
    *   在**下 Lambda 函数处理程序和角色**:
        *   处理程序名:提供 lambda 函数处理程序名`**com.baeldung.MethodHandlerLambda::handleRequest**`
        *   角色名称:如果 lambda 函数中使用了任何其他 AWS 资源，则通过创建/使用现有角色来提供访问权限，并定义策略模板。
    *   在**高级设置下:**
        *   内存:提供我们的 lambda 函数将使用的内存。
        *   超时:为每个请求选择执行 lambda 函数的时间。
4.  完成所有输入后，点击“**下一步”**，查看配置。
5.  评审完成后，点击**创建功能**。

## 6。调用功能

一旦创建了 AWS lambda 函数，我们将通过传入一些数据来测试它:

*   点击列表中的 lambda 函数，然后点击“**测试”**按钮
*   将出现一个弹出窗口，其中包含用于发送数据的虚拟值。用**“bael dung”**覆盖数据
*   点击**保存并测试**按钮

在屏幕上，您可以看到**执行结果**部分，成功返回的输出如下:

```java
"Hello World - Baeldung"
```

## 7 .**。结论**

在这篇快速介绍文章中，我们使用 Java 8 创建了一个简单的 AWS Lambda 应用程序，将其部署到 AWS 并进行测试。

示例应用的完整源代码可以在 Github 的[中找到。](https://web.archive.org/web/20220630124205/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-lambda)