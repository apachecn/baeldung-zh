# Java 游戏简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-intro-to-the-play-framework>

## 1。概述

本介绍教程的目的是探索 Play 框架，并弄清楚我们如何使用它创建 web 应用程序。

Play 是一个针对编程语言的高生产率 web 应用程序框架，其代码在 JVM 上编译和运行，主要是 Java 和 Scala。它集成了现代 web 应用程序开发所需的组件和 API。

## 2。游戏框架设置

让我们去 Play 框架的官方页面下载最新版本的发行版。本教程发布时，最新版本是 2.7。

我们将下载 Play Java Hello World 教程 zip 文件夹，并将文件解压缩到一个方便的位置。在这个文件夹的根目录下，我们会找到一个可以用来运行应用程序的`sbt`可执行文件。或者，我们可以从他们的[官方页面](https://web.archive.org/web/20220529024215/https://www.scala-sbt.org/)安装`sbt`。

要使用下载文件夹中的`sbt `,让我们执行以下操作:

```java
cd /path/to/folder/
./sbt run
```

注意，我们正在当前目录中运行一个脚本，因此使用了`./`语法。

如果我们安装了`sbt,` ，那么我们可以用它来代替:

```java
cd /path/to/folder/
sbt run
```

运行该命令后，我们将看到一条语句，内容为“(服务器已启动，使用 Enter 停止并返回控制台…)”。这意味着我们的应用程序已经准备好了，因此我们现在可以前往 [`http://localhost:9000`](https://web.archive.org/web/20220529024215/http://localhost:9000/) ，在那里我们将看到一个游戏欢迎页面:

[![](img/c5de8f8de33a977632aaaafd51c6af24.png)](/web/20220529024215/https://www.baeldung.com/wp-content/uploads/2016/10/play1.png)

## 3。游戏应用剖析

在这一节中，我们将更好地理解 Play 应用程序的结构以及该结构中每个文件和目录的用途。

如果你想马上挑战自己，举一个简单的例子，那么跳到下一节。

这些是我们在典型的 Play Framework 应用程序中找到的文件和文件夹:

```java
├── app                      → Application sources
│   ├── assets               → Compiled Asset sources
│   │   ├── javascripts      → Typically Coffee Script sources
│   │   └── stylesheets      → Typically LESS CSS sources
│   ├── controllers          → Application controllers
│   ├── models               → Application business layer
│   └── views                → Templates
├── build.sbt                → Application build script
├── conf                     → Configurations files and other non-compiled resources (on classpath)
│   ├── application.conf     → Main configuration file
│   └── routes               → Routes definition
├── dist                     → Arbitrary files to be included in your projects distribution
├── lib                      → Unmanaged libraries dependencies
├── logs                     → Logs folder
│   └── application.log      → Default log file
├── project                  → sbt configuration files
│   ├── build.properties     → Marker for sbt project
│   └── plugins.sbt          → sbt plugins including the declaration for Play itself
├── public                   → Public assets
│   ├── images               → Image files
│   ├── javascripts          → Javascript files
│   └── stylesheets          → CSS files
├── target                   → Generated files
│   ├── resolution-cache     → Information about dependencies
│   ├── scala-2.11            
│   │   ├── api              → Generated API docs
│   │   ├── classes          → Compiled class files
│   │   ├── routes           → Sources generated from routes
│   │   └── twirl            → Sources generated from templates
│   ├── universal            → Application packaging
│   └── web                  → Compiled web assets
└── test                     → source folder for unit or functional tests 
```

### 3.1.`app`目录

这个目录包含 Java 源代码、web 模板和编译资产的源代码——基本上是所有源代码和所有可执行资源。

`app`目录包含一些重要的子目录，每个子目录都封装了 MVC 架构模式的一部分:

*   这是应用程序业务层，这个包中的文件可能会模拟我们的数据库表，并使我们能够访问持久层
*   `views`–所有可以渲染到浏览器的 HTML 模板都包含在该文件夹中
*   `controllers`–包含控制器的子目录。`Controllers`是 Java 源文件，包含每个 API 调用要执行的动作。`Actions`是处理 HTTP 请求并返回与 HTTP 响应相同的结果的公共方法
*   `assets`–包含 CSS 和 javascript 等编译资源的子目录。上面的命名约定是灵活的，我们可以创建自己的包，例如一个`app/utils`包。我们也可以自定义包的命名`app/com/baeldung/controllers`

它还包含特定应用程序所需的可选文件和目录。

### 3.2.`public`目录

存储在`public`目录中的资源是静态资产，由 Web 服务器直接提供服务。

这个目录通常有三个子目录，分别存放图像、CSS 和 JavaScript 文件。建议我们像这样组织资产文件，以便在所有游戏应用程序中保持一致。

### 3.3.`conf`目录

`conf `目录包含应用程序配置文件。`application.conf `是我们放置 Play 应用程序的大多数配置属性的地方。我们将在`routes`为应用程序定义端点。

如果应用程序需要任何额外的配置文件，它们应该放在这个目录中。

### 3.4.`lib`目录

`lib`目录是可选的，包含非托管库依赖项。如果我们有任何没有在构建系统中指定的 jar，我们把它们放在这个目录中。它们将被自动添加到应用程序类路径中。

### 3.5.`build.sbt`文件

`build.sbt `文件是应用程序构建脚本。在这里，我们列出了运行应用程序所必需的依赖项，例如测试和持久性库。

### 3.6.`project`目录

基于 SBT 配置构建过程的所有文件都可以在`project` 目录中找到。

### 3.7.`target`目录

这个目录包含构建系统生成的所有文件——例如，所有的`.class`文件。

已经看到并研究了我们刚刚下载的 Play 框架 Hello World 示例的目录结构，现在我们可以通过一个示例来了解框架的基础。

## 4。简单的例子

在这一节中，我们将创建一个非常基本的 web 应用程序示例。我们将使用这个应用程序来熟悉 Play 框架的基础知识。

与其下载一个示例项目并以此为基础进行构建，**让我们看看另一种创建 Play Framework 应用程序的方法，使用`sbt new`命令**。

让我们打开命令提示符，导航到我们选择的位置，并执行以下命令:

```java
sbt new playframework/play-java-seed.g8
```

对于这一个，我们需要已经安装了第 2 节中解释的`sbt `。

上面的命令将首先提示我们输入项目的名称。接下来，它将询问将用于包的域(与 Java 中的包命名约定相反)。如果我们想保留方括号中给出的缺省值，我们可以按下`Enter`而不输入名称。

用这个命令生成的应用程序与前面生成的应用程序具有相同的结构。因此，我们可以像以前一样继续运行应用程序:

```java
cd /path/to/folder/ 
sbt run
```

上面的命令，执行完成后，**会在端口号`9000`上衍生出一个服务器来暴露我们的 API** ，我们可以通过`[http://localhost:9000](https://web.archive.org/web/20220529024215/http://localhost:9000/)`访问。我们应该会在浏览器中看到“欢迎玩”的消息。

我们的新 API 有两个端点，现在我们可以从浏览器中依次试用。第一个——我们刚刚加载的——是根端点，它加载一个带有“欢迎使用！”消息。

第二个在`http://localhost:9000/assets,`处，通过在路径中添加一个文件名来从服务器下载文件。我们可以通过在`[http://localhost:9000/asseimg/favicon.png](https://web.archive.org/web/20220529024215/http://localhost:9000/asseimg/favicon.png)`获取与应用程序一起下载的`favicon.png `文件来测试这个端点。

## 5。动作和控制器

控制器类中处理请求参数并产生发送给客户机的结果的 Java 方法称为动作。

控制器是一个扩展了`play.mvc.Controller`的 Java 类，它在逻辑上将可能与它们为客户端产生的结果相关的动作组合在一起。

现在让我们转向`app-parent-dir/app/controllers`并关注`HomeController.java`。

`HomeController`的索引操作返回一个带有简单欢迎消息的网页:

```java
public Result index() {
    return ok(views.html.index.render());
}
```

该网页是视图包中的默认`index`模板:

```java
@main("Welcome to Play") {
  <h1>Welcome to Play!</h1>
}
```

如上图所示，`index `页面调用了`main`模板。然后，主模板处理页面标题和正文标签的呈现。它需要两个参数:一个用于页面标题的`String `和一个用于插入页面主体的`Html `对象。

```java
@(title: String)(content: Html)

<!DOCTYPE html>
<html lang="en">
    <head>
        @* Here's where we render the page title `String`. *@
        <title>@title</title>
        <link rel="stylesheet" media="screen" href="@routes.Assets.versioned("stylesheets/main.css")">
        <link rel="shortcut icon" type="image/png" href="@routes.Assets.versioned("images/favicon.png")">
    </head>
    <body>
        @* And here's where we render the `Html` object containing
         * the page content. *@
        @content

        <script src="@routes.Assets.versioned("javascripts/main.js")" type="text/javascript"></script>
    </body>
</html>
```

让我们稍微修改一下`index`文件中的文本:

```java
@main("Welcome to Baeldung") {
  <h1>Welcome to Play Framework Tutorial on Baeldung!</h1>
}
```

重新加载浏览器会给我们一个粗体标题:

```java
Welcome to Play Framework Tutorial on Baeldung!
```

**我们可以通过删除`HomeController`的`index() `方法中的`render `指令来完全去掉模板，这样我们就可以直接返回纯文本或 HTML 文本:**

```java
public Result index() {
    return ok("REST API with Play by Baeldung");
}
```

编辑完代码后，如上所示，我们在浏览器中将只有文本。这将只是没有任何 HTML 或样式的纯文本:

```java
REST API with Play by Baeldung
```

我们也可以通过将文本包装在 header `<h1></h1>`标签中，然后将 HTML 文本传递给`Html.apply`方法来输出 HTML。你可以随意摆弄它。

让我们在`routes`中添加一个`/baeldung/html`端点:

```java
GET    /baeldung/html    controllers.HomeController.applyHtml
```

现在，让我们创建处理该端点上的请求的控制器:

```java
public Result applyHtml() {
    return ok(Html.apply("<h1>This text will appear as a heading 1</h1>"));
}
```

当我们访问`http://localhost:9000/baeldung/html`时，我们会看到上面的 HTML 格式的文本。

我们通过定制响应类型来操纵我们的响应。我们将在后面的小节中更详细地研究这个特性。

我们还看到了 Play 框架的另外两个重要特性。

首先，重新加载浏览器反映了我们代码的最新版本；那是因为我们的**代码变化是动态编译的。**

其次，Play 在`play.mvc.Results`类中为我们提供了标准 HTTP 响应的 helper 方法。一个例子是`ok()`方法，它返回一个 OK HTTP 200 响应以及我们作为参数传递给它的响应体。我们已经使用了在浏览器中显示文本的方法。

在`Results`类中有更多的帮助方法，如`notFound()`和`badRequest()`。

## 6。操作结果

我们已经从 Play 的内容协商功能中获益**，甚至没有意识到这一点。Play 自动从响应正文中推断响应内容类型。这就是我们能够在`ok` 方法中返回文本的原因:**

```java
return ok("text to display");
```

然后 Play 会自动将`Content-Type`头设置为`text/plain`。虽然这在大多数情况下是可行的，但是我们可以接管控制权并定制内容类型头。

让**自定义`HomeController.customContentType`动作对`text/html`** 的响应:

```java
public Result customContentType() {
    return ok("This is some text content").as("text/html");
}
```

这种模式适用于所有类型的内容。根据我们传递给`ok` helper 方法的数据格式，我们可以用`text/plain`或`application/json`替换`text/html`。

我们可以做一些类似于设置标题的事情:

```java
public Result setHeaders() {
    return ok("This is some text content")
            .as("text/html")
            .withHeader("Header-Key", "Some value");
}
```

## 7。结论

在本文中，我们探索了 Play 框架的基础。我们还能够使用 Play 创建一个基本的 Java web 应用程序。

像往常一样，本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220529024215/https://github.com/eugenp/tutorials/tree/master/play-framework)