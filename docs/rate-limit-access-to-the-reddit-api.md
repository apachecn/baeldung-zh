# 限制访问 Reddit API 的速率

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rate-limit-access-to-the-reddit-api>

## 1。概述

在这篇简短的文章中，我们将继续通过**限制访问实时 Reddit API** 的方式来改进[我们的小型 Reddit 应用](/web/20220117213845/https://www.baeldung.com/case-study-a-reddit-app-with-spring)。

简单的想法是，我们希望确保**我们不会过多地访问他们的 API**——否则 Reddit 会开始阻止这些请求。我们要好好利用芭乐到达那里。

## 2。`RedditTemplate` 一种风俗

首先，让我们创建一个 Reddit 模板—**一个 Reddit API** 的小客户端——它将所有底层通信整合到一个组件中:

```java
@Component
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RedditTemplate {

    @Autowired
    @Qualifier("redditRestTemplate")
    private OAuth2RestTemplate redditRestTemplate;

    private RateLimiter rateLimiter;

    public RedditTemplate() {
        rateLimiter = RateLimiter.create(1);
    }

    public JsonNode getUserInfo() {
        rateLimiter.acquire();
        return redditRestTemplate.getForObject(
          "https://oauth.reddit.com/api/v1/me", JsonNode.class);
    }

    public JsonNode submitPost(MultiValueMap<String, String> params) {
        rateLimiter.acquire();
        return redditRestTemplate.postForObject(
          "https://oauth.reddit.com/api/submit", params, JsonNode.class);
    }

    public String needsCaptcha() {
        rateLimiter.acquire();
        return redditRestTemplate.getForObject(
          "https://oauth.reddit.com/api/needs_captcha.json", String.class);
    }

    public String getNewCaptcha() {
        rateLimiter.acquire();
        Map<String, String> param = new HashMap<String, String>();
        param.put("api_type", "json");
        return redditRestTemplate.postForObject(
          "https://oauth.reddit.com/api/new_captcha", param, String.class, param);
    }

    public OAuth2AccessToken getAccessToken() {
        rateLimiter.acquire();
        return redditRestTemplate.getAccessToken();
    }
}
```

这里发生了一些有趣的事情。

首先——**我们对这个 bean** 使用了`Session`范围——这样我们应用程序中的每个用户/会话都将获得自己的`RedditTemplate`实例。

现在—`OAuth2RestTemplate`已经支持保持凭证会话的作用域，但是我们在这里超越了这一点，使实际的 bean 实例会话作用域——这样我们还可以**分别限制每个用户的速率**。

这将我们引向实际的速率限制逻辑——简单地说，我们在让请求通过并点击活动 API 之前，使用番石榴`RateLimiter`来获取许可**。**

## 3。`RedditController`

接下来，让我们开始在`RedditContoller`中使用这个新的`RedditTemplate`，例如:

```java
@Controller
public class RedditController {
    @Autowired
    private RedditTemplate redditTemplate;

    @Autowired
    private UserRepository userReopsitory;

    @RequestMapping("/login")
    public String redditLogin() {
        JsonNode node = redditTemplate.getUserInfo();

        loadAuthentication(node.get("name").asText(), 
          redditTemplate.getAccessToken());
        return "redirect:home.html";
    }
}
```

## 4。结论

在案例研究的这一部分，我们为 Reddit 应用程序添加了速率限制，以确保我们不会因为太多活动而被实时 API 阻止。

这也不是一个理论上的问题——而是我在使用这款应用时遇到的一个问题。

正是这种小的改进最终会导致一个成熟和可用的应用程序，所以我对这一特殊的步骤感到兴奋。