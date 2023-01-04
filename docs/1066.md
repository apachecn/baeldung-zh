# 用@Async 进行 Spring 安全上下文传播

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-async-principal-propagation>

## 1。简介

在本教程中，我们将重点介绍**传播的春天安全校长与`@Async`** `.`

默认情况下，Spring 安全认证被绑定到一个`ThreadLocal`——因此，当执行流在一个新线程中用@Async 运行时，这不会是一个认证的上下文。

那不理想，让我们修理它。

## 2。Maven 依赖关系

为了在 Spring Security 中使用异步集成，我们需要在我们的`pom.xml`的`dependencies`中包含以下部分:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.6.0</version>
</dependency> 
```

Spring 安全依赖的最新版本可以在[这里](https://web.archive.org/web/20220812060426/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.security%22)找到。

## 3。`@Async`春安传播

让我们先写一个简单的例子:

```
@RequestMapping(method = RequestMethod.GET, value = "/async")
@ResponseBody
public Object standardProcessing() throws Exception {
    log.info("Outside the @Async logic - before the async call: "
      + SecurityContextHolder.getContext().getAuthentication().getPrincipal());

    asyncService.asyncCall();

    log.info("Inside the @Async logic - after the async call: "
      + SecurityContextHolder.getContext().getAuthentication().getPrincipal());

    return SecurityContextHolder.getContext().getAuthentication().getPrincipal();
}
```

**我们要检查弹簧`SecurityContext`是否传播到新线程。**首先，我们在异步调用之前记录上下文，接下来我们运行异步方法，最后我们再次记录上下文。`asyncCall()`方法有以下实现:

```
@Async
@Override
public void asyncCall() {
    log.info("Inside the @Async logic: "
      + SecurityContextHolder.getContext().getAuthentication().getPrincipal());
}
```

正如我们所看到的，只有一行代码将输出异步方法的新线程内部的上下文。

## 4。默认配置

**默认情况下， `@Async`方法中的安全上下文会有一个`null`值。**

特别是，如果我们运行异步逻辑，我们将能够在主程序中记录`Authentication`对象，但是当我们在`@Async`中记录它时，它将是`null`。这是一个日志输出示例:

```
web - 2016-12-30 22:41:58,916 [http-nio-8081-exec-3] INFO
  o.baeldung.web.service.AsyncService -
  Outside the @Async logic - before the async call:
  [[email protected]](/web/20220812060426/https://www.baeldung.com/cdn-cgi/l/email-protection):
  Username: temporary; ...

web - 2016-12-30 22:41:58,921 [http-nio-8081-exec-3] INFO
  o.baeldung.web.service.AsyncService -
  Inside the @Async logic - after the async call:
  [[email protected]](/web/20220812060426/https://www.baeldung.com/cdn-cgi/l/email-protection):
  Username: temporary; ...

  web - 2016-12-30 22:41:58,926 [SimpleAsyncTaskExecutor-1] ERROR
  o.s.a.i.SimpleAsyncUncaughtExceptionHandler -
  Unexpected error occurred invoking async method
  'public void com.baeldung.web.service.AsyncServiceImpl.asyncCall()'.
  java.lang.NullPointerException: null
```

因此，如您所见，在 executor 线程中，我们的调用因 NPE 而失败，这是意料之中的——因为主体不在那里。

## 5。异步安全上下文配置

如果我们想要访问异步线程内部的主体，就像我们在外部访问它一样，我们需要创建`DelegatingSecurityContextAsyncTaskExecutor` bean:

```
@Bean 
public DelegatingSecurityContextAsyncTaskExecutor taskExecutor(ThreadPoolTaskExecutor delegate) { 
    return new DelegatingSecurityContextAsyncTaskExecutor(delegate); 
}
```

通过这样做，Spring 将在每个`@Async`调用中使用当前的`SecurityContext`。

现在，让我们再次运行该应用程序，并查看日志记录信息以确保情况确实如此:

```
web - 2016-12-30 22:45:18,013 [http-nio-8081-exec-3] INFO
  o.baeldung.web.service.AsyncService -
  Outside the @Async logic - before the async call:
  [[email protected]](/web/20220812060426/https://www.baeldung.com/cdn-cgi/l/email-protection):
  Username: temporary; ...

web - 2016-12-30 22:45:18,018 [http-nio-8081-exec-3] INFO
  o.baeldung.web.service.AsyncService -
  Inside the @Async logic - after the async call:
  [[email protected]](/web/20220812060426/https://www.baeldung.com/cdn-cgi/l/email-protection):
  Username: temporary; ...

web - 2016-12-30 22:45:18,019 [SimpleAsyncTaskExecutor-1] INFO
  o.baeldung.web.service.AsyncService -
  Inside the @Async logic:
  [[email protected]](/web/20220812060426/https://www.baeldung.com/cdn-cgi/l/email-protection):
  Username: temporary; ...
```

现在，正如我们所料，我们在异步执行器线程中看到了相同的主体。 ****

## 6。用例

有几个有趣的用例，我们可能希望确保`SecurityContext` 像这样传播:

*   我们希望发出多个外部请求，这些请求可以并行运行，并且可能需要很长时间来执行
*   我们在本地有一些重要的处理要做，我们的外部请求可以并行执行
*   其他的代表“一发了之”的场景，比如发送电子邮件

## 7。结论

在这个快速教程中，我们展示了 Spring 对发送带有传播`SecurityContext.` 的异步请求的支持。从编程模型的角度来看，这些新功能看起来似乎很简单。

请注意，如果多个方法调用以前以同步方式链接在一起，转换到异步方式可能需要同步结果。

这个例子也可以在 Github 上作为 Maven 项目[获得。](https://web.archive.org/web/20220812060426/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-rest)