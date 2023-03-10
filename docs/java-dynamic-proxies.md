# Java 中的动态代理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-dynamic-proxies>

## 1。简介

这篇文章是关于 [Java 的动态代理](https://web.archive.org/web/20221026145528/https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/proxy.html)——这是该语言中我们可用的主要代理机制之一。

简而言之，代理是通过自己的工具传递函数调用(通常传递到真实的方法上)的前端或包装器，可能会增加一些功能。

动态代理允许一个单独的类和一个单独的方法服务于对任意类和任意数量的方法的多个方法调用。动态代理可以被认为是一种`Facade`，但是它可以伪装成任何接口的实现。在幕后，**将所有方法调用路由到一个单独的处理程序**—`invoke()`方法。

虽然它不是一个用于日常编程任务的工具，但是动态代理对于框架编写者来说非常有用。它也可以用于具体的类实现直到运行时才知道的情况。

此功能内置于标准 JDK 中，因此不需要额外的依赖项。

## 2。调用处理程序

让我们构建一个简单的代理，它除了打印请求调用的方法并返回一个硬编码的数字之外，实际上不做任何事情。

首先，我们需要创建一个`java.lang.reflect.InvocationHandler`的子类型:

```java
public class DynamicInvocationHandler implements InvocationHandler {

    private static Logger LOGGER = LoggerFactory.getLogger(
      DynamicInvocationHandler.class);

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) 
      throws Throwable {
        LOGGER.info("Invoked method: {}", method.getName());

        return 42;
    }
}
```

这里我们定义了一个简单的代理，它记录调用了哪个方法并返回 42。

## 3。创建代理实例

由我们刚刚定义的调用处理程序服务的代理实例是通过对`java.lang.reflect.Proxy`类的工厂方法调用创建的:

```java
Map proxyInstance = (Map) Proxy.newProxyInstance(
  DynamicProxyTest.class.getClassLoader(), 
  new Class[] { Map.class }, 
  new DynamicInvocationHandler());
```

一旦我们有了代理实例，我们就可以像平常一样调用它的接口方法:

```java
proxyInstance.put("hello", "world");
```

正如所料，在日志文件中打印出一条关于调用`put()`方法的消息。

## 4。通过 Lambda 表达式调用处理器

因为`InvocationHandler`是一个函数接口，所以可以使用 lambda 表达式内联定义处理程序:

```java
Map proxyInstance = (Map) Proxy.newProxyInstance(
  DynamicProxyTest.class.getClassLoader(), 
  new Class[] { Map.class }, 
  (proxy, method, methodArgs) -> {
    if (method.getName().equals("get")) {
        return 42;
    } else {
        throw new UnsupportedOperationException(
          "Unsupported method: " + method.getName());
    }
});
```

这里，我们定义了一个处理程序，对所有 get 操作返回 42，对其他操作抛出`UnsupportedOperationException`。

它的调用方式完全相同:

```java
(int) proxyInstance.get("hello"); // 42
proxyInstance.put("hello", "world"); // exception
```

## 5。定时动态代理示例

让我们研究一个潜在的动态代理真实场景。

假设我们想记录函数执行的时间。为此，我们首先定义一个能够包装“真实”对象、跟踪计时信息和反射调用的处理程序:

```java
public class TimingDynamicInvocationHandler implements InvocationHandler {

    private static Logger LOGGER = LoggerFactory.getLogger(
      TimingDynamicInvocationHandler.class);

    private final Map<String, Method> methods = new HashMap<>();

    private Object target;

    public TimingDynamicInvocationHandler(Object target) {
        this.target = target;

        for(Method method: target.getClass().getDeclaredMethods()) {
            this.methods.put(method.getName(), method);
        }
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) 
      throws Throwable {
        long start = System.nanoTime();
        Object result = methods.get(method.getName()).invoke(target, args);
        long elapsed = System.nanoTime() - start;

        LOGGER.info("Executing {} finished in {} ns", method.getName(), 
          elapsed);

        return result;
    }
}
```

随后，此代理可用于各种对象类型:

```java
Map mapProxyInstance = (Map) Proxy.newProxyInstance(
  DynamicProxyTest.class.getClassLoader(), new Class[] { Map.class }, 
  new TimingDynamicInvocationHandler(new HashMap<>()));

mapProxyInstance.put("hello", "world");

CharSequence csProxyInstance = (CharSequence) Proxy.newProxyInstance(
  DynamicProxyTest.class.getClassLoader(), 
  new Class[] { CharSequence.class }, 
  new TimingDynamicInvocationHandler("Hello World"));

csProxyInstance.length()
```

这里，我们代理了一个映射和一个字符序列(字符串)。

代理方法的调用将委托给包装的对象，并产生日志记录语句:

```java
Executing put finished in 19153 ns 
Executing get finished in 8891 ns 
Executing charAt finished in 11152 ns 
Executing length finished in 10087 ns
```

## 6。结论

在这篇快速教程中，我们研究了 Java 的动态代理以及它的一些可能的用法。

和往常一样，示例中的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221026145528/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection)