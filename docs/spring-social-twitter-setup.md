# Spring 社交 Twitter 设置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-social-twitter-setup>

本系列的第一部分处理[使用 StackExchange REST API](/web/20230103152303/https://www.baeldung.com/tweeting-stackexchange-with-spring-social-part-1 "Tweeting StackExchange with Spring Social") 的初始工作，以便检索它的顶级问题。这个**的第二部分**将关注于使用 Spring Social Twitter 项目建立与 Twitter REST APIs 交互所必需的支持。最终目标是能够在推特上发布这些问题，每天两个，在几个账号上，每个关注一个话题。

## 1。使用 Spring 社交推特

使用 Spring Social Twitter 项目所需的依赖关系非常简单。首先，我们定义`spring-social-twitter`本身:

```java
<dependency>
   <groupId>org.springframework.social</groupId>
   <artifactId>spring-social-twitter</artifactId>
   <version>1.1.0.RELEASE</version>
</dependency>
```

然后，我们需要用更新的版本覆盖它的一些依赖项:

```java
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-core</artifactId>
   <version>4.1.0.RELEASE</version>
</dependency>
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-web</artifactId>
   <version>4.1.0.RELEASE</version>
</dependency>
<dependency>
   <groupId>org.codehaus.jackson</groupId>
   <artifactId>jackson-mapper-asl</artifactId>
   <version>1.9.13</version>
</dependency>
```

`spring-core`和`spring-web`都被`spring-social-twitter`定义为依赖关系，但分别与**的旧版本**–`3.0.7.RELEASE`和`3.1.0.RELEASE`相关。在我们自己的 pom 中覆盖这些可以确保项目使用我们定义的最新版本，而不是这些旧的继承版本。

## 2。创建 Twitter 应用程序

这个用例是一个简单的例子——在一个**个人账户**上发推文，而不是代表他们账户上的其他用户。如果应用程序需要在多个用户的每个 twitter 账户上为多个用户发布 tweet，那么简单的事实允许我们免除大部分必要的 [OAuth 编排](https://web.archive.org/web/20230103152303/https://stackoverflow.com/questions/7968641/spring-social-twitter-oauth/7970911#7970911 "Spring Social Twitter Oauth")。

因此，对于我们的用例，我们将**直接创建`TwitterTemplate`**，因为我们可以手动设置我们需要这样做的一切。

我们需要的第一件事是一个**开发应用程序**——登录后，可以在这里创建一个[。创建应用程序后，我们将拥有一个**消费者密钥**和**消费者秘密**——它们是从应用程序的页面上获得的——在`Details`选项卡上，在`OAuth settings`下。](https://web.archive.org/web/20230103152303/https://dev.twitter.com/build/apis "Create Twitter App")

此外，为了允许应用程序在帐户上发布 tweet，必须将 **`Read and Write`访问权限**设置为仅替换默认的 **`Read`** 权限。

## 3。`TwitterTemplate`拨备一

接下来，`TwitterTemplate`需要提供一个**访问令牌**和一个**访问令牌秘密**。这些也可以从应用程序页面生成——在`Details`选项卡下—**创建我的访问令牌**。然后可以从`OAuth tool`选项卡下检索访问令牌和密码。

通过`Recreate my access token`动作，新的总是可以在`Details`选项卡上重新生成。

此时，我们已经拥有了所需的一切——消费者密钥和消费者机密，以及访问令牌和访问令牌机密——这意味着我们可以继续为该应用创建我们的`TwitterTemplate` :

```java
new TwitterTemplate(consumerKey, consumerSecret, accessToken, accessTokenSecret);
```

## 4。每个账户一个模板

既然我们已经看到了如何为**创建一个单独的`TwitterTemplate`账户**，我们可以再次回顾我们的用例——我们需要在几个账户上发 tweet 这意味着我们需要几个`TwitterTemplate`实例。

这些可以根据要求通过简单的机制轻松创建:

```java
@Component
public class TwitterTemplateCreator {
   @Autowired
   private Environment env;

   public Twitter getTwitterTemplate(String accountName) {
      String consumerKey = env.getProperty(accountName + ".consumerKey");
      String consumerSecret = env.getProperty(accountName + ".consumerSecret");
      String accessToken = env.getProperty(accountName + ".accessToken");
      String accessTokenSecret = env.getProperty(accountName + ".accessTokenSecret");
      Preconditions.checkNotNull(consumerKey);
      Preconditions.checkNotNull(consumerSecret);
      Preconditions.checkNotNull(accessToken);
      Preconditions.checkNotNull(accessTokenSecret);

      TwitterTemplate twitterTemplate = 
         new TwitterTemplate(consumerKey, consumerSecret, accessToken, accessTokenSecret);
      return twitterTemplate;
   }
}
```

这四个安全工件当然是按照账户在属性文件中具体化的**；例如，对于 [SpringAtSO 账户](https://web.archive.org/web/20230103152303/https://twitter.com/SpringTip "SpringAtSO twitter account"):**

```java
SpringAtSO.consumerKey=nqYezCjxkHabaX6cdte12g
SpringAtSO.consumerSecret=7REmgFW4SnVWpD4EV5Zy9wB2ZEMM9WKxTaZwrgX3i4A
SpringAtSO.accessToken=1197830142-t44T7vwgmOnue8EoAxI1cDyDAEBAvple80s1SQ3
SpringAtSO.accessTokenSecret=ZIpghEJgFGNGQZzDFBT5TgsyeqDKY2zQmYsounPafE
```

这实现了灵活性和安全性的良好结合——安全证书不是代码库的一部分(代码库[是开源的](https://web.archive.org/web/20230103152303/https://github.com/eugenp/stackexchange2twitter/tree/master/java-stackexchange2twitter "The project @github")),而是独立存在于文件系统中，由 Spring 获取，并通过简单的配置在 Spring 环境中可用:

```java
@Configuration
@PropertySource({ "file:///opt/stack/twitter.properties" })
public class TwitterConfig {
    // 
}
```

Spring 中的属性是一个之前已经讨论过的主题，所以我们不会在这里进一步讨论这个主题。

最后，一个**测试**将验证一个帐户在 Spring 环境中是否有必要的安全信息；如果属性不存在，`getTwitterTemplate`逻辑应该通过`NullPointerException`使测试失败:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { TwitterConfig.class })
public class TwitterTemplateCreatorIntegrationTest {
   @Autowired
   private TwitterTemplateCreator twitterTemplateCreator;
   //
   @Test
   public void givenValidAccountSpringAtSO_whenRetrievingTwitterClient_thenNoException() {
      twitterTemplateCreator.getTwitterTemplate(SimpleTwitterAccount.SpringAtSO.name());
   }
}
```

## 5。推特

随着`TwitterTemplate`的创建，让我们转向**发微博**的实际操作。为此，我们将使用一个非常简单的服务，接受一个`TwitterTemplate`并使用其底层 API 创建一条 tweet:

```java
@Service
public class TwitterService {
   private Logger logger = LoggerFactory.getLogger(getClass());

   public void tweet(Twitter twitter, String tweetText) {
      try {
         twitter.timelineOperations().updateStatus(tweetText);
      } catch (RuntimeException ex) {
         logger.error("Unable to tweet" + tweetText, ex);
      }
   }
}
```

## 6。测试`TwitterTemplate`

最后，我们可以编写一个集成测试来执行为一个帐户提供一个`TwitterTemplate`并在该帐户上发布 test 的整个过程:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { TwitterConfig.class })
public class TweetServiceLiveTest {
   @Autowired
   private TwitterService twitterService;
   @Autowired
   private TwitterTemplateCreator twitterCreator;

   @Test
   public void whenTweeting_thenNoExceptions() {
      Twitter twitterTemplate = twitterCreator.getTwitterTemplate("SpringAtSO");
      twitterService.tweet(twitterTemplate, "First Tweet");
   }
}
```

## 7 .**。结论**

在这一点上，我们创建的 Twitter API 完全独立于 StackExchange API，可以独立于特定用例使用，以发布任何消息。

从 Stack Exchange 帐户发布问题的下一个逻辑步骤是创建一个组件——与我们到目前为止介绍过的 Twitter 和 Stack Exchange API 进行交互——这将是本系列下一篇文章的重点。