# 发布 Reddit API 的链接

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth-post-to-reddit>

 ![](img/d963f5ec14d8688c1dc030e6537045f3.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220521224433/https://www.baeldung.com/lightrun-n-security)

## 1。概述

在[系列](/web/20220521224433/https://www.baeldung.com/case-study-a-reddit-app-with-spring)的第二篇文章中，我们将构建一些简单的功能，通过他们的 API 从我们的应用程序发布到 Reddit 上。

## 2。必要的安全保障

首先，让我们先解决安全方面的问题。

为了**提交一个到 Reddit** 的链接，我们需要用`submit`的`scope`定义一个 OAuth 保护资源:

```java
@Bean
public OAuth2ProtectedResourceDetails reddit() {
    AuthorizationCodeResourceDetails details = new AuthorizationCodeResourceDetails();
    details.setId("reddit");
    details.setClientId(clientID);
    details.setClientSecret(clientSecret);
    details.setAccessTokenUri(accessTokenUri);
    details.setUserAuthorizationUri(userAuthorizationUri);
    details.setTokenName("oauth_token");
    details.setScope(Arrays.asList("identity", "submit"));
    details.setGrantType("authorization_code");
    return details;
}
```

注意，我们还指定了`scope``identity`，因为我们还需要访问用户帐户信息。

## 3。需要验证码吗？

Reddit **的新用户必须填写验证码**才能提交；这是在他们通过 Reddit 中的某个业力门槛之前。

对于这些用户，我们首先需要检查是否需要验证码:

```java
private String needsCaptcha() {
    String result = redditRestTemplate.getForObject(
      "https://oauth.reddit.com/api/needs_captcha.json", String.class);
    return result;
}

private String getNewCaptcha() {
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);
    HttpEntity req = new HttpEntity(headers);

    Map<String, String> param = new HashMap<String, String>();
    param.put("api_type", "json");

    ResponseEntity<String> result = redditRestTemplate.postForEntity(
      "https://oauth.reddit.com/api/new_captcha", req, String.class, param);
    String[] split = result.getBody().split("""); 
    return split[split.length - 2];
}
```

## 4。`Submit Post`的形式

接下来，让我们创建向 Reddit 提交新帖子的主表单。提交链接需要以下详细信息:

*   `**title**`–文章的标题
*   `**url**`–文章的网址
*   `**subreddit**`–提交链接的子 reddit

让我们看看如何显示这个简单的提交页面:

```java
@RequestMapping("/post")
public String showSubmissionForm(Model model) throws JsonProcessingException, IOException {
    String needsCaptchaResult = needsCaptcha();
    if (needsCaptchaResult.equalsIgnoreCase("true")) {
        String iden = getNewCaptcha();
        model.addAttribute("iden", iden);
    }
    return "submissionForm";
}
```

当然还有最基本的`submissionForm.html`:

```java
<form>
    <input name="title"/>
    <input name="url" />
    <input name="sr"/>
    <input  type="checkbox" name="sendReplies" value="true"/>

    <div th:if="${iden != null}">
        <input type="hidden" name="iden" value="${iden}"/>
        <input name="captcha"/>
        <img src="http://www.reddit.com/captcha/${iden}" alt="captcha" width="200"/>
    </div>
    <button type="submit" onclick="submitPost()">Post</button>
</form>

<script>
function submitPost(){
    var data = {};
    $('form').serializeArray().map(function(x){data[x.name] = x.value;});
    $.ajax({
        url: "api/posts",
        data: JSON.stringify(data),
        type: 'POST',
        contentType:'application/json'
    }).done(function(data) {
        if(data.length < 2){ alert(data[0]);}
        else{
            window.location.href="submissionResponse?msg="+
              data[0]+"&url;="+data[1];
        }
    }).fail(function(error) { alert(error.responseText); }); 
}
</script>
```

## 5。提交一个链接到 Reddit

现在，让我们看看最后一步——通过 Reddit API 提交实际链接。

我们将使用来自`submissionForm`的参数向 Reddit 提交一个提交请求:

```java
@Controller
@RequestMapping(value = "/api/posts")
public class RedditPostRestController {

    @Autowired
    private RedditService service;

    @RequestMapping(method = RequestMethod.POST)
    @ResponseBody
    public List<String> submit(@Valid @RequestBody PostDto postDto) {
        return service.submitPost(postDto);
    }
}
```

下面是实际的方法实现:

```java
public List<String> submitPost(PostDto postDto) {
    MultiValueMap<String, String> param1 = constructParams(postDto);
    JsonNode node = redditTemplate.submitPost(param1);
    return parseResponse(node);
}

private MultiValueMap<String, String> constructParams(PostDto postDto) {
    MultiValueMap<String, String> param = new LinkedMultiValueMap<String, String>();
    param.add("title", postDto.getTitle());
    param.add("sr", postDto.getSubreddit());
    param.add("url", postDto.getUrl());
    param.add("iden", postDto.getIden());
    param.add("captcha", postDto.getCaptcha());
    if (postDto.isSendReplies()) {
        param.add("sendReplies", "true");
    }

    param.add("api_type", "json");
    param.add("kind", "link");
    param.add("resubmit", "true");
    param.add("then", "comments");
    return param;
}
```

以及简单的解析逻辑，**处理来自 Reddit API 的响应**:

```java
private List<String> parseResponse(JsonNode node) {
    String result = "";
    JsonNode errorNode = node.get("json").get("errors").get(0);
    if (errorNode != null) {
        for (JsonNode child : errorNode) {
            result = result + child.toString().replaceAll("\"|null", "") + "<br>";
        }
        return Arrays.asList(result);
    } else {
        if ((node.get("json").get("data") != null) && 
            (node.get("json").get("data").get("url") != null)) {
            return Arrays.asList("Post submitted successfully", 
              node.get("json").get("data").get("url").asText());
        } else {
            return Arrays.asList("Error Occurred while parsing Response");
        }
    }
}
```

所有这一切都与**一个基本的 DTO** 一起工作:

```java
public class PostDto {
    @NotNull
    private String title;

    @NotNull
    private String url;

    @NotNull
    private String subreddit;

    private boolean sendReplies;

    private String iden;
    private String captcha;
}
```

最后——第`submissionResponse.html`:

```java
<html>
<body>
    <h1 th:text="${msg}">Hello</h1>
    <h1 th:if="${param.containsKey('msg')}" th:text="${param.msg[0]}">Hello</h1>
    <h2 th:if="${param.containsKey('url')}"><a th:href="${param.url[0]}">Here</a></h2>
</body>
</html>
```

## 6。结论

在这个快速教程中，我们实现了一些基本的`Submit to Reddit` 功能——简单但功能齐全。

在本案例研究的下一部分，我们将在我们的应用中实现一个`Schedule Post for Later` 功能。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20220521224433/https://github.com/Baeldung/reddit-app "The Full Spring / Reddit Example Project on Github")中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。