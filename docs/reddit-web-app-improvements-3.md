# Reddit 应用程序的第三轮改进

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-web-app-improvements-3>

## 1。概述

在本文中，我们将通过对现有功能进行小而有用的改进，继续推进我们的小案例研究应用程序。

## 2。更好的桌子

让我们从使用 jQuery DataTables 插件替换应用程序之前使用的旧的基本表开始。

### 2.1。发布存储库和服务

首先，我们将添加一个方法来**统计用户**的预定帖子——当然是利用 Spring 数据语法:

```java
public interface PostRepository extends JpaRepository<Post, Long> {
    ...
    Long countByUser(User user);
}
```

接下来，让我们快速看一下**服务层实现**——基于分页参数检索用户的帖子:

```java
@Override
public List<SimplePostDto> getPostsList(int page, int size, String sortDir, String sort) {
    PageRequest pageReq = new PageRequest(page, size, Sort.Direction.fromString(sortDir), sort);
    Page<Post> posts = postRepository.findByUser(userService.getCurrentUser(), pageReq);
    return constructDataAccordingToUserTimezone(posts.getContent());
}
```

我们正在根据用户的时区**转换日期:**

```java
private List<SimplePostDto> constructDataAccordingToUserTimezone(List<Post> posts) {
    String timeZone = userService.getCurrentUser().getPreference().getTimezone();
    return posts.stream().map(post -> new SimplePostDto(
      post, convertToUserTomeZone(post.getSubmissionDate(), timeZone)))
      .collect(Collectors.toList());
}
private String convertToUserTomeZone(Date date, String timeZone) { 
    dateFormat.setTimeZone(TimeZone.getTimeZone(timeZone)); 
    return dateFormat.format(date); 
}
```

### 2.2。带分页的 API

接下来，我们将通过 API 发布带有完整分页和排序的操作:

```java
@RequestMapping(method = RequestMethod.GET)
@ResponseBody
public List<SimplePost> getScheduledPosts(
  @RequestParam(value = "page", required = false, defaultValue = "0") int page, 
  @RequestParam(value = "size", required = false, defaultValue = "10") int size,
  @RequestParam(value = "sortDir", required = false, defaultValue = "asc") String sortDir, 
  @RequestParam(value = "sort", required = false, defaultValue = "title") String sort, 
  HttpServletResponse response) {
    response.addHeader("PAGING_INFO", 
      scheduledPostService.generatePagingInfo(page, size).toString());
    return scheduledPostService.getPostsList(page, size, sortDir, sort);
}
```

请注意我们是如何使用自定义头将分页信息传递给客户端的。还有其他稍微标准一点的方法来做这件事，这些方法我们稍后会探讨。

然而，这种实现非常简单——我们有一个简单的方法来生成寻呼信息:

```java
public PagingInfo generatePagingInfo(int page, int size) {
    long total = postRepository.countByUser(userService.getCurrentUser());
    return new PagingInfo(page, size, total);
}
```

而`PagingInfo`本身:

```java
public class PagingInfo {
    private long totalNoRecords;
    private int totalNoPages;
    private String uriToNextPage;
    private String uriToPrevPage;

    public PagingInfo(int page, int size, long totalNoRecords) {
        this.totalNoRecords = totalNoRecords;
        this.totalNoPages = Math.round(totalNoRecords / size);
        if (page > 0) {
            this.uriToPrevPage = "page=" + (page - 1) + "&size;=" + size;
        }
        if (page < this.totalNoPages) {
            this.uriToNextPage = "page=" + (page + 1) + "&size;=" + size;
        }
    }
}
```

### 2.3。前端

