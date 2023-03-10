# 带点的 Spring MVC @PathVariable(。)被截断

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-pathvariable-dot>

## 1。概述

在这个简短的教程中，我们将讨论使用 Spring MVC 时的一个常见问题——**当使用 Spring `@PathVariable`和一个`@RequestMapping`来映射包含一个点的请求 URI 的结尾时，我们将在变量中结束一个部分值，在最后一个点处被截断。**

在接下来的部分中，我们将关注为什么会发生这种情况以及如何改变这种行为。

关于 Spring MVC 的介绍，请参考本文中的[。](/web/20220626201006/https://www.baeldung.com/spring-mvc-tutorial)

## 2。不需要的弹簧帮助

由于框架解释 path 变量的方式，导致了这种通常不需要的行为。

具体来说， **Spring 认为最后一个点后面的任何东西都是文件扩展名**，比如`.json`或`.xml.`

因此，它会截断值以检索参数。

让我们看一个使用路径变量的例子，然后分析不同可能值的结果:

```java
@RestController
public class CustomController {
    @GetMapping("/example/{firstValue}/{secondValue}")
    public void example(@PathVariable("firstValue") String firstValue,
      @PathVariable("secondValue") String secondValue) {
        // ...  
    }
}
```

在上面的例子中，让我们考虑下一个请求并评估我们的变量:

*   网址`example/gallery/link` 导致评估`firstValue =`“画廊”和`secondValue =`“链接”
*   当使用`example/gallery.df/link.ar`网址时，我们会有`firstValue`=“gallery . df”和`secondValue` =“链接”
*   使用 `example/gallery.df/link.com.ar` URL，我们的变量将是:`firstValue`=“gallery . df”和`secondValue`=“link . com”

正如我们所见，第一个变量不受影响，但第二个变量总是被截断。

## 3。解决方案

解决这种不便的一个方法是通过添加一个正则表达式映射来修改我们的`@PathVariable`定义。因此，任何点，包括最后一个点，都将被视为我们参数的一部分:

```java
@GetMapping("/example/{firstValue}/{secondValue:.+}")   
public void example(
  @PathVariable("firstValue") String firstValue,
  @PathVariable("secondValue") String secondValue) {
    //...
}
```

另一种避免这个问题的方法是在我们的`@PathVariable` 的末尾添加一个斜线。这将包含我们的第二个变量，保护它不受 Spring 默认行为的影响:

```java
@GetMapping("/example/{firstValue}/{secondValue}/")
```

上面的两个解决方案适用于我们正在修改的单个请求映射。

**如果我们想在全局 MVC 水平上改变行为，我们需要提供一个定制配置**。为此，我们可以扩展`WebMvcConfigurationSupport`并覆盖它的`getPathMatchConfigurer()`方法来调整一个`PathMatchConfigurer`。

```java
@Configuration
public class CustomWebMvcConfigurationSupport extends WebMvcConfigurationSupport {

    @Override
    protected PathMatchConfigurer getPathMatchConfigurer() {
        PathMatchConfigurer pathMatchConfigurer = super.getPathMatchConfigurer();
        pathMatchConfigurer.setUseSuffixPatternMatch(false);

        return pathMatchConfigurer;
    }
}
```

我们必须记住，这种方法影响所有的网址。

使用这三个选项，我们将获得相同的结果:当调用`example/gallery.df/link.com.ar` URL 时，我们的`secondValue`变量将被计算为“link.com.ar”，这就是我们想要的。

### 3.1.弃用通知

从 Spring Framework [5.2.4](https://web.archive.org/web/20220626201006/https://github.com/spring-projects/spring-framework/issues/24179) ，**开始，`setUseSuffixPatternMatch(boolean)`方法被弃用，目的是阻止使用路径扩展进行请求路由和内容协商。**基本上，目前的实现方式很难保护 web 应用程序免受[反射文件下载(RFD)](https://web.archive.org/web/20220626201006/https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-rfd) 攻击。

此外，从 Spring Framework 5.3 开始，后缀模式匹配将只对显式注册的后缀有效，以防止任意扩展。

底线是，从 Spring 5.3 开始，我们将不再需要使用`setUseSuffixPatternMatch(false) `，因为默认情况下它是禁用的。

## 4。结论

在这篇快速的文章中，我们探索了在 Spring MVC 中使用`@PathVariable`和`@RequestMapping`时解决一个常见问题的不同方法，以及这个问题的根源。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220626201006/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java-2)