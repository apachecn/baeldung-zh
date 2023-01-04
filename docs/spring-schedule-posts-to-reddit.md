# 用 Spring 安排发布到 Reddit

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-schedule-posts-to-reddit>

## 1。概述

在本案例研究的[早期](/web/20220815042026/https://www.baeldung.com/spring-security-oauth2-authentication-with-reddit "Authenticating with Reddit OAuth2 and Spring Security") [部分](/web/20220815042026/https://www.baeldung.com/spring-security-oauth-post-to-reddit "Post a Link to the Reddit API")中，我们使用 Reddit API 设置了一个简单的应用程序和 OAuth 认证流程。

现在让我们用 Reddit 构建一些有用的东西——支持**为后者安排帖子**。

## 2。`User`和帖子

首先，让我们创建两个主要实体——`User`和`Post`。`User`将跟踪用户名和一些额外的 Oauth 信息:

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false)
    private String username;

    private String accessToken;
    private String refreshToken;
    private Date tokenExpiration;

    private boolean needCaptcha;

    // standard setters and getters
}
```

下一个——`Post`实体——保存向 Reddit 提交链接所需的信息:`title`、`URL`、`subreddit`、…等等。

```java
@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false) private String title;
    @Column(nullable = false) private String subreddit;
    @Column(nullable = false) private String url;
    private boolean sendReplies;

    @Column(nullable = false) private Date submissionDate;

    private boolean isSent;

    private String submissionResponse;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
```

```java
 // standard setters and getters
}
```

## 3。持久层

我们将使用 **Spring Data JPA 来实现持久化**，所以除了众所周知的存储库接口定义之外，这里没有太多东西要看:

*   `**UserRepository:**`

```java
public interface UserRepository extends JpaRepository<User, Long> {

    User findByUsername(String username);

    User findByAccessToken(String token);
}
```

*   **T2`PostRepository:`**

```java
public interface PostRepository extends JpaRepository<Post, Long> {

    List<Post> findBySubmissionDateBeforeAndIsSent(Date date, boolean isSent);

    List<Post> findByUser(User user);
}
```

## 4。一个调度器

对于应用程序的调度方面，我们还将充分利用开箱即用的 Spring 支持。

我们正在定义一个每分钟都要运行的任务；这将简单地查找将要提交给 Reddit 的帖子:

```java
public class ScheduledTasks {
    private final Logger logger = LoggerFactory.getLogger(getClass());

    private OAuth2RestTemplate redditRestTemplate;

    @Autowired
    private PostRepository postReopsitory;

    @Scheduled(fixedRate = 1 * 60 * 1000)
    public void reportCurrentTime() {
        List<Post> posts = 
          postReopsitory.findBySubmissionDateBeforeAndIsSent(new Date(), false);
        for (Post post : posts) {
            submitPost(post);
        }
    }

