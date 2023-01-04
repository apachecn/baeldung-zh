# 重试向 Reddit 提交没有足够吸引力的帖子

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-retry-to-submit-inactive-post>

## 1。概述

在 Reddit 上发帖毫无意义。一个帖子可能做得很好，得到很多关注，而另一个可能更好的帖子可能得不到任何喜爱。如何在早期关注这些帖子——如果它们没有获得足够的关注—**快速删除它们并重新提交它们**。

在这篇简短的文章中，我们通过实现一个有趣的功能**来继续[Reddit 案例研究](/web/20211127053914/https://www.baeldung.com/case-study-a-reddit-app-with-spring)，删除并重新提交一篇没有立即引起足够关注的帖子**。

简单的目标是允许用户配置 Reddit 上的投票数足以认为帖子在特定时间间隔内获得了足够的关注度。

## 2。更多 Reddit 权限

首先——我们需要**从 Reddit API 请求额外的权限**——特别是我们需要编辑帖子。

因此，我们将在 Reddit `Resource`中添加`edit`范围:

```
@Bean
public OAuth2ProtectedResourceDetails reddit() {
    AuthorizationCodeResourceDetails details = 
      new AuthorizationCodeResourceDetails();
    details.setScope(Arrays.asList("identity", "read", "submit", "edit"));
    ...
}
```

## 3。实体和存储库

现在，让我们在我们的`Post`实体中添加额外的信息:

```
@Entity
public class Post {
    ...
    private String redditID;
    private int noOfAttempts;
    private int timeInterval;
    private int minScoreRequired;
}
```

字段:

*   **`redditID`**:Reddit 上的帖子 ID，在查看分数和删除帖子时使用
*   **`noOfAttempts`** :重新提交尝试的最大次数(删除帖子并重新提交)
*   **`timeInterval`** :检查帖子是否获得足够牵引力的时间间隔
*   **`minScoreRequired`** :认为成功到可以放弃的最低分数

接下来，让我们在我们的`PostRepository`界面中添加一些新的操作，以便在我们需要检查帖子时方便地检索它们:

```
public interface PostRepository extends JpaRepository<Post, Long> {

    List<Post> findBySubmissionDateBeforeAndIsSent(Date date, boolean sent);

    List<Post> findByUser(User user);

    List<Post> findByRedditIDNotNullAndNoOfAttemptsGreaterThan(int attempts);
}
```

## 4。新的预定任务

现在，让我们在调度程序中定义一个新任务，即重新提交任务:

```
@Scheduled(fixedRate = 3 * 60 * 1000)
public void checkAndReSubmitPosts() {
    List<Post> submitted = 
      postReopsitory.findByRedditIDNotNullAndNoOfAttemptsGreaterThan(0);
    for (Post post : submitted) {
        checkAndReSubmit(post);
    }
}
```

每隔几分钟，这只是简单地重复仍在播放的帖子，如果它们没有得到足够的关注，它会删除它们并再次提交它们。

这里是`checkAndReSubmit()`方法:

```
private void checkAndReSubmit(Post post) {
    try {
        checkAndReSubmitInternal(post);
    } catch (final Exception e) {
        logger.error("Error occurred while check post " + post.toString(), e);
    }
}
private void checkAndReSubmitInternal(Post post) {
    if (didIntervalPassed(post.getSubmissionDate(), post.getTimeInterval())) {
        int score = getPostScore(post.getRedditID());
        if (score < post.getMinScoreRequired()) {
            deletePost(post.getRedditID());
            resetPost(post);
        } else {
            post.setNoOfAttempts(0);
            postReopsitory.save(post);
        }
    }
}
private boolean didIntervalPassed(Date submissonDate, int postInterval) {
    long currentTime = new Date().getTime();
    long interval = currentTime - submissonDate.getTime();
    long intervalInMinutes = TimeUnit.MINUTES.convert(interval, TimeUnit.MILLISECONDS);
    return intervalInMinutes > postInterval;
}
private void resetPost(Post post) {
    long time = new Date().getTime();
    time += TimeUnit.MILLISECONDS.convert(post.getTimeInterval(), TimeUnit.MINUTES);
    post.setRedditID(null);
    post.setSubmissionDate(new Date(time));
    post.setSent(false);
    post.setSubmissionResponse("Not sent yet");
    postReopsitory.save(post);
}
```

我们还需要在第一次提交给 Reddit 时跟踪`redditID`:

```
private void submitPostInternal(Post post) {
    ...
    JsonNode node = redditRestTemplate.postForObject(
      "https://oauth.reddit.com/api/submit", param, JsonNode.class);
    JsonNode errorNode = node.get("json").get("errors").get(0);
    if (errorNode == null) {
        post.setRedditID(node.get("json").get("data").get("id").asText());
        post.setNoOfAttempts(post.getNoOfAttempts() - 1);
        ...
}
```

这里的逻辑也很简单——我们只需保存 id 并减少尝试次数计数器。

## 5。获取 Reddit 帖子分数

现在，让我们看看如何从 Reddit 获得这篇文章的当前得分:

```
private int getPostScore(String redditId) {
    JsonNode node = redditRestTemplate.getForObject(
      "https://oauth.reddit.com/api/info?id=t3_" + redditId, JsonNode.class);
    int score = node.get("data").get("children").get(0).get("data").get("score").asInt();
    return score;
}
```

请注意:

*   当从 Reddit 检索文章信息时，我们需要“`read`”范围
*   我们向 reddit id 添加“`t3_`”，以获得帖子的全名

## 6。删除 Reddit 帖子

接下来，让我们看看如何使用 id 删除 Reddit 帖子:

```
private void deletePost(String redditId) {
    MultiValueMap<String, String> param = new LinkedMultiValueMap<String, String>();
    param.add("id", "t3_" + redditId);
    redditRestTemplate.postForObject(
      "https://oauth.reddit.com/api/del.json", param, JsonNode.class);
}
```

## 7。`RedditController`

现在，让我们将新信息添加到控制器中:

```
@RequestMapping(value = "/schedule", method = RequestMethod.POST)
public String schedule(Model model, 
  @RequestParam Map<String, String> formParams) throws ParseException {
    Post post = new Post();
    post.setTitle(formParams.get("title"));
    post.setSubreddit(formParams.get("sr"));
    post.setUrl(formParams.get("url"));
    post.setNoOfAttempts(Integer.parseInt(formParams.get("attempt")));
    post.setTimeInterval(Integer.parseInt(formParams.get("interval")));
    post.setMinScoreRequired(Integer.parseInt(formParams.get("score")));
    ....
}
```

## 8。UI–配置规则

最后，让我们修改非常简单的时间表，添加并重新提交新的设置:

```
<label class="col-sm-3">Resubmit Settings</label>

<label>Number of Attempts</label> 
<select name="attempt">
    <option value="0" selected>None</option>
    <option value="2">2</option>
    <option value="3">3</option>
    <option value="4">4</option>
    <option value="5">5</option>
</select>

<label>Time interval</label>
<select name="interval">
    <option value="0" selected >None</option>
    <option value="45">45 minutes</option>
    <option value="60">1 hour</option>
    <option value="120">2 hours</option>
</select>

<label>Min score</label>
<input type="number"value="0" name="score" required/>
```

## 9。结论

我们正在继续改进这个简单应用程序的功能——我们现在可以向 Reddit 发帖——如果帖子没有很快获得足够的关注——我们可以让系统删除它并重新发帖，让它有更好的表现机会。