最后，简单的前端将使用一个定制的 JS 方法与 API 进行交互，并处理 [jQuery 数据表参数](https://web.archive.org/web/20220815042615/https://www.datatables.net/manual/server-side):

```java
<table>
<thead><tr>
<th>Post title</th><th>Submission Date</th><th>Status</th>
<th>Resubmit Attempts left</th><th>Actions</th>
</tr></thead>   
</table>

<script>
$(document).ready(function() {
    $('table').dataTable( {
        "processing": true,
        "searching":false,
        "columnDefs": [
            { "name": "title", "targets": 0 },
            { "name": "submissionDate", "targets": 1 },
            { "name": "submissionResponse", "targets": 2 },
            { "name": "noOfAttempts", "targets": 3 } ],
        "columns": [
            { "data": "title" },
            { "data": "submissionDate" },
            { "data": "submissionResponse" },
            { "data": "noOfAttempts" }],
        "serverSide": true,
        "ajax": function(data, callback, settings) {
            $.get('api/scheduledPosts', {
              size: data.length,
              page: (data.start/data.length),
              sortDir: data.order[0].dir,
              sort: data.columns[data.order[0].column].name
              }, function(res,textStatus, request) {
                var pagingInfo = request.getResponseHeader('PAGING_INFO');
                var total = pagingInfo.split(",")[0].split("=")[1];
                callback({recordsTotal: total, recordsFiltered: total,data: res});
              });
          }
    } );
} );
</script>
```

### 2.4。用于分页的 API 测试

有了现在发布的 API，我们可以编写一些简单的 API 测试来确保分页机制的基本功能按预期工作:

```java
@Test
public void givenMoreThanOnePage_whenGettingUserScheduledPosts_thenNextPageExist() 
  throws ParseException, IOException {
    createPost();
    createPost();
    createPost();

    Response response = givenAuth().
      params("page", 0, "size", 2).get(urlPrefix + "/api/scheduledPosts");

    assertEquals(200, response.statusCode());
    assertTrue(response.as(List.class).size() > 0);

    String pagingInfo = response.getHeader("PAGING_INFO");
    long totalNoRecords = Long.parseLong(pagingInfo.split(",")[0].split("=")[1]);
    String uriToNextPage = pagingInfo.split(",")[2].replace("uriToNextPage=", "").trim();

    assertTrue(totalNoRecords > 2);
    assertEquals(uriToNextPage, "page=1&size;=2");
}

@Test
public void givenMoreThanOnePage_whenGettingUserScheduledPostsForSecondPage_thenCorrect() 
  throws ParseException, IOException {
    createPost();
    createPost();
    createPost();

    Response response = givenAuth().
      params("page", 1, "size", 2).get(urlPrefix + "/api/scheduledPosts");

    assertEquals(200, response.statusCode());
    assertTrue(response.as(List.class).size() > 0);

    String pagingInfo = response.getHeader("PAGING_INFO");
    long totalNoRecords = Long.parseLong(pagingInfo.split(",")[0].split("=")[1]);
    String uriToPrevPage = pagingInfo.split(",")[3].replace("uriToPrevPage=", "").trim();

    assertTrue(totalNoRecords > 2);
    assertEquals(uriToPrevPage, "page=0&size;=2");
} 
```

## 3。电子邮件通知

接下来，我们将构建一个基本的电子邮件通知流—**,当用户的预定帖子被发送时，用户会收到电子邮件**:

### 3.1。电子邮件配置

首先，让我们进行电子邮件配置:

```java
@Bean
public JavaMailSenderImpl javaMailSenderImpl() {
    JavaMailSenderImpl mailSenderImpl = new JavaMailSenderImpl();
    mailSenderImpl.setHost(env.getProperty("smtp.host"));
    mailSenderImpl.setPort(env.getProperty("smtp.port", Integer.class));
    mailSenderImpl.setProtocol(env.getProperty("smtp.protocol"));
    mailSenderImpl.setUsername(env.getProperty("smtp.username"));
    mailSenderImpl.setPassword(env.getProperty("smtp.password"));
    Properties javaMailProps = new Properties();
    javaMailProps.put("mail.smtp.auth", true);
    javaMailProps.put("mail.smtp.starttls.enable", true);
    mailSenderImpl.setJavaMailProperties(javaMailProps);
    return mailSenderImpl;
}
```

以及使 SMTP 工作的必要属性:

```java
smtp.host=email-smtp.us-east-1.amazonaws.com
smtp.port=465
smtp.protocol=smtps
smtp.username=example
smtp.password=
[[email protected]](/web/20220815042615/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

### 3.2。发布帖子时触发事件

现在，让我们确保在预定的帖子成功发布到 Reddit 时触发一个事件:

```java
private void updatePostFromResponse(JsonNode node, Post post) {
    JsonNode errorNode = node.get("json").get("errors").get(0);
    if (errorNode == null) {
        ...
        String email = post.getUser().getPreference().getEmail();
        eventPublisher.publishEvent(new OnPostSubmittedEvent(post, email));
    } 
    ...
}
```

### 3.3。事件和监听器

事件实现非常简单:

```java
public class OnPostSubmittedEvent extends ApplicationEvent {
    private Post post;
    private String email;

    public OnPostSubmittedEvent(Post post, String email) {
        super(post);
        this.post = post;
        this.email = email;
    }
}
```

而听者:

```java
@Component
public class SubmissionListner implements ApplicationListener<OnPostSubmittedEvent> {
    @Autowired
    private JavaMailSender mailSender;

    @Autowired
    private Environment env;

    @Override
    public void onApplicationEvent(OnPostSubmittedEvent event) {
        SimpleMailMessage email = constructEmailMessage(event);
        mailSender.send(email);
    }

    private SimpleMailMessage constructEmailMessage(OnPostSubmittedEvent event) {
        String recipientAddress = event.getEmail();
        String subject = "Your scheduled post submitted";
        SimpleMailMessage email = new SimpleMailMessage();
        email.setTo(recipientAddress);
        email.setSubject(subject);
        email.setText(constructMailContent(event.getPost()));
        email.setFrom(env.getProperty("support.email"));
        return email;
    }

    private String constructMailContent(Post post) {
        return "Your post " + post.getTitle() + " is submitted.\n" +
          "http://www.reddit.com/r/" + post.getSubreddit() + 
          "/comments/" + post.getRedditID();
    }
}
```

## 4。使用帖子总票数

接下来，我们将做一些工作来简化重新提交选项——而不是处理 upvote 比率(这很难理解)—**它现在处理总票数**。

我们可以使用帖子得分和投票率来计算投票总数:

*   得分=赞成票–反对票
*   总票数=赞成票+反对票
*   赞成票比率=赞成票/总票数

所以:

`Total number of votes = Math.round( score / ((2 * upvote ratio) – 1) )`

首先，我们将修改分数逻辑来计算并跟踪投票总数:

```java
public PostScores getPostScores(Post post) {
    ...

    float ratio = node.get("upvote_ratio").floatValue();
    postScore.setTotalVotes(Math.round(postScore.getScore() / ((2 * ratio) - 1)));

    ...
}
```

当然，我们将在**检查一个帖子是否被认为失败时使用它**:

```java
private boolean didPostGoalFail(Post post) {
    PostScores postScores = getPostScores(post);
    int totalVotes = postScores.getTotalVotes();
    ...
    return (((score < post.getMinScoreRequired()) || 
            (totalVotes < post.getMinTotalVotes())) && 
            !((noOfComments > 0) && post.isKeepIfHasComments()));
}
```

最后，我们当然会删除旧的`ratio`字段。

## 5。验证重新提交选项

最后，我们将通过向复杂的重新提交选项添加一些验证来帮助用户:

### 5.1。`ScheduledPost`服务

下面是简单的`checkIfValidResubmitOptions()`方法:

```java
private boolean checkIfValidResubmitOptions(Post post) {
    if (checkIfAllNonZero(
          post.getNoOfAttempts(), 
          post.getTimeInterval(), 
          post.getMinScoreRequired())) {
        return true;
    } else {
        return false;
    }
}
private boolean checkIfAllNonZero(int... args) {
    for (int tmp : args) {
       if (tmp == 0) {
           return false;
        }
    }
    return true;
}
```

在安排新帖子时，我们将充分利用这一验证:

```java
public Post schedulePost(boolean isSuperUser, Post post, boolean resubmitOptionsActivated) 
  throws ParseException {
    if (resubmitOptionsActivated && !checkIfValidResubmitOptions(post)) {
        throw new InvalidResubmitOptionsException("Invalid Resubmit Options");
    }
    ...        
}
```

请注意，如果重新提交逻辑打开，则以下字段需要有非零值:

*   尝试次数
*   时间间隔
*   最低分数要求

### 5.2。异常处理

最后——在输入无效的情况下，在我们的主要错误处理逻辑中处理`InvalidResubmitOptionsException`:

```java
@ExceptionHandler({ InvalidResubmitOptionsException.class })
public ResponseEntity<Object> handleInvalidResubmitOptions
  (RuntimeException ex, WebRequest request) {

    logger.error("400 Status Code", ex);
    String bodyOfResponse = ex.getLocalizedMessage();
    return new ResponseEntity<Object>(
      bodyOfResponse, new HttpHeaders(), HttpStatus.BAD_REQUEST);
}
```

### 5.3。测试重新提交选项

最后，让我们现在测试我们的重新提交选项–我们将测试激活和停用条件:

```java
public class ResubmitOptionsLiveTest extends AbstractLiveTest {
    private static final String date = "2016-01-01 00:00";

    @Test
    public void 
      givenResubmitOptionsDeactivated_whenSchedulingANewPost_thenCreated() 
      throws ParseException, IOException {
        Post post = createPost();

        Response response = withRequestBody(givenAuth(), post)
          .queryParams("resubmitOptionsActivated", false)
          .post(urlPrefix + "/api/scheduledPosts");

        assertEquals(201, response.statusCode());
        Post result = objectMapper.reader().forType(Post.class).readValue(response.asString());
        assertEquals(result.getUrl(), post.getUrl());
    }

    @Test
    public void 
      givenResubmitOptionsActivated_whenSchedulingANewPostWithZeroAttempts_thenInvalid() 
      throws ParseException, IOException {
        Post post = createPost();
        post.setNoOfAttempts(0);
        post.setMinScoreRequired(5);
        post.setTimeInterval(60);

        Response response = withRequestBody(givenAuth(), post)
          .queryParams("resubmitOptionsActivated", true)
          .post(urlPrefix + "/api/scheduledPosts");

        assertEquals(400, response.statusCode());
        assertTrue(response.asString().contains("Invalid Resubmit Options"));
    }

    @Test
    public void 
      givenResubmitOptionsActivated_whenSchedulingANewPostWithZeroMinScore_thenInvalid() 
      throws ParseException, IOException {
        Post post = createPost();
        post.setMinScoreRequired(0);
        post.setNoOfAttempts(3);
        post.setTimeInterval(60);

        Response response = withRequestBody(givenAuth(), post)
          .queryParams"resubmitOptionsActivated", true)
          .post(urlPrefix + "/api/scheduledPosts");

        assertEquals(400, response.statusCode());
        assertTrue(response.asString().contains("Invalid Resubmit Options"));
    }

    @Test
    public void 
      givenResubmitOptionsActivated_whenSchedulingANewPostWithZeroTimeInterval_thenInvalid() 
      throws ParseException, IOException {
        Post post = createPost();
        post.setTimeInterval(0);
        post.setMinScoreRequired(5);
        post.setNoOfAttempts(3);

        Response response = withRequestBody(givenAuth(), post)
          .queryParams("resubmitOptionsActivated", true)
          .post(urlPrefix + "/api/scheduledPosts");

        assertEquals(400, response.statusCode());
        assertTrue(response.asString().contains("Invalid Resubmit Options"));
    }

    @Test
    public void 
      givenResubmitOptionsActivated_whenSchedulingNewPostWithValidResubmitOptions_thenCreated() 
      throws ParseException, IOException {
        Post post = createPost();
        post.setMinScoreRequired(5);
        post.setNoOfAttempts(3);
        post.setTimeInterval(60);

        Response response = withRequestBody(givenAuth(), post)
          .queryParams("resubmitOptionsActivated", true)
          .post(urlPrefix + "/api/scheduledPosts");

        assertEquals(201, response.statusCode());
        Post result = objectMapper.reader().forType(Post.class).readValue(response.asString());
        assertEquals(result.getUrl(), post.getUrl());
    }

    private Post createPost() throws ParseException {
        Post post = new Post();
        post.setTitle(randomAlphabetic(6));
        post.setUrl("test.com");
        post.setSubreddit(randomAlphabetic(6));
        post.setSubmissionDate(dateFormat.parse(date));
        return post;
    }
}
```

## 6。结论

在这一期中，我们做了几项改进，即**将案例研究应用程序推向正确的方向**——易用性。

Reddit 日程安排应用程序的整个想法是允许用户通过进入应用程序，完成工作，然后离开，快速地为 Reddit 安排新文章。

它正在到达那里。