# Spring Boot 贫民窟指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-favicon>

## 1。概述

favicon 是浏览器中显示的一个小网站图标，通常位于地址旁边。

通常我们不想满足于各种框架提供的默认设置，比如 Spring Boot。

在这个快速教程中，我们将讨论如何**定制一个 Spring Boot 应用**的 favicon，通过研究各种定制 favicon 的方法。

## 2。覆盖收藏夹图标

覆盖 Spring Boot 应用程序的默认图标的最简单的方法是**将新的图标放在`resources`目录**中:

```java
src/main/resources/favicon.ico 
```

favicon 文件应该具有“`favicon.ico”` 名称。

我们也可以将该文件放在项目资源目录中的`static`目录中:

```java
src/main/resources/static/favicon.ico
```

Spring Boot 在启动时，扫描根资源位置中的`favicon.ico `文件，然后是静态内容位置。

## 3.使用自定义位置

我们可能不想将 favicon 放在 resources 目录的根目录中，而是希望将它与应用程序的其他图像放在一起。

我们可以通过禁用我们的`application.properties `文件中的默认 favicon 来做到这一点:

```java
spring.mvc.favicon.enabled=false
```

值得一提的是，从 Spring Boot 2.2 开始，这个配置属性已被弃用。此外，Spring Boot 不再提供默认的 favicon，因为该图标可归类为[信息泄露。](https://web.archive.org/web/20220628094402/https://github.com/spring-projects/spring-boot/issues/17925)

然后实现我们的处理程序:

```java
@Configuration
public class FaviconConfiguration {

    @Bean
    public SimpleUrlHandlerMapping customFaviconHandlerMapping() {
        SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
        mapping.setOrder(Integer.MIN_VALUE);
        mapping.setUrlMap(Collections.singletonMap(
          "/favicon.ico", faviconRequestHandler()));
        return mapping;
    }

    @Bean
    protected ResourceHttpRequestHandler faviconRequestHandler() {
        ResourceHttpRequestHandler requestHandler
          = new ResourceHttpRequestHandler();
        ClassPathResource classPathResource 
          = new ClassPathResource("com/baelduimg/");
        List<Resource> locations = Arrays.asList(classPathResource);
        requestHandler.setLocations(locations);
        return requestHandler;
    }
}
```

注意，我们已经为映射顺序设置了`Integer.MIN_VALUE `，所以给这个处理程序最高的优先级。

有了这个配置，**我们可以将 favicon 文件存储在应用程序结构**中的任何位置。

## 4.优雅地禁用 Favicon

如果我们的应用程序不需要任何 favicon，我们可以通过将属性`spring.mvc.favicon.enabled `设置为 false 来禁用它。但是当浏览器查找时，会得到一个“404 未找到”的错误。

我们可以使用定制的 favicon 控制器**来避免这种情况，它返回一个空响应**:

```java
//...

@Controller
static class FaviconController {

    @GetMapping("favicon.ico")
    @ResponseBody
    void returnNoFavicon() {
    }
}

//...
```

## 5.结论

在本文中，我们看到了如何覆盖 Spring boot 应用程序的默认 favicon，为 favicon 使用自定义位置，以及如果我们不想使用 favicon，如何避免 404 错误。

像往常一样，代码样本可以在 GitHub 上获得。