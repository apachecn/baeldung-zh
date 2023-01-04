# 用 Spring MVC 显示 RSS 提要

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-rss-feed>

## 1。简介

这个快速教程将展示如何使用 Spring MVC 和`AbstractRssFeedView`类构建一个简单的 RSS 提要。

之后，我们还将实现一个简单的 REST API——通过网络公开我们的提要。

## 2。RSS 源

在讨论实现细节之前，让我们快速回顾一下什么是 RSS 以及它是如何工作的。

RSS 是一种网络订阅源，它可以让用户轻松跟踪网站的更新。此外， **RSS 提要是基于一个 XML 文件的，它概括了一个站点的内容。**然后，新闻聚合器可以订阅一个或多个提要，并通过定期检查 XML 是否发生变化来显示更新。

## 3。依赖性

首先，由于 **Spring 的 RSS 支持是基于罗马框架**，[的，在我们实际使用它之前，我们需要将它作为一个依赖](https://web.archive.org/web/20220526041127/https://search.maven.org/classic/#artifactdetails%7Ccom.rometools%7Crome%7C1.10.0%7Cjar)添加到我们的`pom `中:

```java
<dependency>
    <groupId>com.rometools</groupId>
    <artifactId>rome</artifactId>
    <version>1.10.0</version>
</dependency>
```

关于罗马的指南，请看我们之前的文章。

## 4。进料实施

接下来，我们将构建实际的提要。为了做到这一点，**我们将扩展`AbstractRssFeedView`类并实现它的两个方法。**

**第一个将接收一个`Channel`对象作为输入，并用提要的元数据填充它。**

**另一个将返回代表提要内容的项目列表**:

```java
@Component
public class RssFeedView extends AbstractRssFeedView {

    @Override
    protected void buildFeedMetadata(Map<String, Object> model, 
      Channel feed, HttpServletRequest request) {
        feed.setTitle("Baeldung RSS Feed");
        feed.setDescription("Learn how to program in Java");
        feed.setLink("http://www.baeldung.com");
    }

    @Override
    protected List<Item> buildFeedItems(Map<String, Object> model, 
      HttpServletRequest request, HttpServletResponse response) {
        Item entryOne = new Item();
        entryOne.setTitle("JUnit 5 @Test Annotation");
        entryOne.setAuthor("[[email protected]](/web/20220526041127/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        entryOne.setLink("http://www.baeldung.com/junit-5-test-annotation");
        entryOne.setPubDate(Date.from(Instant.parse("2017-12-19T00:00:00Z")));
        return Arrays.asList(entryOne);
    }
}
```

## 5。曝光进给

最后，我们将构建一个简单的 REST 服务来使我们的提要在 web 上可用。该服务将返回我们刚刚创建的视图对象:

```java
@RestController
public class RssFeedController {

    @Autowired
    private RssFeedView view;

    @GetMapping("/rss")
    public View getFeed() {
        return view;
    }
}
```

此外，由于我们使用 Spring Boot 来启动我们的应用程序，我们将实现一个简单的启动器类:

```java
@SpringBootApplication
public class RssFeedApplication {
    public static void main(final String[] args) {
        SpringApplication.run(RssFeedApplication.class, args);
    }
}
```

运行我们的应用程序后，当执行对我们的服务的请求时，我们将看到以下 RSS 提要:

```java
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
    <channel>
        <title>Baeldung RSS Feed</title>
        <link>http://www.baeldung.com</link>
        <description>Learn how to program in Java</description>
        <item>
            <title>JUnit 5 @Test Annotation</title>
            <link>http://www.baeldung.com/junit-5-test-annotation</link>
            <pubDate>Tue, 19 Dec 2017 00:00:00 GMT</pubDate>
            <author>[[email protected]](/web/20220526041127/https://www.baeldung.com/cdn-cgi/l/email-protection)</author>
        </item>
    </channel>
</rss>
```

## 6。结论

本文介绍了如何使用 Spring 和 ROME 构建一个简单的 RSS 提要，并通过使用 Web 服务使其对消费者可用。

在我们的例子中，我们使用 Spring Boot 来启动我们的应用程序。关于这个话题的更多细节，请继续阅读这篇关于 Spring Boot 的介绍性文章。

和往常一样，所有使用的代码都可以在 GitHub 上获得[。](https://web.archive.org/web/20220526041127/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc)