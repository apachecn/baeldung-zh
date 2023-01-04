# Java 中的路由播放应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/routing-in-play>

## 1。概述

路由是一个常见的概念，出现在大多数 web 开发框架中，包括 [Spring MVC](https://web.archive.org/web/20220815030831/https://spring.io/) 。

路由是映射到处理程序的 URL 模式。处理程序可以是物理文件，比如 web 应用程序中的可下载资源，或者是处理请求的类，比如 MVC 应用程序中的控制器。

在本教程中，我们将探索使用 [Play 框架](https://web.archive.org/web/20220815030831/https://playframework.com/)开发 web 应用程序时的路由方面。

## 2。设置

首先，我们需要创建一个 Java Play 应用程序。关于如何在机器上设置 Play 框架的细节可以在我们的介绍文章中找到。

在设置结束时，我们应该有一个可以从浏览器访问的 Play 应用程序。

## 3。HTTP 路由

那么每当我们发送 HTTP 请求时，Play 是如何知道应该咨询哪个控制器的呢？这个问题的答案就在`app/conf/routes`配置文件中。

Play 的路由器把 HTTP 请求翻译成动作调用。 **HTTP 请求被认为是 MVC 架构**中的事件，路由器通过查询`routes`文件对它们做出反应，以执行哪个控制器和该控制器中的哪个动作。

这些事件中的每一个都为路由器提供了两个参数:带有查询字符串的请求路径和请求的 HTTP 方法。

## 4。带游隙的基本路由

为了让路由器完成它的工作，`conf/routes`文件必须定义 HTTP 方法和 URI 模式到适当的控制器动作的映射:

```java
GET     /     controllers.HomeController.index
GET     /     assets/*file controllers.Assets.versioned(path="/public", file: Asset)
```

所有路由文件还必须映射`play-routing/public`文件夹中对`/assets`端点上的客户端可用的静态资源。
注意定义 HTTP 路由的语法，以及 HTTP 方法`space` URI 模式`space`控制器动作。

## 5。URI 图案

在这一节中，我们将对 URI 模式进行一些阐述。

### 5.1。静态 URI 模式

上面的前三个 URI 模式是静态的。这意味着 URL 到资源的映射无需控制器动作中的任何进一步处理。

只要调用一个控制器方法，它就会返回一个静态资源，其内容在请求之前就已确定。

### 5.2。动态 URI 模式

上面的最后一个 URI 模式是动态的。这意味着服务于这些 URIs 上的请求的控制器动作需要来自请求的一些信息来确定响应。在上面的例子中，它需要一个文件名。

事件的正常顺序是路由器接收一个事件，从 URL 中选择路径，解码其数据段，并将它们传递给控制器。

然后将路径和查询参数作为参数注入控制器动作。我们将在接下来的部分用一个例子来演示这一点。

## 6。带间隙的高级路由

在本节中，我们将详细讨论使用动态 URI 模式进行路由的高级选项。

### 6.1。简单路径参数

简单路径参数是请求 URL 中未命名的参数，出现在主机和端口之后，并按照出现的顺序进行解析。

在`play-routing/app/HomeController.java`中，让我们创建一个新动作:

```java
public Result greet(String name) {
    return ok("Hello " + name);
}
```

我们希望能够从请求 URL 中选择一个路径参数，并将其映射到变量名。

路由器将从路由配置中获取这些值。

因此，让我们打开`play-routing/conf/routes`并为这个新动作创建一个映射:

```java
GET     /greet/:name     controllers.HomeController.greet(name: String)
```

请注意，我们如何用冒号语法通知路由器 name 是一个动态路径段，然后将它作为参数传递给 greet action 调用。

现在，让我们在浏览器中加载`http://locahost:9000/greet/john`，我们会被名字问候:

```java
Hello john
```

碰巧**如果我们的动作参数是字符串类型，我们可以在动作调用期间传递它而不指定参数类型**，尽管这对于其他类型是不同的。

让我们用年龄信息来丰富我们的`/greet`端点。

回到`HomeController`的问候动作，我们将其改为:

```java
public Result greet(String name, int age) {
    return ok("Hello " + name + ", you are " + age + " years old");
}
```

以及通往以下地点的路线:

```java
GET     /greet/:name/:age               controllers.HomeController.greet(name: String, age: Integer)
```

还要注意声明变量的 Scala 语法，`age: Integer`。在 Java 中，我们会使用`Integer age`语法。Play 框架是用 Scala 构建的。因此，有很多 scala 语法。

让我们载入`http://localhost:9000/greet/john/26`:

```java
Hello john, you are 26 years old
```

### 6.2。路径参数中的通配符

在我们的路由配置文件中，最后一个映射是:

```java
GET     /assets/*file  controllers.Assets.versioned(path="/public", file: Asset)
```

我们在路径的动态部分使用通配符。我们要告诉 Play 的是，在实际的请求中，无论什么值代替了`*file`，都应该作为一个整体来解析，而不是像其他情况下的路径参数那样解码。

在这个例子中，控制器是一个内置的控制器`Assets`，它允许客户端从`play-routing/public`文件夹下载文件。当我们加载`http://localhost:9000/asseimg/favicon.png`时，我们应该在浏览器中看到播放图标的图像，因为它出现在`/public/images`文件夹中。

让我们在`HomeController.java`中创建我们自己的示例动作:

```java
public Result introduceMe(String data) {
    String[] clientData = data.split(",");
    return ok("Your name is " + clientData[0] + ", you are " + clientData[1] + " years old");
}
```

请注意，在这个操作中，我们接收一个字符串参数，并应用我们的逻辑对其进行解码。在这种情况下，逻辑是将逗号分隔的`String`拆分成一个数组。以前，我们依靠路由器来解码这些数据。

有了通配符，我们只能靠自己了。我们希望客户机在传递数据时语法正确。理想情况下，**我们应该在使用它之前验证传入的字符串**。

让我们为此操作创建一条路线:

```java
GET   /*data   controllers.HomeController.introduceMe(data)
```

现在加载 URL `http://localhost:9000/john,26`。这将打印:

```java
Your name is john, you are 26 years old
```

### 6.3。路径参数中的正则表达式

就像通配符一样，我们可以对动态部分使用正则表达式。让我们添加一个接收数字并返回其平方的操作:

```java
public Result squareMe(Long num) {
    return ok(num + " Squared is " + (num * num));
}
```

现在我们将添加它的路线:

```java
GET   /square/$num<[0-9]+>   controllers.HomeController.squareMe(num:Long)
```

我们把这条路线放在`introduceMe`路线下面，引入一个新概念。在这种路由配置下，我们只能处理正则表达式部分为正整数的路由。

现在，如果我们已经按照上一段中的说明放置了路线，并且加载了`http://localhost:9000/square/2`，我们应该会看到一个`ArrayIndexOutOfBoundsException`:

[![play2](img/5d12870621eed9816d9a9e57f8e4de96.png)](/web/20220815030831/https://www.baeldung.com/wp-content/uploads/2016/10/play2.png)

如果我们检查服务器控制台中的错误日志，我们将意识到动作调用实际上是在`introduceMe`动作而不是`squareMe`动作上执行的。正如前面提到的通配符，我们是独立的，我们没有验证传入的数据。

使用字符串“`square/2`”调用了`introduceMe`方法，而不是逗号分隔的字符串。因此，在分割之后，我们得到了一个大小为 1 的数组。试图达到指标`1 `然后抛出异常。

自然，我们会期望调用被路由到`squareMe` 方法。为什么路由到`introduceMe`？原因是我们接下来要讨论的一个叫做`Routing Priority.`的戏剧特征

## 7。路由优先级

如果路线之间存在冲突，如`squareMe`和`introduceMe`之间的冲突，那么 **Play 选择声明顺序中的第一条路线**。

为什么会有冲突？因为通配符上下文路径`/*data`匹配除基本路径`/`之外的任何请求 URL。所以**URI 模式使用通配符的每条路由应该出现在顺序的最后**。

现在让我们改变路线的声明顺序，使`introduceMe`路线在`squareMe` 之后，然后重新加载:

```java
2 Squared is 4
```

为了测试正则表达式在路由中的能力，尝试加载`http://locahost:9000/square/-1`，路由器将无法匹配`squareMe`路由。相反，它将匹配`introduceMe,`，我们将再次获得`ArrayIndexOutOfBoundsException`。

这是因为`-1`与提供的正则表达式不匹配，任何字母字符也不匹配。

## 8。参数

到目前为止，我们已经介绍了在 routes 文件中声明参数类型的语法。

在这一节中，我们将看到在处理路径中的参数时有更多可用的选项。

### 8.1。具有固定值的参数

有时我们会希望为一个参数使用一个固定值。这是我们告诉 Play 使用所提供的路径参数的方式，或者如果请求上下文是路径`/`，则使用某个固定值。

另一种方式是让两个端点或上下文路径指向同一个控制器动作——一个端点需要来自请求 URL 的参数，如果缺少所述参数，则默认为另一个端点。

为了演示这一点，让我们向`HomeController`添加一个`writer()` 动作:

```java
public Result writer() {
    return ok("Routing in Play by Baeldung");
}
```

假设我们不总是希望我们的 API 返回一个`String`:

```java
Routing in Play by Baeldung
```

我们希望通过随请求一起发送文章作者的名字来控制它，只有当请求没有参数`author`时才默认为固定值`Baeldung`。

因此，让我们通过添加一个参数来进一步更改`writer`动作:

```java
public Result writer(String author) {
    return ok("REST API with Play by " + author);
}
```

让我们看看如何向路线添加固定值参数:

```java
GET     /writer           controllers.HomeController.writer(author = "Baeldung")
GET     /writer/:author   controllers.HomeController.writer(author: String)
```

注意我们现在有两条独立的路线都通向`HomeController.index`动作，而不是一条。

当我们现在从浏览器加载`http://localhost:9000/writer`时，我们得到:

```java
Routing in Play by Baeldung
```

当我们加载`http://localhost:9000/writer/john`时，我们得到:

```java
Routing in Play by john
```

### 8.2。带默认值的参数

除了有固定值之外，参数还可以有默认值。如果请求没有提供所需的值，两者都为控制器动作参数提供后备值。

两者的区别在于**固定值用作路径参数的后备，而默认值用作查询参数**的后备。

路径参数的形式为`http://localhost:9000/param1/param2`，查询参数的形式为`http://localhost:9000/?param1=value1&param2;=value2`。

第二个区别是在路由中声明两者的语法。固定值参数使用赋值运算符，如下所示:

```java
author = "Baeldung"
```

默认值使用不同类型的赋值:

```java
author ?= "Baeldung"
```

我们使用`?=`操作符，在发现`author`不包含值的情况下，该操作符有条件地将`Baeldung`分配给`author`。

为了有一个完整的演示，让我们创建`HomeController.writer`动作。比方说，除了作为路径参数的作者姓名之外，我们还想将作者`id`作为查询参数传递，如果在请求中没有传递，该参数将默认为`1`。

我们将把`writer`动作改为:

```java
public Result writer(String author, int id) {
    return ok("Routing in Play by: " + author + " ID: " + id);
}
```

和`writer`路线到:

```java
GET     /writer           controllers.HomeController.writer(author="Baeldung", id: Int ?= 1)
GET     /writer/:author   controllers.HomeController.writer(author: String, id: Int ?= 1)
```

现在加载`http://localhost:9000/writer` 我们看到:

```java
Routing in Play by: Baeldung ID: 1
```

点击`http://localhost:9000/writer?id=10` 给我们:

```java
Routing in Play by: Baeldung ID: 10
```

那`http://localhost:9000/writer/john`呢？

```java
Routing in Play by: john ID: 1
```

最后，`http://localhost:9000/writer/john?id=5 `返回:

```java
Routing in Play by: john ID: 5
```

## 9。结论

在本文中，我们探讨了 Play 应用程序中路由的概念。我们还有一篇关于[用 Play Framework](/web/20220815030831/https://www.baeldung.com/rest-api-with-play) 构建 RESTful API 的文章，其中本教程中的路由概念在一个实际例子中得到了应用。

像往常一样，本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220815030831/https://github.com/eugenp/tutorials/tree/master/play-framework)