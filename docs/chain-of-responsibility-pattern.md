# Java 中的责任链设计模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/chain-of-responsibility-pattern>

## 1。简介

在这篇文章中，我们将看看一个广泛使用的**行为设计模式** : **责任链**。

我们可以在我们的[上一篇文章](/web/20221208143832/https://www.baeldung.com/creational-design-patterns)中找到更多的设计模式。

## 2。责任链

[维基百科](https://web.archive.org/web/20221208143832/https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)将责任链定义为由“一个命令对象源和一系列处理对象”组成的设计模式。

链中的每个处理对象负责特定类型的命令，处理完成后，它将命令转发给链中的下一个处理器。

责任链模式适用于:

*   **解耦命令的发送者和接收者**
*   **在加工时选择加工策略**

因此，让我们来看一个简单的模式示例。

## 3。示例

我们将使用责任链来创建处理身份验证请求的链。

因此，输入认证提供者将是`command`，每个认证处理器将是一个单独的`processor`对象。

让我们首先为我们的处理器创建一个抽象基类:

```
public abstract class AuthenticationProcessor {

    public AuthenticationProcessor nextProcessor;

    // standard constructors

    public abstract boolean isAuthorized(AuthenticationProvider authProvider);
}
```

接下来，让我们创建扩展`AuthenticationProcessor`的具体处理器:

```
public class OAuthProcessor extends AuthenticationProcessor {

    public OAuthProcessor(AuthenticationProcessor nextProcessor) {
        super(nextProcessor);
    }

    @Override
    public boolean isAuthorized(AuthenticationProvider authProvider) {
        if (authProvider instanceof OAuthTokenProvider) {
            return true;
        } else if (nextProcessor != null) {
            return nextProcessor.isAuthorized(authProvider);
        }

        return false;
    }
}
```

```
public class UsernamePasswordProcessor extends AuthenticationProcessor {

    public UsernamePasswordProcessor(AuthenticationProcessor nextProcessor) {
        super(nextProcessor);
    }

    @Override
    public boolean isAuthorized(AuthenticationProvider authProvider) {
        if (authProvider instanceof UsernamePasswordProvider) {
            return true;
        } else if (nextProcessor != null) {
            return nextProcessor.isAuthorized(authProvider);
        }
    return false;
    }
}
```

这里，我们为传入的授权请求创建了两个具体的处理器:`UsernamePasswordProcessor`和`OAuthProcessor` `.`

对于每一个，我们重写了`isAuthorized`方法。

现在让我们创建几个测试:

```
public class ChainOfResponsibilityTest {

    private static AuthenticationProcessor getChainOfAuthProcessor() {
        AuthenticationProcessor oAuthProcessor = new OAuthProcessor(null);
        return new UsernamePasswordProcessor(oAuthProcessor);
    }

    @Test
    public void givenOAuthProvider_whenCheckingAuthorized_thenSuccess() {
        AuthenticationProcessor authProcessorChain = getChainOfAuthProcessor();
        assertTrue(authProcessorChain.isAuthorized(new OAuthTokenProvider()));
    }

    @Test
    public void givenSamlProvider_whenCheckingAuthorized_thenSuccess() {
        AuthenticationProcessor authProcessorChain = getChainOfAuthProcessor();

        assertFalse(authProcessorChain.isAuthorized(new SamlTokenProvider()));
    }
}
```

上面的例子创建了一个认证处理器链:`UsernamePasswordProcessor -> OAuthProcessor`。在第一个测试中，授权成功，而在另一个测试中，授权失败。

首先，`UsernamePasswordProcessor`检查验证提供者是否是`UsernamePasswordProvider`的实例。

不是预期的输入，`UsernamePasswordProcessor`委托给`OAuthProcessor`。

最后，`OAuthProcessor`处理命令。在第一个测试中，有一个匹配，测试通过。在第二种情况下，链中不再有处理器，因此测试失败。

## 4。实施原则

在实施责任链时，我们需要牢记几个重要原则:

*   **链中的每个处理器都有处理命令的实现**
    *   在上面的例子中，所有处理器都有自己的`isAuthorized`实现
*   **链中的每个处理器都应该引用下一个处理器**
    *   上图，`UsernamePasswordProcessor`代表到`OAuthProcessor`
*   **每个处理器负责委托给下一个处理器，因此要小心丢失的命令**
    *   同样在我们的例子中，如果命令是`SamlProvider`的一个实例，那么请求可能不会被处理，并且将是未授权的
*   **处理器不应形成递归循环**
    *   在我们的例子中，我们的链中没有一个循环:`UsernamePasswordProcessor -> OAuthProcessor` `.`，但是，如果我们显式地将`UsernamePasswordProcessor` 设置为 `OAuthProcessor,` **的下一个处理器，那么我们的链中就会有一个循环** : `UsernamePasswordProcessor -> OAuthProcessor -> UsernamePasswordProcessor.` 在构造函数中获取下一个处理器可以有助于此
*   链中只有一个处理器处理给定的命令
    *   在我们的例子中，如果一个传入的命令包含一个`OAuthTokenProvider`的实例，那么只有`OAuthProcessor`将处理这个命令

## 5。真实世界中的用法

在 Java 世界中，我们每天都受益于责任链。一个经典的例子是 **Java** 中的 Servlet 过滤器，它允许多个过滤器处理一个 HTTP 请求。虽然在这种情况下，**每个过滤器调用链，而不是下一个过滤器。**

让我们看看下面的代码片段，以便更好地理解 Servlet 过滤器 **:** 中的这种模式

```
public class CustomFilter implements Filter {

    public void doFilter(
      ServletRequest request,
      ServletResponse response,
      FilterChain chain)
      throws IOException, ServletException {

        // process the request

        // pass the request (i.e. the command) along the filter chain
        chain.doFilter(request, response);
    }
}
```

如上面的代码片段所示，我们需要调用`FilterChain`的 `doFilter`方法，以便将请求传递给链中的下一个处理器。

## 6。缺点

既然我们已经看到了责任链是多么有趣，让我们记住一些缺点:

*   大多数情况下，它很容易坏掉:
    *   如果一个处理器未能调用下一个处理器，命令将被丢弃
    *   如果一个处理器调用了错误的处理器，就会导致一个循环
*   它可能会创建很深的堆栈跟踪，从而影响性能
*   这会导致处理器间的重复代码，增加维护量

## 7 .**。结论**

在本文中，我们讨论了责任链以及它在授权传入身份验证请求链的帮助下的优缺点。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-behavioral)