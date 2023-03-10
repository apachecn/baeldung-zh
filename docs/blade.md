# 刀锋——一本完整的指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/blade>

## 1。概述

[Blade](https://web.archive.org/web/20221208143837/https://lets-blade.com/) 是一个小型的 Java 8+ MVC 框架，从头开始构建，心中有一些明确的目标:独立、高效、优雅、直观、超快。

许多不同的框架启发了它的设计:Node 的 [Express](https://web.archive.org/web/20221208143837/https://expressjs.com/) ，Python 的 [Flask](https://web.archive.org/web/20221208143837/http://flask.pocoo.org/) ，Golang 的 [Macaron](https://web.archive.org/web/20221208143837/https://github.com/go-macaron/macaron) / [Martini](https://web.archive.org/web/20221208143837/https://github.com/go-martini/martini) 。

刀锋也是一个雄心勃勃的大项目的一部分，[让我们刀锋](https://web.archive.org/web/20221208143837/https://github.com/lets-blade)。它包括其他小型库的异构集合，从 Captcha 生成到 JSON 转换，从模板到简单的数据库连接。

然而，在本教程中，我们将只关注 MVC。

## 2。入门

首先，让我们创建一个空的 Maven 项目，并在`pom.xml`中添加[最新的刀片 MVC 依赖关系](https://web.archive.org/web/20221208143837/https://search.maven.org/search?q=g:com.bladejava%20AND%20a:blade-mvc):

```java
<dependency>
    <groupId>com.bladejava</groupId>
    <artifactId>blade-mvc</artifactId>
    <version>2.0.14.RELEASE</version>
</dependency> 
```

### 2.1。捆绑刀片应用

由于我们的应用程序将被创建为一个 JAR，它不会像在战争中一样有一个`/lib`文件夹。结果，这就导致了如何为我们的应用程序提供`blade-mvc` JAR 以及我们可能需要的任何其他 JAR 的问题。

不同的方法各有利弊，在[如何用 Maven](/web/20221208143837/https://www.baeldung.com/executable-jar-with-maven) 创建可执行的 JAR 教程中有解释。

为了简单起见，**我们将使用`Maven Assembly Plugin`技术**，它分解在`pom.xml`中导入的任何 JAR，然后将所有的类捆绑在一个超级 JAR 中。

### 2.2。运行刀片应用

**Blade 基于[Netty](/web/20221208143837/https://www.baeldung.com/netty)T3，一个令人惊叹的异步事件驱动的网络应用框架。因此，要运行我们基于刀片的应用程序，我们不需要任何外部应用服务器或 Servlet 容器；JRE 就足够了:**

```java
java -jar target/sample-blade-app.jar 
```

之后，可以在`http://localhost:9000` URL 访问该应用程序。

## 3。了解架构

Blade 的体系结构非常简单:

[![architecture](img/3c5502e1a41fb2e2327b91b0c29be168.png)](/web/20221208143837/https://www.baeldung.com/wp-content/uploads/2019/01/architecture.png)

它总是遵循相同的生命周期:

1.  Netty 收到一个请求
2.  执行中间件(可选)
3.  执行 WebHooks(可选)
4.  执行路由
5.  响应被发送到客户端
6.  清除

我们将在接下来的章节中探讨上述函数。

## 4。路由

简而言之，MVC 中的路由是用于创建 URL 和控制器之间的绑定的机制。

Blade 提供了两种类型的路线:基本路线和带注释的路线。

### 4.1。基本路线

基本路线适用于非常小的软件，如微服务或最小的 web 应用程序:

```java
Blade.of()
  .get("/basic-routes-example", ctx -> ctx.text("GET called"))
  .post("/basic-routes-example", ctx -> ctx.text("POST called"))
  .put("/basic-routes-example", ctx -> ctx.text("PUT called"))
  .delete("/basic-routes-example", ctx -> ctx.text("DELETE called"))
  .start(App.class, args); 
```

用于注册路由的方法的名称对应于将用于转发请求的 HTTP 谓词。就这么简单。

在这种情况下，我们返回一个文本，但我们也可以呈现页面，我们将在本教程的后面看到。

### 4.2。带注释的路线

当然，对于更现实的用例，我们可以使用注释来定义我们需要的所有路线。我们应该为此使用单独的类。

首先，我们需要通过`@Path`注释创建一个控制器，启动时会被 Blade 扫描。

然后，我们需要使用与我们想要拦截的 HTTP 方法相关的路由注释:

```java
@Path
public class RouteExampleController {    

    @GetRoute("/routes-example") 
    public String get(){ 
        return "get.html"; 
    }

    @PostRoute("/routes-example") 
    public String post(){ 
        return "post.html"; 
    }

    @PutRoute("/routes-example") 
    public String put(){ 
        return "put.html"; 
    }

    @DeleteRoute("/routes-example") 
    public String delete(){ 
        return "delete.html"; 
    }
} 
```

我们还可以使用简单的`@Route`注释并将 HTTP 方法指定为参数:

```java
@Route(value="/another-route-example", method=HttpMethod.GET) 
public String anotherGet(){ 
    return "get.html" ; 
} 
```

另一方面，**如果我们不输入任何方法参数，路由将拦截每个对 URL** 的 HTTP 调用，不管是哪个动词。

### 4.3。参数注入

有几种方法可以将参数传递给我们的路由。让我们用文档中的一些例子[来探究它们。](https://web.archive.org/web/20221208143837/https://lets-blade.com/docs/route.html#%E5%8F%82%E6%95%B0%E6%B3%A8%E5%85%A5)

*   表单参数:

```java
@GetRoute("/home")
public void formParam(@Param String name){
    System.out.println("name: " + name);
} 
```

*   Restful 参数:

```java
@GetRoute("/users/:uid")
public void restfulParam(@PathParam Integer uid){
    System.out.println("uid: " + uid);
} 
```

*   文件上传参数:

```java
@PostRoute("/upload")
public void fileParam(@MultipartParam FileItem fileItem){
    byte[] file = fileItem.getData();
} 
```

*   标题参数:

```java
@GetRoute("/header")
public void headerParam(@HeaderParam String referer){
    System.out.println("Referer: " + referer);
} 
```

*   Cookie 参数:

```java
@GetRoute("/cookie")
public void cookieParam(@CookieParam String myCookie){
    System.out.println("myCookie: " + myCookie);
} 
```

*   身体参数:

```java
@PostRoute("/bodyParam")
public void bodyParam(@BodyParam User user){
    System.out.println("user: " + user.toString());
} 
```

*   值对象参数，通过将其属性发送到路由来调用:

```java
@PostRoute("/voParam")
public void voParam(@Param User user){
    System.out.println("user: " + user.toString());
} 
```

```java
<form method="post">
    <input type="text" name="age"/>
    <input type="text" name="name"/>
</form> 
```

## 5。静态资源

如果需要，Blade 还可以提供静态资源，只需将它们放在`/resources/static`文件夹中。

例如，`src/main/resources/static/app.css`将在`http://localhost:9000/static/app.css`可用。

### 5.1。定制路径

我们可以通过以编程方式添加一个或多个静态路径来调整这种行为:

```java
blade.addStatics("/custom-static"); 
```

通过编辑文件`src/main/resources/application.properties`，可以通过配置获得相同的结果:

```java
mvc.statics=/custom-static 
```

### 5.2。启用资源列表

我们可以允许列出静态文件夹的内容，出于安全原因，这个特性在默认情况下是关闭的:

```java
blade.showFileList(true); 
```

或者在配置中:

```java
mvc.statics.show-list=true 
```

我们现在可以打开`http://localhost:9000/custom-static/`来显示文件夹的内容。

### 5.3。使用 WebJars

正如在[web jars 简介](/web/20221208143837/https://www.baeldung.com/maven-webjars)教程中看到的，打包成 JAR 的静态资源也是一个可行的选择。

刀片在`/webjars/`路径下自动曝光。

例如，让我们在`pom.xml`中导入[引导程序](https://web.archive.org/web/20221208143837/https://search.maven.org/search?q=g:org.webjars%20AND%20a:bootstrap):

```java
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>4.2.1</version>
</dependency> 
```

因此，它将在`http://localhost:9000/webjars/bootstrap/4.2.1/css/bootstrap.css`下可用

## 6。HTTP 请求

由于**刀片不是基于 Servlet 规范**，像它的接口`[Request](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade/blob/master/src/main/java/com/blade/mvc/http/Request.java)`和它的类`[HttpRequest](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade/blob/master/src/main/java/com/blade/mvc/http/HttpRequest.java)`这样的对象与我们习惯的略有不同。

### 6.1。表单参数

在读取表单参数时，Blade 在查询方法的结果中充分利用了 Java 的`Optional`(下面所有方法都返回一个`Optional`对象):

*   `query(String name)`
*   `queryInt(String name)`
*   `queryLong(String name)`
*   `queryDouble(String name)`

它们还有一个后备值:

*   `String query(String name, String defaultValue)`
*   `int queryInt(String name, int defaultValue)`
*   `long queryLong(String name, long defaultValue)`
*   `double queryDouble(String name, double defaultValue)`

我们可以通过 automapped 属性读取表单参数:

```java
@PostRoute("/save")
public void formParams(@Param String username){
    // ...
} 
```

或者来自`Request`对象:

```java
@PostRoute("/save")
public void formParams(Request request){
    String username = request.query("username", "Baeldung");
} 
```

### 6.2。JSON 数据

现在让我们看看如何将 JSON 对象映射到 POJO:

```java
curl -X POST http://localhost:9000/users -H 'Content-Type: application/json' \ 
  -d '{"name":"Baeldung","site":"baeldung.com"}' 
```

POJO(为了可读性，用 [Lombok](/web/20221208143837/https://www.baeldung.com/intro-to-project-lombok) 标注):

```java
public class User {
    @Getter @Setter private String name;
    @Getter @Setter private String site;
} 
```

同样，该值可作为注入属性使用:

```java
@PostRoute("/users")
public void bodyParams(@BodyParam User user){
    // ...
} 
```

并且从`Request`开始:

```java
@PostRoute("/users")
public void bodyParams(Request request) {
    String bodyString = request.bodyToString();
} 
```

### 6.3。RESTful 参数

像`localhost:9000/user/42`这样漂亮的 URL 中的 RESTFul 参数也是一等公民:

```java
@GetRoute("/user/:id")
public void user(@PathParam Integer id){
    // ...
} 
```

像往常一样，我们可以在需要时依靠`Request`对象:

```java
@GetRoute("/user")
public void user(Request request){
    Integer id = request.pathInt("id");
} 
```

显然，同样的方法也适用于`Long`和`String`类型。

### 6.4。数据绑定

Blade 支持 JSON 和表单绑定参数，并自动将它们附加到模型对象:

```java
@PostRoute("/users")
public void bodyParams(User user){} 
```

### 6.5。请求和会话属性

在`Request`和`Session`中读写对象的 API 非常清楚。

带有两个参数的方法，分别代表键和值，是我们可以用来在不同的上下文中存储值的赋值函数:

```java
Session session = request.session();
request.attribute("request-val", "Some Request value");
session.attribute("session-val", 1337); 
```

另一方面，只接受 key 参数的相同方法是访问器:

```java
String requestVal = request.attribute("request-val");
String sessionVal = session.attribute("session-val"); //It's an Integer 
```

一个有趣的特性是[它们的泛型返回类型< T > T](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade/blob/master/src/main/java/com/blade/mvc/http/Request.java#L500) ，这让我们不必对结果进行强制转换。

### 6.6。标题

相反，请求头只能从请求中读取:

```java
String header1 = request.header("a-header");
String header2 = request.header("a-safe-header", "with a default value");
Map<String, String> allHeaders = request.headers(); 
```

### 6.7。公用事业

以下实用程序方法也是现成可用的，它们非常明显，不需要进一步解释:

*   `boolean isIE()`
*   `boolean isAjax()`
*   `String contentType()`
*   `String userAgent()`

### 6.8。读取 cookie

让我们看看`Request`对象如何帮助我们处理 Cookies，特别是在读取`Optional<Cookie>`时:

```java
Optional<Cookie> cookieRaw(String name); 
```

如果 Cookie 不存在，我们也可以通过指定要应用的默认值来获取它作为一个`String`:

```java
String cookie(String name, String defaultValue); 
```

最后，这是我们如何一次读取所有 cookie 的方法(`keys`是 cookie 的名称，`values`是 cookie 的值):

```java
Map<String, String> cookies = request.cookies(); 
```

## 7。HTTP 响应

类似于对`Request`所做的，我们可以通过简单地将它声明为路由方法的一个参数来获得对`Response`对象的引用:

```java
@GetRoute("/")
public void home(Response response) {} 
```

### 7.1。简单输出

我们可以通过一种方便的输出方法向调用者发送一个简单的输出，以及 200 HTTP 代码和适当的内容类型。

首先，我们可以发送纯文本:

```java
response.text("Hello World!");
```

其次，我们可以制作一个 HTML:

```java
response.html("<h1>Hello World!</h1>");
```

第三，我们同样可以生成一个 XML:

```java
response.xml("<Msg>Hello World!</Msg>");
```

最后，我们可以使用一个`String`输出 JSON:

```java
response.json("{\"The Answer\":42}"); 
```

甚至在 POJO 中，利用自动 JSON 转换:

```java
User user = new User("Baeldung", "baeldung.com"); 
response.json(user); 
```

### 7.2。文件输出

从服务器下载文件再简单不过了:

```java
response.download("the-file.txt", "/path/to/the/file.txt"); 
```

第一个参数设置将要下载的文件的名称，而第二个参数(一个`File`对象，这里用一个`String`构造)表示服务器上实际文件的路径。

### 7.3。模板渲染

Blade 还可以通过模板引擎呈现页面:

```java
response.render("admin/users.html"); 
```

模板的默认目录是`src/main/resources/templates/`，因此前面的一行程序将查找文件`src/main/resources/templates/admin/users.html`。

我们将在后面的**模板**部分了解更多。

### 7.4。重定向

重定向意味着向浏览器发送一个 302 HTTP 代码，以及一个 URL，随后是第二个 GET。

我们可以重定向到另一个路由，或者也可以重定向到外部 URL:

```java
response.redirect("/target-route"); 
```

### 7.5。写 cookie

在这一点上，我们应该习惯于 Blade 的简单性。因此，让我们看看如何用一行代码编写一个未过期的 Cookie:

```java
response.cookie("cookie-name", "Some value here"); 
```

事实上，删除 Cookie 同样简单:

```java
response.removeCookie("cookie-name"); 
```

### 7.6。其他操作

最后，`Response`对象为我们提供了几个其他的方法来执行操作，比如写标题、设置内容类型、设置状态代码等等。

让我们快速看一下其中的一些:

*   `Response status(int status)`
*   `Map headers()`
*   `Response notFound()`
*   `Map cookies()`
*   `Response contentType(String contentType)`
*   `void body(@NonNull byte[] data)`
*   `Response header(String name, String value)`

## 8。WebHooks

WebHook 是一个拦截器，通过它我们可以在路由方法执行之前和之后运行代码。

我们可以通过简单地实现`WebHook`函数接口并覆盖`before()`方法来创建一个 WebHook:

```java
@FunctionalInterface
public interface WebHook {

    boolean before(RouteContext ctx);

    default boolean after(RouteContext ctx) {
        return true;
    }
} 
```

正如我们所看到的，`after()`是一个默认的方法，因此我们只在需要的时候覆盖它。

### 8.1。拦截每个请求

`@Bean`注释告诉框架用 IoC 容器扫描类。

用它注释的 WebHook 将因此全局工作，拦截对每个 URL 的请求:

```java
@Bean
public class BaeldungHook implements WebHook {

    @Override
    public boolean before(RouteContext ctx) {
        System.out.println("[BaeldungHook] called before Route method");
        return true;
    }
} 
```

### 8.2。缩小到一个网址

我们还可以拦截特定的 URL，只围绕这些路由方法执行代码:

```java
Blade.of()
  .before("/user/*", ctx -> System.out.println("Before: " + ctx.uri()));
  .start(App.class, args); 
```

### 8.3。中间件

中间件是优先的 WebHook，它在任何标准的 web hook 之前执行:

```java
public class BaeldungMiddleware implements WebHook {

    @Override
    public boolean before(RouteContext context) {
        System.out.println("[BaeldungMiddleware] called before Route method and other WebHooks");
        return true;
    }
} 
```

它们只需要在没有`@Bean`注释的情况下定义，然后通过`use()`声明性地注册:

```java
Blade.of()
  .use(new BaeldungMiddleware())
  .start(App.class, args); 
```

此外，Blade 附带了以下与安全相关的内置中间件，它们的名称不言自明:

*   `[BasicAuthMiddleware](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade/blob/master/src/main/java/com/blade/security/web/auth/BasicAuthMiddleware.java)`
*   [T2`CorsMiddleware`](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade/blob/master/src/main/java/com/blade/security/web/cors/CorsMiddleware.java)
*   [T2`XssMiddleware`](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade/blob/master/src/main/java/com/blade/security/web/xss/XssMiddleware.java)
*   [T2`CsrfMiddleware`](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade/blob/master/src/main/java/com/blade/security/web/csrf/CsrfMiddleware.java)

## 9。配置

**在刀片式服务器中，配置完全是可选的，因为按照惯例，一切都是开箱即用的。**然而，我们可以自定义默认设置，并在`src/main/resources/application.properties`文件中引入新的属性。

### 9.1。读取配置

我们可以用不同的方式读取配置，如果设置不可用，可以指定或不指定默认值。

*   启动期间:

```java
Blade.of()
  .on(EventType.SERVER_STARTED, e -> {
      Optional<String> version = WebContext.blade().env("app.version");
  })
  .start(App.class, args); 
```

*   在路线内:

```java
@GetRoute("/some-route")
public void someRoute(){
    String authors = WebContext.blade().env("app.authors","Unknown authors");
} 
```

*   在自定义加载器中，通过实现`BladeLoader`接口，覆盖`load()`方法，并用`@Bean`注释该类:

```java
@Bean
public class LoadConfig implements BladeLoader {

    @Override
    public void load(Blade blade) {
        Optional<String> version = WebContext.blade().env("app.version");
        String authors = WebContext.blade().env("app.authors","Unknown authors");
    }
} 
```

### 9.2。配置属性

已经配置好但准备定制的几个设置按类型分组，并在三列表格(名称、描述、默认值)中的此地址处列出[。我们也可以参考翻译的页面，注意翻译错误地将设置名称大写。真正的设置是完全小写的。](https://web.archive.org/web/20221208143837/https://lets-blade.com/docs/configuration.html#%E9%85%8D%E7%BD%AE%E6%B8%85%E5%8D%95)

按前缀对配置设置进行分组，可以使它们一次全部显示在一个映射中，这在有许多配置设置时非常有用:

```java
Environment environment = blade.environment();
Map<String, Object> map = environment.getPrefix("app");
String version = map.get("version").toString();
String authors = map.get("authors","Unknown authors").toString(); 
```

### 9.3。处理多种环境

当将我们的应用程序部署到不同的环境时，我们可能需要指定不同的设置，例如与数据库连接相关的设置。Blade 为我们提供了一种为不同环境配置应用的方式，而不是手动替换`application.properties`文件。我们可以简单地保留所有开发设置的`application.properties`，然后**在同一个文件夹中创建其他文件，比如`application-prod.properties`，只包含与**不同的设置。

在启动过程中，我们可以指定我们想要使用的环境，框架将通过使用来自`application-prod.properties`的最具体的设置和来自默认`application.properties`文件的所有其他设置来合并文件:

```java
java -jar target/sample-blade-app.jar --app.env=prod 
```

## 10。模板

Blade 中的模板是一个模块化的方面。虽然它集成了一个非常基本的模板引擎，但是对于视图的任何专业使用，我们都应该依赖外部模板引擎。然后**我们可以从 GitHub 上的[刀片-模板-引擎](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade-template-engines)库中可用的**中选择一个引擎，它们是 **FreeMarker** 、 **Jetbrick** 、 **Pebble** 和 **Velocity** ，或者甚至创建一个包装器来导入另一个我们喜欢的模板。

Blade 的作者建议使用另一个聪明的中国项目 [Jetbrick](https://web.archive.org/web/20221208143837/https://search.maven.org/search?q=g:com.github.subchen%20AND%20a:jetbrick-template) 。

### 10.1。使用默认引擎

默认模板通过`${}`符号解析不同上下文中的变量:

```java
<h1>Hello, ${name}!</h1> 
```

### 10.2。插入外部引擎

切换到不同的模板引擎是一件轻而易举的事！我们只需导入引擎(的刀片包装器)的依赖关系:

```java
<dependency>
    <groupId>com.bladejava</groupId>
    <artifactId>blade-template-jetbrick</artifactId>
    <version>0.1.3</version>
</dependency> 
```

此时，只需编写一个简单的配置来指示框架使用该库就足够了:

```java
@Bean
public class TemplateConfig implements BladeLoader {

    @Override
    public void load(Blade blade) {
        blade.templateEngine(new JetbrickTemplateEngine());
    }
} 
```

因此，`src/main/resources/templates/`下的每个文件都将被新的引擎解析，其语法超出了本教程的范围。

### 10.3。包装新发动机

包装一个新的模板引擎需要创建一个类，这个类必须实现`TemplateEngine`接口并覆盖`render()`方法:

```java
void render (ModelAndView modelAndView, Writer writer) throws TemplateException; 
```

出于这个目的，我们可以看一看实际 Jetbrick 包装器的代码来了解这意味着什么。

## 11。记录日志

Blade 使用`slf4j-api`作为测井接口。

它还包括一个已经配置好的日志实现，名为 [`blade-log`](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade-log) 。因此，我们不需要导入任何东西；它的工作方式是，简单地定义一个`Logger`:

```java
private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class); 
```

### 11.1。定制集成记录器

如果我们想要修改默认配置，我们需要调整以下参数作为系统属性:

*   日志记录级别(可以是“跟踪”、“调试”、“信息”、“警告”或“错误”):

```java
# Root Logger
com.blade.logger.rootLevel=info

# Package Custom Logging Level
com.blade.logger.somepackage=debug

# Class Custom Logging Level
com.blade.logger.com.baeldung.sample.SomeClass=trace 
```

*   显示的信息:

```java
# Date and Time
com.blade.logger.showDate=false

# Date and Time Pattern
com.blade.logger.datePattern=yyyy-MM-dd HH:mm:ss:SSS Z

# Thread Name
com.blade.logger.showThread=true

# Logger Instance Name
com.blade.logger.showLogName=true

# Only the Last Part of FQCN
com.blade.logger.shortName=true 
```

*   记录器:

```java
# Path 
com.blade.logger.dir=./logs

# Name (it defaults to the current app.name)
com.blade.logger.name=sample 
```

### 11.2。不包括集成记录器

尽管已经配置了一个集成的日志记录器对于开始我们的小项目来说非常方便，但是我们可能很容易陷入其他库导入它们自己的日志实现的情况。在这种情况下，为了避免冲突，我们可以删除集成模块:

```java
<dependency>
    <groupId>com.bladejava</groupId>
    <artifactId>blade-mvc</artifactId>
    <version>${blade.version}</version>
    <exclusions>
        <exclusion>
            <groupId>com.bladejava</groupId>
            <artifactId>blade-log</artifactId>
        </exclusion>
    </exclusions>
</dependency> 
```

## 12。定制

### 12.1。自定义异常处理

框架中还默认内置了一个异常处理程序。它将异常打印到控制台，如果`app.devMode`是`true`，那么堆栈跟踪也在网页上可见。

然而，我们可以通过定义一个扩展了`DefaultExceptionHandler`类的`@Bean`来以特定的方式处理异常:

```java
@Bean
public class GlobalExceptionHandler extends DefaultExceptionHandler {

    @Override
    public void handle(Exception e) {
        if (e instanceof BaeldungException) {
            BaeldungException baeldungException = (BaeldungException) e;
            String msg = baeldungException.getMessage();
            WebContext.response().json(RestResponse.fail(msg));
        } else {
            super.handle(e);
        }
    }
} 
```

### 12.2。自定义错误页面

类似地，错误`404 – Not Found`和`500 – Internal Server Error`通过瘦缺省页面处理。

我们可以通过使用以下设置在`application.properties`文件中声明页面来强制框架使用我们自己的页面:

```java
mvc.view.404=my-404.html
mvc.view.500=my-500.html 
```

当然，那些 HTML 页面必须放在`src/main/resources/templates`文件夹下。

在 500 中，我们还可以通过它们的特殊变量检索异常`message`和`stackTrace`:

```java
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>500 Internal Server Error</title>
    </head>
    <body>
        <h1> Custom Error 500 Page </h1>
        <p> The following error occurred： "<strong>${message}</strong>"</p>
        <pre> ${stackTrace} </pre>
    </body>
</html>
```

## 13。预定任务

该框架的另一个有趣的特性是调度方法执行的可能性。

这可以通过用`@Schedule`注释来注释`@Bean`类的方法来实现:

```java
@Bean
public class ScheduleExample {

    @Schedule(name = "baeldungTask", cron = "0 */1 * * * ?")
    public void runScheduledTask() {
        System.out.println("This is a scheduled Task running once per minute.");
    }
} 
```

事实上，它使用经典的 cron 表达式来指定`DateTime`坐标。我们可以在[克朗表达指南](/web/20221208143837/https://www.baeldung.com/cron-expressions)中读到更多。

稍后，我们可能会利用 [`TaskManager`](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade/blob/master/src/main/java/com/blade/task/TaskManager.java) 类的静态方法来执行预定任务的操作。

*   获取所有计划任务:

```java
List<Task> allScheduledTasks = TaskManager.getTasks(); 
```

*   按名称获取任务:

```java
Task myTask = TaskManager.getTask("baeldungTask"); 
```

*   按名称停止任务:

```java
boolean closed = TaskManager.stopTask("baeldungTask"); 
```

## 14。事件

正如在 9.1 节中已经看到的，在运行一些定制代码之前监听一个指定的事件是可能的。

Blade 提供了以下现成的事件:

```java
public enum EventType {
    SERVER_STARTING,
    SERVER_STARTED,
    SERVER_STOPPING,
    SERVER_STOPPED,
    SESSION_CREATED,
    SESSION_DESTROY,
    SOURCE_CHANGED,
    ENVIRONMENT_CHANGED
} 
```

虽然前六个很容易猜测，但后两个需要一些提示:`ENVIRONMENT_CHANGED`允许我们在服务器启动时，如果配置文件发生更改，就执行一个操作。相反，`SOURCE_CHANGED`尚未实现，仅供将来使用。

让我们看看如何在创建会话时给它赋值:

```java
Blade.of()
  .on(EventType.SESSION_CREATED, e -> {
      Session session = (Session) e.attribute("session");
      session.attribute("name", "Baeldung");
  })
  .start(App.class, args); 
```

## 15。会话实现

说到会话，它的默认实现将会话值存储在内存中。

因此，我们可能希望切换到不同的实现来提供缓存、持久性或其他功能。就拿 Redis 来说吧。我们首先需要通过实现`[Session](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade/blob/master/src/main/java/com/blade/mvc/http/Session.java)`接口来创建我们的`RedisSession`包装器，如`HttpSession` 的文档中的[所示。](https://web.archive.org/web/20221208143837/https://lets-blade.com/docs/session-ext.html)

然后，只需要让框架知道我们想要使用它。我们可以用与定制模板引擎相同的方式来实现这一点，唯一的区别是我们调用了`sessionType()`方法:

```java
@Bean
public class SessionConfig implements BladeLoader {

    @Override
    public void load(Blade blade) {
        blade.sessionType(new RedisSession());
    }
} 
```

## 16。命令行参数

当从命令行运行 Blade 时，我们可以指定三个设置来改变它的行为。

首先，我们可以更改 IP 地址，默认情况下是本地`0.0.0.0`回环:

```java
java -jar target/sample-blade-app.jar --server.address=192.168.1.100 
```

其次，我们还可以更改端口，默认为`9000`:

```java
java -jar target/sample-blade-app.jar --server.port=8080 
```

最后，如第 9.3 节所示，我们可以改变环境，让不同的`application-XXX.properties`文件覆盖默认文件`application.properties`:

```java
java -jar target/sample-blade-app.jar --app.env=prod 
```

## 17。在 IDE 中运行

任何现代的 Java IDE 都能够运行一个 Blade 项目，甚至不需要 Maven 插件。在 IDE 中运行 Blade 在运行 [Blade 演示](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade-demos)时特别有用，这些示例是为了展示框架的功能而专门编写的。它们都继承了一个父 pom，所以让 IDE 来完成这项工作比手动调整它们作为独立的应用程序来运行更容易。

### 17.1。日食

在 Eclipse 中，右键单击项目并启动`Run as Java Application`，选择我们的`App`类，然后按`OK`就足够了。

然而，Eclipse 的控制台不能正确显示 ANSI 颜色，而是倒出它们的代码:

[![eclipse workspace sample app - Eclipse IDE](img/e5c69a6883e728704b24cb35b9c37b75.png)](/web/20221208143837/https://www.baeldung.com/wp-content/uploads/2019/01/Screenshot-from-2019-01-08-23-43-10.png)

幸运的是，在控制台扩展中安装 [ANSI Escape 永久地解决了这个问题:](https://web.archive.org/web/20221208143837/https://marketplace.eclipse.org/content/ansi-escape-console)

[![eclipse workspace - Eclipse IDE](img/b228ee9fbe0d7ea54cd39d80408c742e.png)](/web/20221208143837/https://www.baeldung.com/wp-content/uploads/2019/01/Screenshot-from-2019-01-08-23-44-25.png)

### 17.2。IntelliJ IDEA

IntelliJ IDEA 开箱即可使用 ANSI 颜色。所以创建项目，右键点击`App`文件，启动`Run ‘App.main()'`(相当于按下`Ctrl+Shift+F10`)就可以了:

[![sample blade app](img/671f8c9fb75767114508c3f0fec4276f.png)](/web/20221208143837/https://www.baeldung.com/wp-content/uploads/2019/01/Screenshot-from-2019-01-12-00-44-01.png)

### 17.3。Visual Studio 代码

通过预先安装 [Java 扩展包](https://web.archive.org/web/20221208143837/https://code.visualstudio.com/docs/java/extensions)，也可以使用 VSCode，这是一个流行的非以 Java 为中心的 IDE。

按下`Ctrl+F5`将运行项目:

[![pom.xml sample app](img/e8d92c96124bd92072562c332604ac03.png)](/web/20221208143837/https://www.baeldung.com/wp-content/uploads/2019/01/Screenshot-from-2019-01-12-00-59-43.png)

## 18。结论

我们已经看到了如何使用 Blade 创建一个小型的 MVC 应用程序。

[整个文档](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade)只有中文版本。尽管主要在中国广泛传播，但由于[的中文起源](https://web.archive.org/web/20221208143837/https://lets-blade.com/)，[，作者](https://web.archive.org/web/20221208143837/https://github.com/hellokaton)最近在 [GitHub](https://web.archive.org/web/20221208143837/https://github.com/lets-blade/blade) 上翻译了 API 并用英语记录了项目的核心功能。

和往常一样，我们可以在 GitHub 上找到这个例子的源代码。