    private void submitPost(Post post) {
        try {
            User user = post.getUser();
            DefaultOAuth2AccessToken token = 
              new DefaultOAuth2AccessToken(user.getAccessToken());
            token.setRefreshToken(new DefaultOAuth2RefreshToken((user.getRefreshToken())));
            token.setExpiration(user.getTokenExpiration());
            redditRestTemplate.getOAuth2ClientContext().setAccessToken(token);

            UsernamePasswordAuthenticationToken userAuthToken = 
              new UsernamePasswordAuthenticationToken(
              user.getUsername(), token.getValue(), 
              Arrays.asList(new SimpleGrantedAuthority("ROLE_USER")));
            SecurityContextHolder.getContext().setAuthentication(userAuthToken);

            MultiValueMap<String, String> param = new LinkedMultiValueMap<String, String>();
            param.add("api_type", "json");
            param.add("kind", "link");
            param.add("resubmit", "true");
            param.add("then", "comments");
            param.add("title", post.getTitle());
            param.add("sr", post.getSubreddit());
            param.add("url", post.getUrl());
            if (post.isSendReplies()) {
                param.add(RedditApiConstants.SENDREPLIES, "true");
            }

            JsonNode node = redditRestTemplate.postForObject(
              "https://oauth.reddit.com/api/submit", param, JsonNode.class);
            JsonNode errorNode = node.get("json").get("errors").get(0);
            if (errorNode == null) {
                post.setSent(true);
                post.setSubmissionResponse("Successfully sent");
                postReopsitory.save(post);
            } else {
                post.setSubmissionResponse(errorNode.toString());
                postReopsitory.save(post);
            }
        } catch (Exception e) {
            logger.error("Error occurred", e);
        }
    }
}
```

请注意，如果出现任何问题，帖子将不会被标记为`sent`——因此**下一个周期将在一分钟**后再次尝试提交。

## 5。登录流程

有了新的用户实体，保存了一些特定于安全性的信息，我们需要**修改我们简单的登录过程来存储这些信息**:

```java
@RequestMapping("/login")
public String redditLogin() {
    JsonNode node = redditRestTemplate.getForObject(
      "https://oauth.reddit.com/api/v1/me", JsonNode.class);
    loadAuthentication(node.get("name").asText(), redditRestTemplate.getAccessToken());
    return "redirect:home.html";
}
```

和`loadAuthentication()`:

```java
private void loadAuthentication(String name, OAuth2AccessToken token) {
    User user = userReopsitory.findByUsername(name);
    if (user == null) {
        user = new User();
        user.setUsername(name);
    }

    if (needsCaptcha().equalsIgnoreCase("true")) {
        user.setNeedCaptcha(true);
    } else {
        user.setNeedCaptcha(false);
    }

    user.setAccessToken(token.getValue());
    user.setRefreshToken(token.getRefreshToken().getValue());
    user.setTokenExpiration(token.getExpiration());
    userReopsitory.save(user);

    UsernamePasswordAuthenticationToken auth = 
      new UsernamePasswordAuthenticationToken(user, token.getValue(), 
      Arrays.asList(new SimpleGrantedAuthority("ROLE_USER")));
    SecurityContextHolder.getContext().setAuthentication(auth);
}
```

请注意，如果用户不存在，它是如何自动创建的。这使得“使用 Reddit 登录”过程在首次登录时在系统中创建一个本地用户。

## 6。时间表页面

接下来，让我们看看允许安排新帖子的页面:

```java
@RequestMapping("/postSchedule")
public String showSchedulePostForm(Model model) {
    boolean isCaptchaNeeded = getCurrentUser().isCaptchaNeeded();
    if (isCaptchaNeeded) {
        model.addAttribute("msg", "Sorry, You do not have enought karma");
        return "submissionResponse";
    }
    return "schedulePostForm";
}
```

```java
private User getCurrentUser() {
    return (User) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
}
```

**:**

```java
<form>
    <input name="title" />
    <input name="url" />
    <input name="subreddit" />
    <input type="checkbox" name="sendreplies" value="true"/> 
    <input name="submissionDate">
    <button type="submit" onclick="schedulePost()">Schedule</button>
</form>

<script>
function schedulePost(){
    var data = {};
    $('form').serializeArray().map(function(x){data[x.name] = x.value;});
    $.ajax({
        url: 'api/scheduledPosts',
        data: JSON.stringify(data),
        type: 'POST',
        contentType:'application/json',
        success: function(result) { window.location.href="scheduledPosts"; },
        error: function(error) { alert(error.responseText); }   
    }); 
}
</script> 
</body> 
</html>
```

请注意我们需要如何检查验证码。这是因为—**如果用户的 karma 数少于 10 个**——他们就不能在不填写验证码的情况下安排发布。

## 7 .**。过账**

当提交时间表时，**帖子信息被简单地验证并持久化**,以便稍后由调度程序获取:

```java
@RequestMapping(value = "/api/scheduledPosts", method = RequestMethod.POST)
@ResponseBody
public Post schedule(@RequestBody Post post) {
    if (submissionDate.before(new Date())) {
        throw new InvalidDateException("Scheduling Date already passed");
    }

    post.setUser(getCurrentUser());
    post.setSubmissionResponse("Not sent yet");
    return postReopsitory.save(post);
}
```

## 8。预定岗位列表

现在让我们实现一个简单的 REST API 来检索我们已有的预定帖子:

```java
@RequestMapping(value = "/api/scheduledPosts")
@ResponseBody
public List<Post> getScheduledPosts() {
    User user = getCurrentUser();
    return postReopsitory.findByUser(user);
}
```

一种简单、快捷的方式来**在前端**显示这些预定的帖子:

```java
<table>
    <thead><tr><th>Post title</th><th>Submission Date</th></tr></thead>
