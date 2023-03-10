# 谷歌自动服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/google-autoservice>

## 1.介绍

在这个快速教程中，我们将简要解释谷歌的自动服务。

这是一个[注释处理器库](/web/20220626203526/https://www.baeldung.com/java-annotation-processing-builder)，它帮助我们生成 [Java 服务提供者接口](/web/20220626203526/https://www.baeldung.com/java-spi) (SPI)配置文件。

## 2.Java SPI

简单地说，我们可以利用 Java SPI 来开发可扩展的应用程序，因为它提供了快速、安全和动态的定制。

Java SPI 使用配置文件来查找和加载给定服务提供者接口的具体实现。动态定制应用程序是其主要功能之一。

另一方面，添加或编辑配置文件很容易出错，也让我们有点困惑。这一步也容易忘记。

此外，总有我们可能没有注意到的打字错误的风险，因为编译器不考虑配置文件。

## 3.谷歌自动服务

Google AutoService 是一个开源代码生成器工具，在 Google Auto 项目下开发。除了 AutoService 还有另外两个工具: [AutoValue](/web/20220626203526/https://www.baeldung.com/introduction-to-autovalue) 和 [AutoFactory](/web/20220626203526/https://www.baeldung.com/autofactory) 。

**这个库的目的是节省精力和时间**，同时**防止错误配置**。

### 3.1.Maven 设置

首先，让我们在应用程序中添加[自动服务](https://web.archive.org/web/20220626203526/https://search.maven.org/search?q=g:com.google.auto.service%20AND%20a:auto-service&core=gav)依赖项。我们可以将依赖项设置为`optional`，因为我们只在编译时需要它:

```java
<dependency>
    <groupId>com.google.auto.service</groupId>
    <artifactId>auto-service</artifactId>
    <version>1.0-rc5</version>
    <optional>true</optional>
</dependency>
```

### 3.2.`@AutoService`举例

其次，我们将创建一个服务提供者接口。

让我们假设我们的应用程序具有翻译功能。我们的目标是使这个特性可扩展。因此，我们可以轻松插入任何翻译服务提供商组件:

```java
public interface TranslationService {
    String translate(String message, Locale from, Locale to);
}
```

我们的应用程序将使用这个接口作为扩展点。类路径上的实现将作为组件注入。

接下来，我们将使用`@AutoService` 注释，用两个不同的翻译提供者实现这个服务:

```java
@AutoService(TranslationService.class)
public class BingTranslationServiceProvider implements TranslationService {
    @Override
    public String translate(String message, Locale from, Locale to) {
        // implementation details
        return message + " (translated by Bing)"; 
    }
}
```

```java
@AutoService(TranslationService.class)
public class GoogleTranslationServiceProvider implements TranslationService {
    @Override
    public String translate(String message, Locale from, Locale to) {
        // implementation details
        return message + " (translated by Google)"; 
    }
}
```

在编译时，AutoService 将查找注释，并为每个相应的接口和实现生成一个配置文件。

因此，我们现在将有一个名为`com.baeldung.autoservice.TranslationService.`的配置文件。该文件包含两个提供者的全限定名称:

```java
com.baeldung.autoservice.BingTranslationServiceProvider
com.baeldung.autoservice.GoogleTranslationServiceProvider
```

### 3.3.`@AutoService`在行动

现在，一切准备就绪。让我们通过`ServiceLoader`加载提供者:

```java
ServiceLoader<TranslationService> loader = ServiceLoader.load(TranslationService.class);
```

`ServiceLoader`将加载配置文件中定义的每个提供者。

让我们检查加载的提供程序数:

```java
long count = StreamSupport.stream(loader.spliterator(), false).count();
assertEquals(2, count);
```

换句话说，`ServiceLoader`已经加载了所有的提供者实例。因此，我们的工作就是选择其中之一。

所以现在，让我们挑选一个提供者，然后调用服务方法来查看加载器是否按预期工作:

```java
TranslationService googleService = StreamSupport.stream(loader.spliterator(), false)
  .filter(p -> p.getClass().getSimpleName().equals("GoogleTranslationServiceProvider"))
  .findFirst()
  .get();

String message = "message";

assertEquals(message + " (translated by Google)", googleService.translate(message, null, null));
```

## 4.结论

在本文中，我们解释了 Google AutoService 库，并通过一个简单的例子进行了实践。

Google AutoService 是一个有用但简单的源代码生成器库。它**使我们免于创建和编辑服务提供商配置文件**。它还保证不会有任何写错或放错位置的文件。

像往常一样，本教程的源代码可以在 GitHub 项目上找到。