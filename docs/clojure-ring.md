# 用 Ring 编写 Clojure Webapps

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/clojure-ring>

## 1。简介

**Ring 是一个用 Clojure** 编写 web 应用的库。它支持编写功能全面的 web 应用程序所需的一切，并且拥有一个蓬勃发展的生态系统，使它更加强大。

在本教程中，我们将介绍 Ring，并展示一些我们可以用它实现的事情。

与许多现代工具包一样，Ring 不是一个为创建 REST APIs 而设计的框架。**它是一个处理 HTTP 请求的底层框架**，主要用于传统的 web 开发。然而，一些库在它的基础上构建，以支持许多其他期望的应用程序结构。

## 2。依赖性

在开始使用 Ring 之前，我们需要将它添加到我们的项目中。**我们需要的最小依赖性是**:

*   [T2`ring/ring-core`](https://web.archive.org/web/20221126231944/https://mvnrepository.com/artifact/ring/ring-core)
*   `[ring/ring-jetty-adapter](https://web.archive.org/web/20221126231944/https://mvnrepository.com/artifact/ring/ring-jetty-adapter)`

我们可以将这些添加到我们的 Leiningen 项目中:

```
 :dependencies [[org.clojure/clojure "1.10.0"]
                 [ring/ring-core "1.7.1"]
                 [ring/ring-jetty-adapter "1.7.1"]]
```

然后，我们可以将它添加到一个最小的项目中:

```
(ns ring.core
  (:use ring.adapter.jetty))

(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "Hello World"})

(defn -main
  [& args]
  (run-jetty handler {:port 3000}))
```

在这里，我们定义了一个处理函数——我们很快会讲到——它总是返回字符串“Hello World”。此外，我们添加了我们的主函数来使用这个处理程序——它将侦听端口 3000 上的请求。

## 3。核心概念

Leiningen 有几个核心概念，所有东西都是围绕这些概念构建的:请求、响应、处理程序和中间件。

### 3.1。请求数

请求是传入 HTTP 请求的表示。Ring 将一个请求表示为一个映射，允许我们的 Clojure 应用程序轻松地与各个字段进行交互。该地图中有一组标准的键，包括但不限于:

*   完整的 URI 之路。
*   `:query-string `–完整的查询字符串。
*   `:request-method`–请求方法，`:get, :head, :post, :put, :delete`或`:options.`之一
*   `:headers`–提供给请求的所有 HTTP 头的映射。
*   `:body`–代表请求主体的`InputStream`，如果存在。

**中间件可以根据需要向该映射添加更多的键**。

### 3.2。回应

类似地，响应是传出 HTTP 响应的表示。 **Ring 也用三个标准键**将它们表示为地图:

*   `:status`–发送回的状态码
*   :`headers`–要发送回的所有 HTTP 头的映射
*   :`body`–要送回的可选主体

和以前一样，**中间件可能会在我们的处理程序产生它和最终结果被发送到客户端**之间改变这一点。

**Ring 还提供了一些助手，使构建响应变得更加容易**。

其中最基本的是`ring.util.response/response`函数，它创建一个简单的响应，状态代码为`200 OK`:

```
ring.core=> (ring.util.response/response "Hello")
{:status 200, :headers {}, :body "Hello"}
```

**对于常见的状态码**，还有其他一些方法，例如`bad-request`、`not-found`和`redirect`:

```
ring.core=> (ring.util.response/bad-request "Hello")
{:status 400, :headers {}, :body "Hello"}
ring.core=> (ring.util.response/created "/post/123")
{:status 201, :headers {"Location" "/post/123"}, :body nil}
ring.core=> (ring.util.response/redirect "https://ring-clojure.github.io/ring/")
{:status 302, :headers {"Location" "https://ring-clojure.github.io/ring/"}, :body ""}
```

我们还有`status `方法，它将把现有的响应转换成任意的状态代码:

```
ring.core=> (ring.util.response/status (ring.util.response/response "Hello") 409)
{:status 409, :headers {}, :body "Hello"}
```

**然后我们有一些方法来调整类似**响应的其他特征——例如，`content-type, header `或`set-cookie`:

```
ring.core=> (ring.util.response/content-type (ring.util.response/response "Hello") "text/plain")
{:status 200, :headers {"Content-Type" "text/plain"}, :body "Hello"}
ring.core=> (ring.util.response/header (ring.util.response/response "Hello") "X-Tutorial-For" "Baeldung")
{:status 200, :headers {"X-Tutorial-For" "Baeldung"}, :body "Hello"}
ring.core=> (ring.util.response/set-cookie (ring.util.response/response "Hello") "User" "123")
{:status 200, :headers {}, :body "Hello", :cookies {"User" {:value "123"}}}
```

注意，**`set-cookie`方法向响应映射**添加了一个全新的条目。**这需要`wrap-cookies`中间件**正确处理它才能工作。

### 3.3。处理者

现在我们已经理解了请求和响应，我们可以开始编写处理函数来将它们联系在一起。

**处理程序是一个简单的函数，它将传入的请求作为参数，并返回传出的响应**。我们在这个函数中做什么完全取决于我们的应用程序，只要它符合这个契约。

最简单的是，我们可以编写一个总是返回相同响应的函数:

```
(defn handler [request] (ring.util.response/response "Hello"))
```

我们也可以根据需要与请求进行交互。

例如，我们可以编写一个处理程序来返回传入的 IP 地址:

```
(defn check-ip-handler [request]
    (ring.util.response/content-type
        (ring.util.response/response (:remote-addr request))
        "text/plain"))
```

### 3.4。中间件

中间件这个名字在一些语言中很常见，但在 Java 世界中却不太常见。概念上，它们类似于 Servlet 过滤器和 Spring 拦截器。

在 Ring 中，中间件指的是包装主处理程序并以某种方式调整它的某些方面的简单功能。这可能意味着在处理传入请求之前对其进行变异，在生成传出响应之后对其进行变异，或者除了记录处理时间之外可能什么也不做。

一般来说，**中间件函数接受处理程序的第一个参数进行包装，并返回具有新功能的新处理程序函数**。

**中间件可以根据需要使用许多其他参数**。例如，我们可以使用下面的代码为来自包装处理程序的每个响应设置`Content-Type`头:

```
(defn wrap-content-type [handler content-type]
  (fn [request]
    (let [response (handler request)]
      (assoc-in response [:headers "Content-Type"] content-type))))
```

通读它，我们可以看到我们返回了一个接受请求的函数——这是新的处理程序。这将调用提供的处理程序，然后返回响应的变异版本。

我们可以用它来产生一个新的处理程序，只需将它们链接在一起:

```
(def app-handler (wrap-content-type handler "text/html"))
```

Clojure 还提供了一种以更自然的方式将许多链接在一起的方法——通过使用线程宏[。这是一种提供要调用的函数列表的方法，每个函数都有前一个函数的输出。](https://web.archive.org/web/20221126231944/https://clojure.org/guides/threading_macros)

**具体来说，我们需要线程优先宏，** `**->**.` 这将允许我们用提供的值作为第一个参数来调用每个中间件:

```
(def app-handler
  (-> handler
      (wrap-content-type "text/html")
      wrap-keyword-params
      wrap-params))
```

这就产生了一个处理程序，它是包装在三个不同中间件功能中的原始处理程序。

## 4。编写处理程序

现在我们已经了解了组成 Ring 应用程序的组件，我们需要知道我们可以用实际的处理程序做什么。这些是整个应用程序的核心，也是大部分业务逻辑的归宿。

我们可以将任何我们想要的代码放入这些处理程序中，包括数据库访问或调用其他服务。Ring 为我们提供了一些直接处理传入请求或传出响应的额外能力，这也非常有用。

### 4.1。服务静态资源

任何 web 应用程序可以执行的最简单的功能之一就是提供静态资源。 **Ring 提供了两个中间件函数来简化这一过程——`wrap-file`和`wrap-resource`T3。**

**`wrap-file`中间件在文件系统**上获取一个目录。如果传入的请求与此目录中的文件匹配，则返回该文件，而不是调用处理函数:

```
(use 'ring.middleware.file) 
```

```
(def app-handler (wrap-file your-handler "/var/www/public"))
```

以非常相似的方式，**`wrap-resource`中间件使用一个类路径前缀，它在这个前缀中查找文件**:

```
(use 'ring.middleware.resource) 
```

```
(def app-handler (wrap-resource your-handler "public"))
```

在这两种情况下，**包装的处理函数只有在没有找到返回给客户端**的文件时才会被调用。

Ring 还提供了额外的中间件来使这些更干净地在 HTTP API 上使用:

```
(use 'ring.middleware.resource
     'ring.middleware.content-type
     'ring.middleware.not-modified)

(def app-handler
  (-> your-handler
      (wrap-resource "public")
      wrap-content-type
      wrap-not-modified)
```

`wrap-content-type`中间件将根据请求的文件扩展名自动确定要设置的`Content-Type`头。`wrap-not-modified`中间件将`If-Not-Modified`头与`Last-Modified`值进行比较，以支持 HTTP 缓存，仅在需要时返回文件。

### 4.2。访问请求参数

在处理请求时，客户端可以通过一些重要的方式向服务器提供信息。这些包括查询字符串参数——包含在 URL 和表单参数中——作为 POST 和 PUT 请求的请求有效负载提交。

**在我们可以使用参数之前，我们必须使用`wrap-params`中间件来包装处理程序**。这将正确解析参数，支持 URL 编码，并使它们可用于请求。这可以选择指定要使用的字符编码，如果没有指定，默认为 UTF-8:

```
(def app-handler
  (-> your-handler
      (wrap-params {:encoding "UTF-8"})
  ))
```

一旦完成，**请求将被更新以使参数可用**。这些进入传入请求中的适当键:

*   `:query-params`–从查询字符串中解析出的参数
*   `:form-params`–从表体解析出的参数
*   `:params`—`:query-params`和`:form-params`的组合

我们可以完全按照预期在请求处理程序中利用这一点。

```
(defn echo-handler [{params :params}]
    (ring.util.response/content-type
        (ring.util.response/response (get params "input"))
        "text/plain"))
```

这个处理程序将返回一个包含参数`input`值的响应。

**如果只有一个值，参数映射到一个字符串；如果有多个值，参数映射到一个列表**。

例如，我们得到以下参数映射:

```
// /echo?input=hello
{"input "hello"}

// /echo?input=hello&name;=Fred
{"input "hello" "name" "Fred"}

// /echo?input=hello&input;=world
{"input ["hello" "world"]}
```

### 4.3。接收文件上传

通常，我们希望能够编写用户可以上传文件的 web 应用程序。在 HTTP 协议中，这通常使用多部分请求来处理。这些允许单个请求包含表单参数和一组文件。

**Ring 附带了一个名为`wrap-multipart-params`的中间件来处理这种请求。**这类似于`wrap-params`解析简单请求的方式。

`wrap-multipart-params` 自动解码并存储任何上传到文件系统的文件，并告诉处理程序它们的位置，以便处理它们:

```
(def app-handler
  (-> your-handler
      wrap-params
      wrap-multipart-params
  ))
```

默认情况下，**上传的文件存储在临时系统目录中，并在一小时后自动删除**。请注意，这确实要求 JVM 在接下来的一个小时内仍在运行，以执行清理。

如果愿意的话，**还有一个内存存储**，尽管很明显，如果上传了大文件，这有耗尽内存的风险。

如果需要，我们也可以编写我们的存储引擎，只要它满足 API 要求。

```
(def app-handler
  (-> your-handler
      wrap-params
      (wrap-multipart-params {:store ring.middleware.multipart-params.byte-array/byte-array-store})
  ))
```

一旦这个中间件建立起来，**上传的文件在`params`键**下的传入请求对象上是可用的。这与使用`wrap-params `中间件是一样的。该条目是一个映射，包含处理文件所需的详细信息，具体取决于所使用的存储。

例如，默认的临时文件存储返回值:

```
 {"file" {:filename     "words.txt"
           :content-type "text/plain"
           :tempfile     #object[java.io.File ...]
           :size         51}}
```

其中`:tempfile `条目是直接表示文件系统上的文件的`java.io.File`对象。

### 4.4。使用 cookie

Cookies 是一种机制，服务器可以提供少量数据，客户端将在后续请求中继续发回这些数据。这通常用于会话 id、访问令牌或持久用户数据，如配置的本地化设置。

Ring 有中间件，可以让我们轻松地使用 cookies】。 这将自动解析传入请求上的 cookie，并且还将允许我们在传出响应 上创建新的 cookie。

配置这个中间件遵循与以前相同的模式:

```
(def app-handler
  (-> your-handler
      wrap-cookies
  ))
```

此时，**所有传入的请求都将解析它们的 cookies，并放入请求**的`:cookies`键中。这将包含 cookie 名称和值的映射:

```
{"session_id" {:value "session-id-hash"}}
```

**然后，我们可以通过将`:cookies`键添加到输出响应**来将 cookies 添加到输出响应中。我们可以通过直接创建响应来做到这一点:

```
{:status 200
 :headers {}
 :cookies {"session_id" {:value "session-id-hash"}}
 :body "Setting a cookie."}
```

**还有一个助手函数，我们可以用它来给响应**添加 cookies，类似于我们之前设置状态代码或标题的方式:

```
(ring.util.response/set-cookie 
    (ring.util.response/response "Setting a cookie.") 
    "session_id" 
    "session-id-hash")
```

根据 HTTP 规范的需要，Cookies 上还可以设置额外的选项。如果我们使用`set-cookie`，那么我们在键和值之后提供这些作为 map 参数。此地图的关键字是:

*   `:domain`–将 cookie 限制到的域
*   `:path`–限制 cookie 的路径
*   `:secure`–`true`仅在 HTTPS 连接上发送 cookie
*   `:http-only`–`true`让 JavaScript 无法访问 cookie
*   `:max-age`–浏览器删除 cookie 的秒数
*   `:expires`–浏览器删除 cookie 之前的特定时间戳
*   `:same-site`–如果设置为`:strict`，则浏览器不会将此 cookie 与跨站点请求一起发回。

```
(ring.util.response/set-cookie
    (ring.util.response/response "Setting a cookie.")
    "session_id"
    "session-id-hash"
    {:secure true :http-only true :max-age 3600})
```

### 4.5。会话

Cookies 让我们能够存储客户在每次请求时发送回服务器的信息。实现这一点的更有效的方法是使用会话。这些全部存储在服务器上，但是客户端维护决定使用哪个会话的标识符。

和这里的其他东西一样，**会话是使用中间件函数**实现的:

```
(def app-handler
  (-> your-handler
      wrap-session
  ))
```

默认情况下，**将会话数据存储在内存**中。如果需要，我们可以改变这一点，Ring 提供了一个替代存储，使用 cookies 来存储所有的会话数据。

与上传文件一样，**如果需要的话，我们可以提供存储功能**。

```
(def app-handler
  (-> your-handler
      wrap-cookies
      (wrap-session {:store (cookie-store {:key "a 16-byte secret"})})
  ))
```

**我们还可以调整用于存储会话密钥**的 cookie 的细节。

例如，要使会话 cookie 持续一小时，我们可以这样做:

```
(def app-handler
  (-> your-handler
      wrap-cookies
      (wrap-session {:cookie-attrs {:max-age 3600}})
  ))
```

**这里的 cookie 属性与`wrap-cookies`中间件支持的属性相同。**

会话通常可以作为数据存储来使用。这在函数式编程模型中并不总是有效，所以 Ring 对它们的实现略有不同。

相反，**我们从请求中访问会话数据，并返回一个数据映射，作为响应的一部分存储在其中**。这是要存储的整个会话状态，而不仅仅是更改后的值。

例如，下面记录了处理程序被请求的次数:

```
(defn handler [{session :session}]
  (let [count   (:count session 0)
        session (assoc session :count (inc count))]
    (-> (response (str "You accessed this page " count " times."))
        (assoc :session session))))
```

通过这种方式，我们可以简单地通过不包含键来从会话中删除数据。我们还可以通过返回新地图的`nil`来删除整个会话。

```
(defn handler [request]
  (-> (response "Session deleted.")
      (assoc :session nil)))
```

## 5\. Leiningen Plugin

Ring 为 [Leiningen 构建工具](/web/20221126231944/https://www.baeldung.com/leiningen-clojure)提供了一个插件来帮助开发和生产。

我们通过向`project.clj`文件添加正确的插件细节来设置插件:

```
 :plugins [[lein-ring "0.12.5"]]
  :ring {:handler ring.core/handler}
```

**重要的是 [`lein-ring`](https://web.archive.org/web/20221126231944/https://mvnrepository.com/artifact/lein-ring/lein-ring) 的版本对于戒指**的版本是正确的。这里我们一直用的是 Ring 1.7.1，也就是说我们需要`lein-ring ` 0.12.5。一般来说，最安全的方法是使用两者的最新版本，就像在 Maven central 上看到的那样，或者使用`lein search`命令:

```
$ lein search ring-core
Searching clojars ...
[ring/ring-core "1.7.1"]
  Ring core libraries.

$ lein search lein-ring
Searching clojars ...
[lein-ring "0.12.5"]
  Leiningen Ring plugin
```

`:ring`调用的`:handler`参数是我们想要使用的处理程序的全限定名称。这可以包括我们定义的任何中间件。

使用这个插件意味着我们不再需要一个主函数。我们可以使用 Leiningen 在开发模式下运行，或者我们可以构建一个用于部署的产品。**我们的代码现在完全符合我们的逻辑，仅此而已**。

### 5.1。打造生产神器

设置好之后，**我们现在可以构建一个 WAR 文件，我们可以将它部署到任何标准的 servlet 容器**:

```
$ lein ring uberwar
2019-04-12 07:10:08.033:INFO::main: Logging initialized @1054ms to org.eclipse.jetty.util.log.StdErrLog
Created ./clojure/ring/target/uberjar/ring-0.1.0-SNAPSHOT-standalone.war
```

**我们还可以构建一个独立的 JAR 文件，完全按照预期运行我们的处理程序**:

```
$ lein ring uberjar
Compiling ring.core
2019-04-12 07:11:27.669:INFO::main: Logging initialized @3016ms to org.eclipse.jetty.util.log.StdErrLog
Created ./clojure/ring/target/uberjar/ring-0.1.0-SNAPSHOT.jar
Created ./clojure/ring/target/uberjar/ring-0.1.0-SNAPSHOT-standalone.jar
```

**这个 JAR 文件将包含一个主类，它将启动我们包含的**的嵌入式容器中的处理程序。这也将支持一个环境变量`PORT`,允许我们在生产环境中轻松运行它:

```
PORT=2000 java -jar ./clojure/ring/target/uberjar/ring-0.1.0-SNAPSHOT-standalone.jar
2019-04-12 07:14:08.954:INFO::main: Logging initialized @1009ms to org.eclipse.jetty.util.log.StdErrLog
WARNING: seqable? already refers to: #'clojure.core/seqable? in namespace: clojure.core.incubator, being replaced by: #'clojure.core.incubator/seqable?
2019-04-12 07:14:10.795:INFO:oejs.Server:main: jetty-9.4.z-SNAPSHOT; built: 2018-08-30T13:59:14.071Z; git: 27208684755d94a92186989f695db2d7b21ebc51; jvm 1.8.0_77-b03
2019-04-12 07:14:10.863:INFO:oejs.AbstractConnector:main: Started [[email protected]](/web/20221126231944/https://www.baeldung.com/cdn-cgi/l/email-protection){HTTP/1.1,[http/1.1]}{0.0.0.0:2000}
2019-04-12 07:14:10.863:INFO:oejs.Server:main: Started @2918ms
Started server on port 2000
```

### 5.2。在开发模式下运行

出于开发目的，**我们可以直接从 Leiningen 运行处理程序，而不需要手动构建和运行它**。这使得在真实的浏览器中测试我们的应用程序变得更加容易:

```
$ lein ring server
2019-04-12 07:16:28.908:INFO::main: Logging initialized @1403ms to org.eclipse.jetty.util.log.StdErrLog
2019-04-12 07:16:29.026:INFO:oejs.Server:main: jetty-9.4.12.v20180830; built: 2018-08-30T13:59:14.071Z; git: 27208684755d94a92186989f695db2d7b21ebc51; jvm 1.8.0_77-b03
2019-04-12 07:16:29.092:INFO:oejs.AbstractConnector:main: Started [[email protected]](/web/20221126231944/https://www.baeldung.com/cdn-cgi/l/email-protection){HTTP/1.1,[http/1.1]}{0.0.0.0:3000}
2019-04-12 07:16:29.092:INFO:oejs.Server:main: Started @1587ms
```

如果我们已经设置了环境变量,`PORT`的话，这也是值得的。

另外，**我们可以将一个环开发库添加到我们的项目**中。如果这是可用的，那么**开发服务器将尝试自动重新加载任何检测到的源代码变更**。这可以给我们一个高效的工作流程来改变代码，并在我们的浏览器中看到它。这需要添加`ring-devel`依赖项:

```
[ring/ring-devel "1.7.1"]
```

## 6。结论

在本文中，我们简要介绍了作为在 Clojure 中编写 web 应用程序的手段的 Ring 库。为什么不在下一个项目中尝试一下呢？

我们在这里讨论的一些概念的例子可以在 [GitHub](https://web.archive.org/web/20221126231944/https://github.com/eugenp/tutorials/tree/master/clojure/ring) 中看到。