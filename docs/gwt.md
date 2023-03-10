# GWT 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gwt>

## 1。简介

GWT 或 **Google Web Toolkit 是一个用 Java** 构建高性能 Web 应用程序的框架。

在本教程中，我们将重点介绍它的一些关键功能。

## 2。GWT SDK

SDK 包含 Java API 库、编译器和开发服务器。

### 2.1。Java API

GWT API 有用于构建用户界面、进行服务器调用、国际化、执行单元测试的类。要了解更多信息，请查看 java 文档[这里](https://web.archive.org/web/20220626085848/http://www.gwtproject.org/javadoc/latest/index.html)。

### 2.2。编译器

简单地说，GWT 编译器是一个从 Java 代码到 Javascript 的源代码翻译器。编译的结果是一个 Javascript 应用程序。

其工作逻辑包括从代码中删除未使用的类、方法和字段，以及缩短 Javascript 名称。

由于这个优势，我们不再需要在 Javascript 项目中包含 Ajax 库。当然，也可以在编译代码时设置提示。

以下是一些有用的`GWTCompiler`参数:

*   `-logLevel`–设置`ERROR, WARN, INFO, TRACE, DEBUG, SPAM, ALL`记录级别之一
*   `-workdir`–编译器的工作目录
*   `-gen`–写入生成文件的目录
*   `-out`–输出文件目录
*   `-optimize`–从 0 到 9 设置编译器优化级别
*   `-style`–脚本输出样式`OBF, PRETTY`或`DETAILED`
*   `-module[s]`–要编译的模块的名称

## 3。设置

最新的 SDK 可从[下载](https://web.archive.org/web/20220626085848/http://www.gwtproject.org/download.html)页面获得。其余的设置可在[入门](https://web.archive.org/web/20220626085848/http://www.gwtproject.org/gettingstarted.html)页面获得。

### 3.1。肚子

为了用 Maven 设置项目，我们需要向`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>com.google.gwt</groupId>
    <artifactId>gwt-servlet</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>com.google.gwt</groupId>
    <artifactId>gwt-user</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.google.gwt</groupId>
    <artifactId>gwt-dev</artifactId>
    <scope>provided</scope>
</dependency>
```

**`The gwt-servlet library supports the server-side components for invoking a GWT-RPC endpoint. gwt-user`包含 Java API，我们将使用它来构建我们的 web 应用程序**。`gwt-dev`拥有编译、部署或托管应用程序的代码。

为了确保所有的依赖项使用相同的版本，我们需要包括父 GWT 依赖项:

```java
<dependency>
    <groupId>com.google.gwt</groupId>
    <artifactId>gwt</artifactId>
    <version>2.8.2</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

所有的工件都可以在 [Maven Central](https://web.archive.org/web/20220626085848/https://mvnrepository.com/artifact/com.google.gwt) 上下载。

## 4。应用程序

让我们构建一个简单的 web 应用程序。它将向服务器发送一条消息并显示响应。

一般来说，GWT 应用程序由服务器和客户端组成。客户端发出一个 HTTP 请求来连接服务器。为了使之成为可能，GWT 使用了远程过程调用或简单的 RPC 机制。

## 5。GWT 和 RPC

回到我们的应用程序，让我们看看 RPC 通信是如何进行的。为此，我们创建了一个从服务器接收消息的服务。

让我们首先创建一个接口:

```java
@RemoteServiceRelativePath("greet")
public interface MessageService extends RemoteService {
    String sendMessage(String message) throws IllegalArgumentException;
}
```

**`@RemoteServiceRelativePath`注释将服务映射到模块的`/message`相对 URL。`MessageService`应该从`RemoteService`标记接口扩展来执行 RPC 通信**。

`MessageService`的实现在服务器端:

```java
public class MessageServiceImpl extends RemoteServiceServlet 
  implements MessageService {

    public String sendMessage(String message) 
      throws IllegalArgumentException {
        if (message == null) {
            throw new IllegalArgumentException("message is null");
        }

        return "Hello, " + message + "!<br><br> Time received: " 
          + LocalDateTime.now();
    }
}
```

我们的服务器类从基础 servlet 类`.` **扩展而来，它将自动反序列化来自客户端的传入请求，并序列化来自服务器的传出响应**。

现在让我们看看如何从客户端使用它。**`MessageService`只是我们服务**的最终版本。

为了在客户端执行，我们需要创建服务的异步版本:

```java
public interface MessageServiceAsync {
    void sendMessage(String input, AsyncCallback<String> callback) 
      throws IllegalArgumentException;
}
```

这里我们可以看到`getMessage()`方法中的一个额外的参数。**我们需要`async`在异步调用完成时通知 UI**。这样我们可以防止阻塞正在工作的 UI 线程。

## 6。组件及其生命周期

SDK 为设计图形界面提供了一些 UI 元素和布局。

一般来说，所有的 UI 组件都是从`Widget`类扩展而来的。视觉上，我们有可以在屏幕上看到、点击或移动的元素部件:

*   **组件控件**–`TextBox`、`TextArea`、`Button`、`RadioButton`、`CheckBox`等…

还有组成和组织屏幕的布局或面板小部件:

*   **面板控件**–`HorizontalPanel`、`VerticalPanel`、`PopupPanel`、`TabPanel`等

每次我们在代码中添加一个小部件或任何其他组件，GWT 都会努力将视图元素与浏览器的 DOM 链接起来。

构造函数总是初始化根 DOM 元素。当我们将一个子部件附加到一个父组件上时，也会导致 DOM 级别的绑定。入口点类包含将首先被调用的加载函数。这是我们定义小部件的地方。

## 7。入口点

让我们仔细看看应用程序的主要入口点:

```java
public class Google_web_toolkit implements EntryPoint {

    private MessageServiceAsync messageServiceAsync = GWT.create(MessageService.class);

    public void onModuleLoad() {
        Button sendButton = new Button("Submit");
        TextBox nameField = new TextBox();
        nameField.setText("Hi there");

        sendButton.addStyleName("sendButton");

        RootPanel.get("nameFieldContainer").add(nameField);
        RootPanel.get("sendButtonContainer").add(sendButton);
    }
}
```

**每个 UI 类都实现了`com.google.gwt.core.client.EntryPoint`接口，将其标记为模块**的主入口。它连接到相应的 HTML 文档，在那里执行 java 代码。

我们可以定义 GWT UI 组件，然后分配给具有相同 ID 的 HTML 标签。**入口点类覆盖入口点`onModuleLoad()`方法，该方法在加载模块**时被自动调用。

在这里，我们创建 UI 组件，注册事件处理程序，修改浏览器 DOM。

现在，让我们看看如何创建远程服务器实例。为此，我们使用`GWT.create(MessageService.class)`静态方法。

它在编译时确定请求的类型。看到这种方法， **GWT 编译器在编译时生成许多版本的代码，其中只有一个版本需要在运行时自举期间由特定客户端加载**。这个特性在 RPC 调用中广泛使用。

这里我们还定义了`Button`和`TextBox`小部件。为了将它们添加到 DOM 树中，我们使用了`RootPanel`类。它是根面板，返回一个 singleton 值来绑定小部件元素:

```java
RootPanel.get("sendButtonContainer").add(sendButton);
```

首先，它获取用`sendButtonContainer` id 标记的根容器。在我们将`sendButton `连接到容器上之后。

## 8。HTML

在`/webapp`文件夹中，我们有`Google_web_toolkit.html`文件。

我们可以用特定的 id 标记标签元素，这样框架就可以将它们绑定到 Java 对象中:

```java
<body>
    <h1>Sample GWT Application</h1>
    <table align="center">
        <tr>
            <td colspan="2" style="font-weight:bold;">Please enter your message:</td>
        </tr>
        <tr>
            <td id="nameFieldContainer"></td>
            <td id="sendButtonContainer"></td>
        </tr>
    </table>
</body>
```

带有`nameFieldContainer` 和`sendButtonContainer`id 的`<td>`标签将被映射到`Button`和`TextBox`组件。

## 9。主模块描述符

让我们看看`Google_web_toolkit.gwt.xml`主模块描述符文件的典型配置:

```java
<module rename-to='google_web_toolkit'>
    <inherits name='com.google.gwt.user.User'/>
    <inherits name='com.google.gwt.user.theme.clean.Clean'/>
    <entry-point class='com.baeldung.client.Google_web_toolkit'/>
</module>
```

**我们通过包含`com.google.gwt.user.User`接口**使核心的 GWT 内容可访问。此外，我们可以为我们的应用程序选择一个默认的样式表。这种情况下就是`*.clean.Clean`。

其他可用的样式选项有`*.dark.Dark`、`*.standard.Standard`、`*.chrome.Chrome`。这里的`com.baeldung.client.Google_web_toolkit `也标有`<entry-point />`标签。

## 10.添加事件处理程序

为了管理鼠标或键盘输入事件，GWT 将使用一些处理程序。**它们都是从`EventHandler`接口扩展而来，都有一个带事件类型参数**的方法。

在我们的例子中，我们注册了鼠标点击事件处理程序。

这将在每次按下按钮时触发`onClick()`方法:

```java
closeButton.addClickHandler(new ClickHandler() {
    public void onClick(ClickEvent event) {
        vPanel.hide();
        sendButton.setEnabled(true);
        sendButton.setFocus(true);
    }
});
```

在这里，我们可以修改小部件的状态和行为。在我们的例子中，我们隐藏了`vPanel`并启用了`sendButton`。

另一种方法是定义一个内部类并实现必要的接口:

```java
class MyHandler implements ClickHandler, KeyUpHandler {

    public void onClick(ClickEvent event) {
        // send message to the server
    }

    public void onKeyUp(KeyUpEvent event) {
        if (event.getNativeKeyCode() == KeyCodes.KEY_ENTER) {
            // send message to the server
        }
    }
}
```

除了`ClickHandler`，我们还在这里包含了`KeyUpHandler`接口来捕捉按键事件。在这里，`onKeyUp()` 方法中的**我们可以使用`KeyUpEvent`来检查用户是否按下了回车键**。

下面是我们如何使用`MyHandler`类注册两个事件处理程序:

```java
MyHandler handler = new MyHandler();
sendButton.addClickHandler(handler);
nameField.addKeyUpHandler(handler);
```

## 11.调用服务器

现在，我们准备将消息发送到服务器。我们将使用异步`sendMessage()`方法执行远程过程调用。

**该方法的第二个参数是`AsyncCallback<String>`接口，其中`String`是对应同步方法**的返回类型:

```java
messageServiceAsync.sendMessage(textToServer, new AsyncCallback<String>() {
    public void onFailure(Throwable caught) {
        serverResponseLabel.addStyleName("serverResponseLabelError");
        serverResponseLabel.setHTML("server error occurred");
        closeButton.setFocus(true);
    }

    public void onSuccess(String result) {
        serverResponseLabel.setHTML(result);
        vPanel.setVisible(true);
    }
});
```

我们可以看到，**接收器为每个响应类型**实现了 **`onSuccess(String result)`** **和`onFailure(Throwable)`** **方法。**

根据响应结果，我们或者设置错误消息“发生服务器错误”,或者在容器中显示结果值。

## 12。CSS 样式

当使用 eclipse 插件创建项目时，它会自动在`/webapp`目录下生成`Google_web_toolkit.css`文件，并将其链接到主 HTML 文件。

```java
<link type="text/css" rel="stylesheet" href="Google_web_toolkit.css">
```

当然，我们可以通过编程为特定的 UI 组件定义自定义样式:

```java
sendButton.addStyleName("sendButton");
```

这里我们用类名`sendButton`给我们的`sendButton`组件分配一个 CSS 样式:

```java
.sendButton {
    display: block;
    font-size: 16pt;
}
```

## 13.结果

结果，我们有了这个简单的 web 应用程序:

[![simpleApplication](img/95d08f873d99cf199eed42ebcd3a5225.png)](/web/20220626085848/https://www.baeldung.com/wp-content/uploads/2018/07/simpleApplication-300x259.png)

在这里，我们向服务器提交一个“Hi there”消息，并打印“Hello，Hi there！”屏幕上的回应。

## 14。结论

在这篇简短的文章中，我们了解了 GWT 框架的基础知识。之后，我们讨论了其 SDK 的架构、生命周期、功能和不同组件。

结果，我们学会了如何创建一个简单的 web 应用程序。

和往常一样，该教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220626085848/https://github.com/eugenp/tutorials/tree/master/google-web-toolkit)