</table>

<script>
$(function(){
    $.get("api/scheduledPosts", function(data){
        $.each(data, function( index, post ) {
            $('.table').append('<tr><td>'+post.title+'</td><td>'+
              post.submissionDate+'</td></tr>');
        });
    });
});
</script>
```

## 9。编辑预定的帖子

接下来，让我们看看如何编辑预定的帖子。

我们将从前端开始，首先是非常简单的 MVC 操作:

```java
@RequestMapping(value = "/editPost/{id}", method = RequestMethod.GET)
public String showEditPostForm() {
    return "editPostForm";
}
```

在简单的 API 之后，这里是前端消费它:

```java
<form>
    <input type="hidden" name="id" />
    <input name="title" />
    <input name="url" />
    <input name="subreddit" />
    <input type="checkbox" name="sendReplies" value="true"/>
    <input name="submissionDate">
    <button type="submit" onclick="editPost()">Save Changes</button>
</form>

<script>
$(function() {
   loadPost();
});

function loadPost(){ 
    var arr = window.location.href.split("/"); 
    var id = arr[arr.length-1]; 
    $.get("../api/scheduledPosts/"+id, function (data){ 
        $.each(data, function(key, value) { 
            $('*[name="'+key+'"]').val(value); 
        });
    }); 
}
function editPost(){
    var id = $("#id").val();
    var data = {};
    $('form').serializeArray().map(function(x){data[x.name] = x.value;});
	$.ajax({
            url: "../api/scheduledPosts/"+id,
            data: JSON.stringify(data),
            type: 'PUT',
            contentType:'application/json'
            }).done(function() {
    	        window.location.href="../scheduledPosts";
            }).fail(function(error) {
    	        alert(error.responseText);
        }); 
}
</script>
```

现在，让我们看看 **REST API** :

```java
@RequestMapping(value = "/api/scheduledPosts/{id}", method = RequestMethod.GET) 
@ResponseBody 
public Post getPost(@PathVariable("id") Long id) { 
    return postReopsitory.findOne(id); 
}

@RequestMapping(value = "/api/scheduledPosts/{id}", method = RequestMethod.PUT) 
@ResponseStatus(HttpStatus.OK) 
public void updatePost(@RequestBody Post post, @PathVariable Long id) { 
    if (post.getSubmissionDate().before(new Date())) { 
        throw new InvalidDateException("Scheduling Date already passed"); 
    } 
    post.setUser(getCurrentUser()); 
    postReopsitory.save(post); 
}
```

## 10。取消安排/删除帖子

我们还将为任何预定的帖子提供一个简单的删除操作:

```java
@RequestMapping(value = "/api/scheduledPosts/{id}", method = RequestMethod.DELETE)
@ResponseStatus(HttpStatus.OK)
public void deletePost(@PathVariable("id") Long id) {
    postReopsitory.delete(id);
}
```

从客户端来看，我们是这样称呼它的:

```java
<a href="#" onclick="confirmDelete(${post.getId()})">Delete</a>

<script>
function confirmDelete(id) {
    if (confirm("Do you really want to delete this post?") == true) {
    	deletePost(id);
    } 
}

function deletePost(id){
	$.ajax({
	    url: 'api/scheduledPosts/'+id,
	    type: 'DELETE',
	    success: function(result) {
	    	window.location.href="scheduledPosts"
	    }
	});
}
</script>
```

## 11。结论

在 Reddit 案例研究的这一部分，我们使用 Reddit API 构建了第一个重要的功能——安排帖子。

对于一个严肃的 Reddit 用户来说，这是一个超级有用的功能，尤其是考虑到**Reddit 提交的内容是多么的时间敏感**。

接下来，我们将构建一个更有用的功能来帮助获得 Reddit 上的内容投票——机器学习建议。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20220815042026/https://github.com/Baeldung/reddit-app "The Full Spring / Reddit Example Project on Github")中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。