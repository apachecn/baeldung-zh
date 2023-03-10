# 爬虫指南 j

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/crawler4j>

## 1.介绍

每次使用我们最喜欢的搜索引擎时，我们都会看到网络爬虫在使用。它们也常用于从网站上收集和分析数据。

在本教程中，我们将学习如何使用 [crawler4j](https://web.archive.org/web/20221101005732/https://github.com/yasserg/crawler4j) 来设置和运行我们自己的网络爬虫。crawler4j 是一个开源的 Java 项目，它让我们可以很容易地做到这一点。

## 2.设置

让我们使用 [Maven Central](https://web.archive.org/web/20221101005732/https://search.maven.org/search?q=a:crawler4j%20AND%20g:edu.uci.ics) 来查找最新版本并引入 Maven 依赖项:

```java
<dependency>
    <groupId>edu.uci.ics</groupId>
    <artifactId>crawler4j</artifactId>
    <version>4.4.0</version>
</dependency>
```

## 3.创建爬网程序

### 3.1.简单的 HTML 爬虫

我们将从创建一个基本的爬虫开始，这个爬虫抓取 [`https://baeldung.com`](https://web.archive.org/web/20221101005732/https://baeldung.com/) 上的 HTML 页面。

让我们通过扩展 crawler 类中的`WebCrawler`并定义一个排除某些文件类型的模式来创建我们的 crawler:

```java
public class HtmlCrawler extends WebCrawler {

    private final static Pattern EXCLUSIONS
      = Pattern.compile(".*(\\.(css|js|xml|gif|jpg|png|mp3|mp4|zip|gz|pdf))$");

    // more code
}
```

在每个爬虫类中，**我们必须覆盖并实现两个方法:`shouldVisit`和`visit`。**

现在让我们使用我们创建的`EXCLUSIONS`模式来创建我们的`shouldVisit`方法:

```java
@Override
public boolean shouldVisit(Page referringPage, WebURL url) {
    String urlString = url.getURL().toLowerCase();
    return !EXCLUSIONS.matcher(urlString).matches() 
      && urlString.startsWith("https://www.baeldung.com/");
}
```

然后，我们可以在`visit`方法中对访问过的页面进行处理:

```java
@Override
public void visit(Page page) {
    String url = page.getWebURL().getURL();

    if (page.getParseData() instanceof HtmlParseData) {
        HtmlParseData htmlParseData = (HtmlParseData) page.getParseData();
        String title = htmlParseData.getTitle();
        String text = htmlParseData.getText();
        String html = htmlParseData.getHtml();
        Set<WebURL> links = htmlParseData.getOutgoingUrls();

        // do something with the collected data
    }
}
```

一旦我们写好了爬虫，我们需要配置并运行它:

```java
File crawlStorage = new File("src/test/resources/crawler4j");
CrawlConfig config = new CrawlConfig();
config.setCrawlStorageFolder(crawlStorage.getAbsolutePath());

int numCrawlers = 12;

PageFetcher pageFetcher = new PageFetcher(config);
RobotstxtConfig robotstxtConfig = new RobotstxtConfig();
RobotstxtServer robotstxtServer= new RobotstxtServer(robotstxtConfig, pageFetcher);
CrawlController controller = new CrawlController(config, pageFetcher, robotstxtServer);

controller.addSeed("https://www.baeldung.com/");

CrawlController.WebCrawlerFactory<HtmlCrawler> factory = HtmlCrawler::new;

controller.start(factory, numCrawlers);
```

我们配置了一个临时存储目录，指定了爬行线程的数量，并用一个起始 URL 作为爬虫的种子。

我们还应该注意到**`CrawlController.start()`方法是一个阻塞操作**。该调用之后的任何代码将仅在 crawler 完成运行后执行。

### 3.2.`ImageCrawler`

默认情况下，crawler4j 不会对二进制数据进行爬网。在下一个例子中，我们将打开该功能并**抓取 Baeldung 上的所有 JPEGs。**

让我们首先用一个构造函数定义`ImageCrawler`类，该构造函数采用一个保存图像的目录:

```java
public class ImageCrawler extends WebCrawler {
    private final static Pattern EXCLUSIONS
      = Pattern.compile(".*(\\.(css|js|xml|gif|png|mp3|mp4|zip|gz|pdf))$");

    private static final Pattern IMG_PATTERNS = Pattern.compile(".*(\\.(jpg|jpeg))$");

    private File saveDir;

    public ImageCrawler(File saveDir) {
        this.saveDir = saveDir;
    }

    // more code

}
```

接下来，让我们实现`shouldVisit`方法:

```java
@Override
public boolean shouldVisit(Page referringPage, WebURL url) {
    String urlString = url.getURL().toLowerCase();
    if (EXCLUSIONS.matcher(urlString).matches()) {
        return false;
    }

    if (IMG_PATTERNS.matcher(urlString).matches() 
        || urlString.startsWith("https://www.baeldung.com/")) {
        return true;
    }

    return false;
}
```

现在，我们准备实现`visit`方法:

```java
@Override
public void visit(Page page) {
    String url = page.getWebURL().getURL();
    if (IMG_PATTERNS.matcher(url).matches() 
        && page.getParseData() instanceof BinaryParseData) {
        String extension = url.substring(url.lastIndexOf("."));
        int contentLength = page.getContentData().length;

        // write the content data to a file in the save directory
    }
}
```

运行我们的`ImageCrawler`类似于运行`HttpCrawler`，但是我们需要配置它以包含二进制内容:

```java
CrawlConfig config = new CrawlConfig();
config.setIncludeBinaryContentInCrawling(true);

// ... same as before

CrawlController.WebCrawlerFactory<ImageCrawler> factory = () -> new ImageCrawler(saveDir);

controller.start(factory, numCrawlers);
```

### 3.3.收集数据

既然我们已经看了几个基本的例子，让我们扩展一下我们的`HtmlCrawler`来收集一些爬行期间的基本统计数据。

首先，让我们定义一个简单的类来保存一些统计数据:

```java
public class CrawlerStatistics {
    private int processedPageCount = 0;
    private int totalLinksCount = 0;

    public void incrementProcessedPageCount() {
        processedPageCount++;
    }

    public void incrementTotalLinksCount(int linksCount) {
        totalLinksCount += linksCount;
    }

    // standard getters
}
```

接下来，让我们修改我们的`HtmlCrawler`,通过构造函数接受一个`CrawlerStatistics`实例:

```java
private CrawlerStatistics stats;

public HtmlCrawler(CrawlerStatistics stats) {
    this.stats = stats;
}
```

有了新的`CrawlerStatistics`对象，让我们修改`visit`方法来收集我们想要的东西:

```java
@Override
public void visit(Page page) {
    String url = page.getWebURL().getURL();
    stats.incrementProcessedPageCount();

    if (page.getParseData() instanceof HtmlParseData) {
        HtmlParseData htmlParseData = (HtmlParseData) page.getParseData();
        String title = htmlParseData.getTitle();
        String text = htmlParseData.getText();
        String html = htmlParseData.getHtml();
        Set<WebURL> links = htmlParseData.getOutgoingUrls();
        stats.incrementTotalLinksCount(links.size());

        // do something with collected data
    }
}
```

现在，让我们回到控制器，给`HtmlCrawler`提供一个`CrawlerStatistics`的实例:

```java
CrawlerStatistics stats = new CrawlerStatistics();
CrawlController.WebCrawlerFactory<HtmlCrawler> factory = () -> new HtmlCrawler(stats);
```

### 3.4.多个爬虫

在前面例子的基础上，现在让我们看看如何从同一个控制器运行多个爬虫。

建议每个爬虫使用自己的临时存储目录，所以我们需要为将要运行的每个爬虫创建单独的配置。

`CrawlControllers`可以共享一个`RobotstxtServer`，但是除此之外，我们基本上需要所有东西的副本。

到目前为止，我们已经使用了`CrawlController.start`方法来运行我们的爬虫，并注意到这是一个阻塞方法。**为了运行多个，我们将结合使用`CrawlerControlller.startNonBlocking`和`CrawlController.waitUntilFinish`。**

现在，让我们创建一个控制器来同时运行`HtmlCrawler`和`ImageCrawler`:

```java
File crawlStorageBase = new File("src/test/resources/crawler4j");
CrawlConfig htmlConfig = new CrawlConfig();
CrawlConfig imageConfig = new CrawlConfig();

// Configure storage folders and other configurations

PageFetcher pageFetcherHtml = new PageFetcher(htmlConfig);
PageFetcher pageFetcherImage = new PageFetcher(imageConfig);

RobotstxtConfig robotstxtConfig = new RobotstxtConfig();
RobotstxtServer robotstxtServer = new RobotstxtServer(robotstxtConfig, pageFetcherHtml);

CrawlController htmlController
  = new CrawlController(htmlConfig, pageFetcherHtml, robotstxtServer);
CrawlController imageController
  = new CrawlController(imageConfig, pageFetcherImage, robotstxtServer);

// add seed URLs

CrawlerStatistics stats = new CrawlerStatistics();
CrawlController.WebCrawlerFactory<HtmlCrawler> htmlFactory = () -> new HtmlCrawler(stats);

File saveDir = new File("src/test/resources/crawler4j");
CrawlController.WebCrawlerFactory<ImageCrawler> imageFactory
  = () -> new ImageCrawler(saveDir);

imageController.startNonBlocking(imageFactory, 7);
htmlController.startNonBlocking(htmlFactory, 10);

htmlController.waitUntilFinish();
imageController.waitUntilFinish();
```

## 4.配置

我们已经看到了一些我们可以配置的内容。现在，让我们来看看其他一些常见的设置。

设置被应用到我们在控制器中指定的`CrawlConfig`实例。

### 4.1.限制爬行深度

默认情况下，我们的爬虫会尽可能深地爬行。为了限制它们前进的深度，我们可以设置爬行深度:

```java
crawlConfig.setMaxDepthOfCrawling(2);
```

种子 URL 被视为深度为 0，因此爬网深度为 2 将超出种子 URL 两层。

### 4.2.要获取的最大页数

另一种限制爬网程序覆盖的页数的方法是设置要爬网的最大页数:

```java
crawlConfig.setMaxPagesToFetch(500);
```

### 4.3.最大传出链接

我们还可以限制每个页面的链接数量:

```java
crawlConfig.setMaxOutgoingLinksToFollow(2000);
```

### 4.4.礼貌延迟

由于非常高效的爬虫很容易成为网络服务器的负担，crawler4j 有一个所谓的礼貌延迟。默认情况下，它被设置为 200 毫秒。如果需要，我们可以调整该值:

```java
crawlConfig.setPolitenessDelay(300);
```

### 4.5.包括二进制内容

我们已经在`ImageCrawler`中使用了包含二进制内容的选项:

```java
crawlConfig.setIncludeBinaryContentInCrawling(true);
```

### 4.6.包括 HTTPS

默认情况下，爬网程序将包括 HTTPS 页面，但我们可以将其关闭:

```java
crawlConfig.setIncludeHttpsPages(false);
```

### 4.7.可恢复爬行

如果我们有一个长时间运行的爬虫，并且我们想让它自动恢复，我们可以设置可恢复的爬行。打开它可能会导致它运行速度变慢:

```java
crawlConfig.setResumableCrawling(true);
```

### 4.8.用户代理字符串

crawler4j 的默认用户代理字符串是`[crawler4j](https://web.archive.org/web/20221101005732/https://github.com/yasserg/crawler4j/)`。让我们定制一下:

```java
crawlConfig.setUserAgentString("baeldung demo (https://github.com/yasserg/crawler4j/)");
```

我们刚刚在这里介绍了一些基本配置。如果我们对一些更高级或晦涩的配置选项感兴趣，我们可以看看`[CrawConfig](https://web.archive.org/web/20221101005732/https://github.com/yasserg/crawler4j/blob/master/crawler4j/src/main/java/edu/uci/ics/crawler4j/crawler/CrawlConfig.java)`类。

## 5.结论

在本文中，我们使用 crawler4j 创建了自己的网络爬虫。我们从抓取 HTML 和图像的两个简单例子开始。然后，我们在这些例子的基础上，看看如何收集统计数据并同时运行多个爬虫。

GitHub 上的[提供了完整的代码示例。](https://web.archive.org/web/20221101005732/https://github.com/eugenp/tutorials/tree/master/libraries-2)