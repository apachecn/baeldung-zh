# 轻松安排到 Reddit

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/schedule-to-reddit-from-website-feed>

## 1。概述

在这篇文章中，我们将[继续案例研究](/web/20220117081608/https://www.baeldung.com/case-study-a-reddit-app-with-spring)并且**为 Reddit 应用程序**添加一个新特性，目的是使文章调度变得更加简单。

用户现在可以只从一些喜欢的网站向 Reddit 发布文章，而不是在 schedule UI 中手动缓慢地添加每篇文章。我们将使用 RSS 来做到这一点。

## 2。`Site`实体

首先，让我们创建一个实体来代表站点:

```
@Entity
public class Site {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String url;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
}
```

请注意，`url`字段代表**的 URL，即**网站的 RSS 提要。

## 3。存储库和服务

接下来–让我们创建存储库，以便与新的站点实体一起工作:

```
public interface SiteRepository extends JpaRepository<Site, Long> {
    List<Site> findByUser(User user);
}
```

以及服务:

```
public interface ISiteService {

    List<Site> getSitesByUser(User user);

    void saveSite(Site site);

    Site findSiteById(Long siteId);

    void deleteSiteById(Long siteId);
}
```

```
@Service
public class SiteService implements ISiteService {

    @Autowired
    private SiteRepository repo;

    @Override
    public List<Site> getSitesByUser(User user) {
        return repo.findByUser(user);
    }

    @Override
    public void saveSite(Site site) {
        repo.save(site);
    }

    @Override
    public Site findSiteById(Long siteId) {
        return repo.findOne(siteId);
    }

    @Override
    public void deleteSiteById(Long siteId) {
        repo.delete(siteId);
    }
}
```

## 4。从进给加载数据

现在，让我们看看如何使用罗马图书馆从网站提要加载文章细节。

我们首先需要将罗马加入我们的`pom.xml`:

```
<dependency>
    <groupId>com.rometools</groupId>
    <artifactId>rome</artifactId>
    <version>1.5.0</version>
</dependency>
```

然后用它来解析网站的提要:

```
public List<SiteArticle> getArticlesFromSite(Long siteId) {
    Site site = repo.findOne(siteId);
    return getArticlesFromSite(site);
}

List<SiteArticle> getArticlesFromSite(Site site) {
    List<SyndEntry> entries;
    try {
        entries = getFeedEntries(site.getUrl());
    } catch (Exception e) {
        throw new FeedServerException("Error Occurred while parsing feed", e);
    }
    return parseFeed(entries);
}

private List<SyndEntry> getFeedEntries(String feedUrl) 
  throws IllegalArgumentException, FeedException, IOException {
    URL url = new URL(feedUrl);
    SyndFeed feed = new SyndFeedInput().build(new XmlReader(url));
    return feed.getEntries();
}

private List<SiteArticle> parseFeed(List<SyndEntry> entries) {
    List<SiteArticle> articles = new ArrayList<SiteArticle>();
    for (SyndEntry entry : entries) {
        articles.add(new SiteArticle(
          entry.getTitle(), entry.getLink(), entry.getPublishedDate()));
    }
    return articles;
} 
```

最后，这是我们将在回应中使用的简单 DTO:

```
public class SiteArticle {
    private String title;
    private String link;
    private Date publishDate;
}
```

## 5。异常处理

请注意，在解析提要时，我们是如何将整个解析逻辑包装到一个`try-catch`块中的——在出现异常(任何异常)的情况下——我们包装它并抛出它。

原因很简单—**我们需要控制解析过程**抛出的异常类型——这样我们就可以处理该异常并向 API 的客户端提供适当的响应:

```
@ExceptionHandler({ FeedServerException.class })
public ResponseEntity<Object> handleFeed(RuntimeException ex, WebRequest request) {
    logger.error("500 Status Code", ex);
    String bodyOfResponse = ex.getLocalizedMessage();
    return new ResponseEntity<Object>(bodyOfResponse, new HttpHeaders(), 
      HttpStatus.INTERNAL_SERVER_ERROR);
}
```

## 6。网站页面

### 6.1。显示站点

首先，我们将看到如何显示属于登录用户的站点列表:

```
@RequestMapping(value = "/sites")
@ResponseBody
public List<Site> getSitesList() {
    return service.getSitesByUser(getCurrentUser());
}
```

这是非常简单的前端:

```
<table>
<thead>
<tr><th>Site Name</th><th>Feed URL</th><th>Actions</th></tr>
</thead>
</table>
<script>
$(function(){
	$.get("sites", function(data){
        $.each(data, function( index, site ) {
            $('.table').append('<tr><td>'+site.name+'</td><td>'+site.url+
              '</td><td><a href="#" onclick="deleteSite('+site.id+') ">Delete</a> </td></tr>');
        });
    });
});

function deleteSite(id){
    $.ajax({ url: 'sites/'+id, type: 'DELETE', success: function(result) {
            window.location.href="mysites"
        }
    });
}
</script>
```

### 6.2。添加新站点

接下来，让我们看看用户如何创建新的收藏夹站点:

```
@RequestMapping(value = "/sites", method = RequestMethod.POST)
@ResponseStatus(HttpStatus.OK)
public void addSite(Site site) {
    if (!service.isValidFeedUrl(site.getUrl())) {
        throw new FeedServerException("Invalid Feed Url");
    }
    site.setUser(getCurrentUser());
    service.saveSite(site);
}
```

这也是非常简单的客户端:

```
<form>
    <input name="name" />
    <input id="url" name="url" />
    <button type="submit" onclick="addSite()">Add Site</button>
</form>

<script>
function addSite(){
    $.post("sites",$('form').serialize(), function(data){
        window.location.href="mysites";
    }).fail(function(error){
        alert(error.responseText);
    });
}
</script>
```

### 6.3。验证提要

新提要的验证是一个有点昂贵的操作——我们需要实际检索提要并解析它以完全验证它。下面是简单的服务方法:

```
public boolean isValidFeedUrl(String feedUrl) {
    try {
        return getFeedEntries(feedUrl).size() > 0;
    } catch (Exception e) {
        return false;
    }
}
```

### 6.3。删除一个站点

现在，让我们看看用户如何从他们最喜欢的站点列表中删除一个站点:

```
@RequestMapping(value = "/sites/{id}", method = RequestMethod.DELETE)
@ResponseStatus(HttpStatus.OK)
public void deleteSite(@PathVariable("id") Long id) {
    service.deleteSiteById(id);
}
```

这里是非常简单的服务级别方法:

```
public void deleteSiteById(Long siteId) {
    repo.delete(siteId);
}
```

## 7。从一个站点安排一篇文章

现在，让我们实际上开始使用这些网站，并实现一个基本的方法，用户可以安排一个新的帖子发布到 Reddit，不是手动的，而是通过从现有网站加载一篇文章。

### 7.1。修改排班表

让我们从客户站点开始，修改现有的`schedulePostForm.html` ——我们将添加:

```
<button data-target="#myModal">Load from My Sites</button>
<div id="myModal">
    <button id="dropdownMenu1">Choose Site</button><ul id="siteList"></ul>
    <button id="dropdownMenu2">Choose Article</button><ul id="articleList"></ul>
    <button onclick="load()">Load</button>
</div>
```

请注意，我们添加了:

*   按钮—“`Load from my Sites`”—启动程序
*   弹出窗口-显示网站及其文章的列表

### 7.2。加载站点

使用一点 javascript，在弹出窗口中加载站点相对容易:

```
$('#myModal').on('shown.bs.modal', function () {
    if($("#siteList").children().length > 0)
        return;
    $.get("sites", function(data){
        $.each(data, function( index, site ) {
            $("#siteList").append('<li><a href="#" onclick="loadArticles('+
              site.id+',\''+site.name+'\')">'+site.name+'</a></li>')
	});
    });
});
```

### 7.3。加载一个站点的帖子

当用户从列表中选择一个网站时，我们需要显示该网站的文章——同样使用一些基本的 js:

```
function loadArticles(siteID,siteName){
    $("#dropdownMenu1").html(siteName);
    $.get("sites/articles?id="+siteID, function(data){
        $("#articleList").html('');
        $("#dropdownMenu2").html('Choose Article');
    $.each(data, function( index, article ) {
        $("#articleList").append(
              '<li><a href="#" onclick="chooseArticle(\''+article.title+
              '\',\''+article.link+'\')"><b>'+article.title+'</b> <small>'+
              new Date(article.publishDate).toUTCString()+'</small></li>')
    });
    }).fail(function(error){ 
        alert(error.responseText); 
    });
}
```

这当然与加载网站文章的简单服务器端操作挂钩:

```
@RequestMapping(value = "/sites/articles")
@ResponseBody
public List<SiteArticle> getSiteArticles(@RequestParam("id") Long siteId) {
    return service.getArticlesFromSite(siteId);
}
```

最后，我们获取文章数据，填写表单并安排文章发送到 Reddit:

```
var title = "";
var link = "";
function chooseArticle(selectedTitle,selectedLink){
    $("#dropdownMenu2").html(selectedTitle);
    title=selectedTitle;
    link = selectedLink;
}
function load(){
    $("input[name='title']").val(title);
    $("input[name='url']").val(link);
}
```

## 8。集成测试

最后，让我们在两种不同的提要格式上测试我们的`SiteService`:

```
public class SiteIntegrationTest {

    private ISiteService service;

    @Before
    public void init() {
        service = new SiteService();
    }

    @Test
    public void whenUsingServiceToReadWordpressFeed_thenCorrect() {
        Site site = new Site("/feed/");
        List<SiteArticle> articles = service.getArticlesFromSite(site);

        assertNotNull(articles);
        for (SiteArticle article : articles) {
            assertNotNull(article.getTitle());
            assertNotNull(article.getLink());
        }
    }

    @Test
    public void whenUsingRomeToReadBloggerFeed_thenCorrect() {
        Site site = new Site("http://blogname.blogspot.com/feeds/posts/default");
        List<SiteArticle> articles = service.getArticlesFromSite(site);

        assertNotNull(articles);
        for (SiteArticle article : articles) {
            assertNotNull(article.getTitle());
            assertNotNull(article.getLink());
        }
    }
}
```

这里显然有一点重复，但我们可以稍后再处理。

## 9。结论

在这一期中，我们重点关注了一个新的小特性——让发布到 Reddit 的时间安排变得更简单。