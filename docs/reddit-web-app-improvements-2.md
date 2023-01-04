# Reddit 应用程序的第二轮改进

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-web-app-improvements-2>

## 1。概述

让我们继续[正在进行的 Reddit web 应用案例研究](/web/20220120143211/https://www.baeldung.com/case-study-a-reddit-app-with-spring)，进行新一轮的改进，目标是使应用程序更加用户友好和易于使用。

## 2。预定文章分页

首先，让我们用页码列出预定的帖子**，让整个事情更容易看和理解。**

### 2.1。分页操作

我们将使用 Spring 数据来生成我们需要的操作，充分利用`Pageable`接口来检索用户预定的帖子:

```java
public interface PostRepository extends JpaRepository<Post, Long> {
    Page<Post> findByUser(User user, Pageable pageable);
}
```

这是我们的控制器方法`getScheduledPosts()`:

```java
private static final int PAGE_SIZE = 10;

@RequestMapping("/scheduledPosts")
@ResponseBody
public List<Post> getScheduledPosts(
  @RequestParam(value = "page", required = false) int page) {
    User user = getCurrentUser();
    Page<Post> posts = 
      postReopsitory.findByUser(user, new PageRequest(page, PAGE_SIZE));

    return posts.getContent();
}
```

### 2.2。显示分页的帖子

现在，让我们在前端实现一个简单的分页控件:

```java
<table>
<thead><tr><th>Post title</th></thead>
</table>
<br/>
<button id="prev" onclick="loadPrev()">Previous</button> 
<button id="next" onclick="loadNext()">Next</button>
```

下面是我们如何用普通 jQuery 加载页面:

```java
$(function(){ 
    loadPage(0); 
}); 

var currentPage = 0;
function loadNext(){ 
    loadPage(currentPage+1);
} 

function loadPrev(){ 
    loadPage(currentPage-1); 
}

function loadPage(page){
    currentPage = page;
    $('table').children().not(':first').remove();
    $.get("api/scheduledPosts?page="+page, function(data){
        $.each(data, function( index, post ) {
            $('.table').append('<tr><td>'+post.title+'</td><td></tr>');
        });
    });
}
```

随着我们的发展，这个手动表格将很快被一个更成熟的表格插件所取代，但是现在，这个插件工作得很好。

## 3。向未登录的用户显示登录页面

当用户访问根目录时，**他们应该得到不同的页面，不管他们是否登录**。

如果用户已登录，他们应该会看到自己的主页/仪表板。如果他们没有登录，他们应该会看到登录页面:

```java
@RequestMapping("/")
public String homePage() {
    if (SecurityContextHolder.getContext().getAuthentication() != null) {
        return "home";
    }
    return "index";
}
```

## 4。后期重新提交的高级选项

在 Reddit 中删除和重新提交帖子是一个非常有用和高效的功能。然而，我们要小心谨慎，并且**完全控制**我们什么时候应该做什么时候不应该做。

例如，如果一篇文章已经有评论，我们可能不想删除它。归根结底，评论就是参与，我们希望尊重平台和对帖子发表评论的人。

**这是我们将添加的第一个小但非常有用的功能**，这是一个新选项，允许我们只删除没有评论的帖子。

另一个需要回答的非常有趣的问题是——如果帖子被重新提交了很多次，但仍然没有得到它所需要的关注——我们是否应该在最后一次尝试后继续保留它？嗯，就像所有有趣的问题一样，这里的答案是——“视情况而定”。如果这是一个正常的帖子，我们可能会就此收工，然后放弃它。然而，如果这是一个超级重要的帖子，我们真的真的想确保它得到一些关注，我们可能会在最后删除它。

因此，这是我们将在这里构建的第二个小但非常方便的功能。

最后，有争议的帖子怎么办？一个帖子在 reddit 上可以有 2 票，因为它必须有赞成票，或者因为它有 100 张赞成票和 98 张反对票。第一个选项意味着它没有获得牵引力，而第二个选项意味着它获得了很大的牵引力，并且投票是分裂的。

**所以——这是我们要添加的第三个小功能**——这是一个新的选项，在决定我们是否需要删除帖子时，要考虑这种赞成票与反对票的比例。

### 4.1。`Post`实体

首先，我们需要修改我们的`Post`实体:

```java
@Entity
public class Post {
    ...
    private int minUpvoteRatio;
    private boolean keepIfHasComments;
    private boolean deleteAfterLastAttempt;
}
```

这里有 3 个字段:

*   `minUpvoteRatio`:用户希望他的帖子达到的最小投票率——投票率代表总票数的百分比
*   `keepIfHasComments`:如果用户的帖子有评论，即使没有达到要求的分数，也要决定用户是否要保留他的帖子。
*   `deleteAfterLastAttempt`:当最后一次尝试没有达到要求的分数时，确定用户是否要删除帖子。

### 4.2。调度器

现在让我们将这些有趣的新选项集成到调度程序中:

```java
@Scheduled(fixedRate = 3 * 60 * 1000)
public void checkAndDeleteAll() {
    List<Post> submitted = 
      postReopsitory.findByRedditIDNotNullAndNoOfAttemptsAndDeleteAfterLastAttemptTrue(0);

    for (Post post : submitted) {
        checkAndDelete(post);
    }
}
```

关于更有趣的部分——`checkAndDelete()`的实际逻辑:

```java
private void checkAndDelete(Post post) {
    if (didIntervalPass(post.getSubmissionDate(), post.getTimeInterval())) {
        if (didPostGoalFail(post)) {
            deletePost(post.getRedditID());
            post.setSubmissionResponse("Consumed Attempts without reaching score");
            post.setRedditID(null);
            postReopsitory.save(post);
        } else {
            post.setNoOfAttempts(0);
            post.setRedditID(null);
            postReopsitory.save(post);
        }
    }
}
```

下面是`didPostGoalFail()`实现—**检查帖子是否没有达到预定义的目标/分数**:

```java
private boolean didPostGoalFail(Post post) {
    PostScores postScores = getPostScores(post);
    int score = postScores.getScore();
    int upvoteRatio = postScores.getUpvoteRatio();
    int noOfComments = postScores.getNoOfComments();
    return (((score < post.getMinScoreRequired()) || 
             (upvoteRatio < post.getMinUpvoteRatio())) && 
           !((noOfComments > 0) && post.isKeepIfHasComments()));
}
```

我们还需要修改从 Reddit 检索`Post`信息的逻辑，以确保我们收集到更多的数据:

```java
public PostScores getPostScores(Post post) {
    JsonNode node = restTemplate.getForObject(
      "http://www.reddit.com/r/" + post.getSubreddit() + 
      "/comments/" + post.getRedditID() + ".json", JsonNode.class);
    PostScores postScores = new PostScores();

    node = node.get(0).get("data").get("children").get(0).get("data");
    postScores.setScore(node.get("score").asInt());

    double ratio = node.get("upvote_ratio").asDouble();
    postScores.setUpvoteRatio((int) (ratio * 100));

    postScores.setNoOfComments(node.get("num_comments").asInt());

    return postScores;
}
```

当我们从 Reddit API 中提取分数时，我们使用一个简单的值对象来表示分数:

```java
public class PostScores {
    private int score;
    private int upvoteRatio;
    private int noOfComments;
}
```

最后，我们需要修改 `checkAndReSubmit()`，将成功重新提交的帖子的`redditID`设置为`null`:

```java
private void checkAndReSubmit(Post post) {
    if (didIntervalPass(post.getSubmissionDate(), post.getTimeInterval())) {
        if (didPostGoalFail(post)) {
            deletePost(post.getRedditID());
            resetPost(post);
        } else {
            post.setNoOfAttempts(0);
            post.setRedditID(null);
            postReopsitory.save(post);
        }
    }
}
```

请注意:

*   `checkAndDeleteAll()`:每 3 分钟运行一次，查看是否有帖子已经消耗了尝试次数，可以删除
*   `getPostScores()`:返回帖子的{得分，投票率，评论数}

### 4.3。修改时间表页面

我们需要将新的修改添加到我们的`schedulePostForm.html`:

```java
<input type="number" name="minUpvoteRatio"/>
<input type="checkbox" name="keepIfHasComments" value="true"/>
<input type="checkbox" name="deleteAfterLastAttempt" value="true"/>
```

## 5。电子邮件重要日志

接下来，我们将在日志回溯配置中实现一个快速但非常有用的设置—**发送重要日志(`ERROR`级别)**。这对于在应用程序生命周期的早期跟踪错误当然非常方便。

首先，我们将向我们的`pom.xml`添加一些必需的依赖项:

```java
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
</dependency>
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>mail</artifactId>
    <version>1.4.1</version>
</dependency>
```

然后，我们将为我们的`logback.xml`添加一个`SMTPAppender`:

```java
<configuration>

    <appender name="STDOUT" ...

    <appender name="EMAIL" class="ch.qos.logback.classic.net.SMTPAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>

        <smtpHost>smtp.example.com</smtpHost>
        <to>[[email protected]](/web/20220120143211/https://www.baeldung.com/cdn-cgi/l/email-protection)</to>
        <from>[[email protected]](/web/20220120143211/https://www.baeldung.com/cdn-cgi/l/email-protection)</from>
        <username>[[email protected]](/web/20220120143211/https://www.baeldung.com/cdn-cgi/l/email-protection)</username>
        <password>password</password>
        <subject>%logger{20} - %m</subject>
        <layout class="ch.qos.logback.classic.html.HTMLLayout"/>
    </appender>

    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="EMAIL" />
    </root>

</configuration>
```

这就是问题所在——现在，部署的应用程序将通过电子邮件发送发生的任何问题。

## 6。缓存子条目

事实证明，**自动完成子记录的成本很高**。每当用户在安排帖子时开始输入子编辑时，我们需要点击 Reddit API 来获取这些子编辑，并向用户显示一些建议。不理想。

不用调用 Reddit API——我们只需缓存常用的子编辑，并使用它们来自动完成。

### 6.1。检索子记录

首先，让我们检索最流行的子编辑，并将它们保存到一个普通文件中:

```java
public void getAllSubreddits() {
    JsonNode node;
    String srAfter = "";
    FileWriter writer = null;
    try {
        writer = new FileWriter("src/main/resources/subreddits.csv");
        for (int i = 0; i < 20; i++) {
            node = restTemplate.getForObject(
              "http://www.reddit.com/" + "subreddits/popular.json?limit=100&after;=" + srAfter, 
              JsonNode.class);
            srAfter = node.get("data").get("after").asText();
            node = node.get("data").get("children");
            for (JsonNode child : node) {
                writer.append(child.get("data").get("display_name").asText() + ",");
            }
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                logger.error("Error while getting subreddits", e);
            }
        }
        writer.close();
    } catch (Exception e) {
        logger.error("Error while getting subreddits", e);
    }
}
```

这是一个成熟的实现吗？不。我们还需要什么吗？不，我们没有。我们需要继续前进。

### 6.2。Subbreddit 自动完成

接下来，让我们确保**子编辑在应用程序启动**时被加载到内存中——通过让服务实现`InitializingBean`:

```java
public void afterPropertiesSet() {
    loadSubreddits();
}
private void loadSubreddits() {
    subreddits = new ArrayList<String>();
    try {
        Resource resource = new ClassPathResource("subreddits.csv");
        Scanner scanner = new Scanner(resource.getFile());
        scanner.useDelimiter(",");
        while (scanner.hasNext()) {
            subreddits.add(scanner.next());
        }
        scanner.close();
    } catch (IOException e) {
        logger.error("error while loading subreddits", e);
    }
}
```

现在子编辑数据已经全部加载到内存中，**我们可以搜索子编辑，而不用点击 Reddit API** :

```java
public List<String> searchSubreddit(String query) {
    return subreddits.stream().
      filter(sr -> sr.startsWith(query)).
      limit(9).
      collect(Collectors.toList());
}
```

公开子编辑建议的 API 当然保持不变:

```java
@RequestMapping(value = "/subredditAutoComplete")
@ResponseBody
public List<String> subredditAutoComplete(@RequestParam("term") String term) {
    return service.searchSubreddit(term);
}
```

## 7。指标

最后，我们将在应用程序中集成一些简单的指标。关于构建这类指标的更多信息，[我在这里](/web/20220120143211/https://www.baeldung.com/spring-rest-api-metrics)写了一些细节。

### 7.1。Servlet 过滤器

这里简单的`MetricFilter`:

```java
@Component
public class MetricFilter implements Filter {

    @Autowired
    private IMetricService metricService;

    @Override
    public void doFilter(
      ServletRequest request, ServletResponse response, FilterChain chain) 
      throws IOException, ServletException {
        HttpServletRequest httpRequest = ((HttpServletRequest) request);
        String req = httpRequest.getMethod() + " " + httpRequest.getRequestURI();

        chain.doFilter(request, response);

        int status = ((HttpServletResponse) response).getStatus();
        metricService.increaseCount(req, status);
    }
}
```

我们还需要将它添加到我们的`ServletInitializer`:

```java
@Override
public void onStartup(ServletContext servletContext) throws ServletException {
    super.onStartup(servletContext);
    servletContext.addListener(new SessionListener());
    registerProxyFilter(servletContext, "oauth2ClientContextFilter");
    registerProxyFilter(servletContext, "springSecurityFilterChain");
    registerProxyFilter(servletContext, "metricFilter");
}
```

### 7.2。公制服务

这是我们的`MetricService`:

```java
public interface IMetricService {
    void increaseCount(String request, int status);

    Map getFullMetric();
    Map getStatusMetric();

    Object[][] getGraphData();
}
```

### 7.3。公制控制器

她的基本控制器负责通过 HTTP 公开这些指标:

```java
@Controller
public class MetricController {

    @Autowired
    private IMetricService metricService;

    // 

    @RequestMapping(value = "/metric", method = RequestMethod.GET)
    @ResponseBody
    public Map getMetric() {
        return metricService.getFullMetric();
    }

    @RequestMapping(value = "/status-metric", method = RequestMethod.GET)
    @ResponseBody
    public Map getStatusMetric() {
        return metricService.getStatusMetric();
    }

    @RequestMapping(value = "/metric-graph-data", method = RequestMethod.GET)
    @ResponseBody
    public Object[][] getMetricGraphData() {
        Object[][] result = metricService.getGraphData();
        for (int i = 1; i < result[0].length; i++) {
            result[0][i] = result[0][i].toString();
        }
        return result;
    }
}
```

## 8。结论

这个案例研究发展得很好。该应用程序实际上是作为一个关于使用 Reddit API 进行 OAuth 的简单教程开始的；现在，它已经发展成为 Reddit 超级用户的有用工具——尤其是在日程安排和重新提交选项方面。

最后，由于我一直在使用它，看起来我自己提交给 Reddit 的内容总体上增加了很多，所以看到这一点总是很好。