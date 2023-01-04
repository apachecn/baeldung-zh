# Spring MVC 流和 SSE 请求处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-sse-streams>

## 1。简介

这个简单的教程演示了 Spring MVC 5.x.x 中几个异步和流对象的使用。

具体来说，我们将回顾三个关键类:

*   `ResponseBodyEmitter`
*   `SseEmitter`
*   `StreamingResponseBody`

此外，我们将讨论如何使用 JavaScript 客户端与他们进行交互。

## 2.`ResponseBodyEmitter`

**`ResponseBodyEmitter`处理异步响应。**

此外，它还代表了许多子类的父类——下面我们将仔细研究其中的一个子类。

### 2.1.服务器端

最好使用一个`ResponseBodyEmitter` 连同它自己专用的异步线程，并用一个`ResponseEntity`包装(我们可以直接将`emitter`注入其中):

```
@Controller
public class ResponseBodyEmitterController {

    private ExecutorService executor 
      = Executors.newCachedThreadPool();

    @GetMapping("/rbe")
    public ResponseEntity<ResponseBodyEmitter> handleRbe() {
        ResponseBodyEmitter emitter = new ResponseBodyEmitter();
        executor.execute(() -> {
            try {
                emitter.send(
                  "/rbe" + " @ " + new Date(), MediaType.TEXT_PLAIN);
                emitter.complete();
            } catch (Exception ex) {
                emitter.completeWithError(ex);
            }
        });
        return new ResponseEntity(emitter, HttpStatus.OK);
    }
}
```

所以，在上面的例子中，**我们可以避开需要使用`CompleteableFutures`** ，更复杂的异步承诺，或者使用`@Async` 注释。

相反，我们简单地声明我们的异步实体，并将其包装在由`ExecutorService.`提供的新的`Thread`中

### 2.2.客户端

对于客户端使用，我们可以使用一个简单的 XHR 方法并调用我们的 API 端点，就像在普通的 AJAX 操作中一样:

```
var xhr = function(url) {
    return new Promise(function(resolve, reject) {
        var xmhr = new XMLHttpRequest();
        //...
        xmhr.open("GET", url, true);
        xmhr.send();
       //...
    });
};

xhr('http://localhost:8080/javamvcasync/rbe')
  .then(function(success){ //... }); 
```

## 3.`SseEmitter`

**`SseEmitter`实际上是`ResponseBodyEmitter`的子类，提供额外的`Server-Sent Event` (SSE)** 开箱即用支持。

### 3.1.服务器端

因此，让我们快速看一下利用这一强大实体的示例控制器:

```
@Controller
public class SseEmitterController {
    private ExecutorService nonBlockingService = Executors
      .newCachedThreadPool();

    @GetMapping("/sse")
    public SseEmitter handleSse() {
         SseEmitter emitter = new SseEmitter();
         nonBlockingService.execute(() -> {
             try {
                 emitter.send("/sse" + " @ " + new Date());
                 // we could send more events
                 emitter.complete();
             } catch (Exception ex) {
                 emitter.completeWithError(ex);
             }
         });
         return emitter;
    }   
} 
```

相当标准的票价，但我们会注意到这与我们通常的休息控制器之间的一些差异:

*   首先，我们返回一个`SseEmitter`
*   此外，我们将核心响应信息包装在它自己的`Thread`中
*   最后，我们使用` emitter.send()`发送响应信息

### 3.2.客户端

我们的客户端这次的工作方式略有不同，因为我们可以利用持续连接的 `Server-Sent Event`库:

```
var sse = new EventSource('http://localhost:8080/javamvcasync/sse');
sse.onmessage = function (evt) {
    var el = document.getElementById('sse');
    el.appendChild(document.createTextNode(evt.data));
    el.appendChild(document.createElement('br'));
};
```

## 4.`StreamingResponseBody`

最后，**我们可以使用`StreamingResponseBody `直接写入一个`OutputStream`，然后使用一个`ResponseEntity.`** 将写入的信息返回给客户端

### 4.1.服务器端

```
@Controller
public class StreamingResponseBodyController {

    @GetMapping("/srb")
    public ResponseEntity<StreamingResponseBody> handleRbe() {
        StreamingResponseBody stream = out -> {
            String msg = "/srb" + " @ " + new Date();
            out.write(msg.getBytes());
        };
        return new ResponseEntity(stream, HttpStatus.OK);
    }
} 
```

### 4.2.客户端

就像之前一样，我们将使用常规的 XHR 方法来访问上面的控制器:

```
var xhr = function(url) {
    return new Promise(function(resolve, reject) {
        var xmhr = new XMLHttpRequest();
        //...
        xmhr.open("GET", url, true);
        xmhr.send();
        //...
    });
};

xhr('http://localhost:8080/javamvcasync/srb')
  .then(function(success){ //... }); 
```

接下来，我们来看看这些例子的一些成功运用。

## 5。将所有这些整合在一起

在我们成功编译我们的服务器并运行我们的客户端(访问提供的`index.jsp`)之后，我们应该在浏览器中看到以下内容:

![SpringMVCAsyncResponseReques.v3](img/1ee82cfd0e4cecb5768fcfea80967763.png)
及以下出现在我们的终端:

![Terminal](img/50d1e45cadade16b56ad0f138836f2ff.png)

我们还可以直接调用端点，并在浏览器中看到它们的流响应。

## 6。结论

虽然`Future`和`CompleteableFuture`已经被证明是 Java 和 Spring 的强大补充，但我们现在有一些资源可以更充分地处理高并发 web 应用程序的异步和流数据。

最后，在 GitHub 上查看完整的代码示例[。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-5-mvc)