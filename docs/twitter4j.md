# Twitter4J 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/twitter4j>

## 1。概述

在本文中，我们将了解如何在 Java 应用程序中使用 [Twitter4J](https://web.archive.org/web/20220814014500/http://twitter4j.org/en/index.html) 与 Twitter 进行通信。

## 2。Twitter4J

[Twitter4J](https://web.archive.org/web/20220814014500/http://twitter4j.org/en/index.html) 是一个开源的 Java 库，它提供了一个访问 [Twitter API](https://web.archive.org/web/20220814014500/https://dev.twitter.com/docs) 的便捷 API。

简单地说，下面是我们如何与 Twitter API 进行交互；我们可以:

*   发布推文
*   获取用户的时间表，包括最新的推文列表
*   发送和接收直接消息
*   搜索推文等等

这个库确保我们可以轻松地进行这些操作，并且它还确保了用户的安全性和隐私性——为此我们自然需要在我们的应用程序中配置 OAuth 凭证。

## 3。Maven 依赖关系

我们需要从在我们的`pom.xml`中定义 Twitter4J 的依赖关系开始:

```java
<dependency>
    <groupId>org.twitter4j</groupId>
    <artifactId>twitter4j-stream</artifactId>
    <version>4.0.6</version>
</dependency>
```

要检查是否有新版本的库发布-[在这里跟踪发布版本](https://web.archive.org/web/20220814014500/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.twitter4j%22%20AND%20a%3A%22twitter4j%22)。

## 4。配置

配置 Twitter4J 很容易，可以通过多种方式完成——例如在纯文本文件或 Java 类中，甚至使用环境变量。

让我们一次一个地看看这些方法。

### 4.1。纯文本文件

我们可以使用一个名为`twitter4j.properties`的纯文本文件来保存我们的配置细节。让我们看看需要提供的属性:

```java
oauth.consumerKey =       // your key
oauth.consumerSecret =    // your secret
oauth.accessToken =       // your token
oauth.accessTokenSecret = // your token secret
```

所有这些属性都可以在你[做了一个新的 app](https://web.archive.org/web/20220814014500/https://apps.twitter.com/) 之后从 Twitter 开发者控制台获得。

### 4.2。Java 类

我们还可以使用一个 [ConfigurationBuilder](https://web.archive.org/web/20220814014500/http://twitter4j.org/ja/javadoc/twitter4j/conf/ConfigurationBuilder.html) 类在 Java 中以编程方式配置 Twitter4J:

```java
ConfigurationBuilder cb = new ConfigurationBuilder();
cb.setDebugEnabled(true)
  .setOAuthConsumerKey("your consumer key")
  .setOAuthConsumerSecret("your consumer secret")
  .setOAuthAccessToken("your access token")
  .setOAuthAccessTokenSecret("your access token secret");
TwitterFactory tf = new TwitterFactory(cb.build());
Twitter twitter = tf.getInstance();
```

注意，我们将在下一节中使用`Twitter`实例——当我们开始获取数据时。

### 4.3。环境变量

通过环境变量进行配置是我们的另一种选择。如果我们这样做，请注意，我们需要在变量中添加一个`twitter4j`前缀:

```java
$ export twitter4j.oauth.consumerKey =       // your key
$ export twitter4j.oauth.consumerSecret =    // your secret
$ export twitter4j.oauth.accessToken =       // your access token
$ export twitter4j.oauth.accessTokenSecret = // your access token secret
```

## 5。添加/检索实时推文数据

有了完整配置的应用程序，我们终于可以与 Twitter 交互了。

让我们看几个例子。

### 5.1。发布一条推文

我们先在 Twitter 上更新一条推文:

```java
public String createTweet(String tweet) throws TwitterException {
    Twitter twitter = getTwitterinstance();
    Status status = twitter.updateStatus("creating baeldung API");
    return status.getText();
}
```

通过使用`status.getText(),`,我们可以检索刚刚发布的推文。

### 5.2。获取时间线

我们还可以从用户的时间表中获取推文列表:

```java
public List<String> getTimeLine() throws TwitterException {
    Twitter twitter = getTwitterinstance();

    return twitter.getHomeTimeline().stream()
      .map(item -> item.getText())
      .collect(Collectors.toList());
}
```

通过使用`twitter.getHomeTimeline(),`,我们获得了当前账户 ID 发布的所有推文。

### 5.3。发送直接消息

也可以使用 Twitter4j 向关注者发送和接收直接消息:

```java
public static String sendDirectMessage(String recipientName, String msg) 
  throws TwitterException {

    Twitter twitter = getTwitterinstance();
    DirectMessage message = twitter.sendDirectMessage(recipientName, msg);
    return message.getText();
}
```

`sendDirectMessage`方法有两个参数:

*   `RecipientName`:消息接收者的 twitter 用户名
*   `msg`:消息内容

如果找不到接收者，`sendDirectMessage` 将抛出一个异常，异常代码为`150`。

### 5.4。搜索推文

我们也可以搜索包含一些文本的推文。通过这样做，我们将获得带有用户名的 tweets 列表。

让我们看看如何执行这样的搜索:

```java
public static List<String> searchtweets() throws TwitterException {

    Twitter twitter = getTwitterinstance();
    Query query = new Query("source:twitter4j baeldung");
    QueryResult result = twitter.search(query);

    return result.getTweets().stream()
      .map(item -> item.getText())
      .collect(Collectors.toList());
}
```

显然，我们可以迭代在`QueryResult` 中收到的每条 tweet，并获取相关数据。

### 5.5。流媒体应用编程接口

当需要实时更新时，Twitter 流 API 非常有用；它处理线程创建并监听事件。

让我们创建一个侦听器来侦听来自用户的 tweet 更新:

```java
public static void streamFeed() {

    StatusListener listener = new StatusListener() {

        @Override
        public void onException(Exception e) {
            e.printStackTrace();
        }
        @Override
        public void onDeletionNotice(StatusDeletionNotice arg) {
        }
        @Override
        public void onScrubGeo(long userId, long upToStatusId) {
        }
        @Override
        public void onStallWarning(StallWarning warning) {
        }
        @Override
        public void onStatus(Status status) {
        }
        @Override
        public void onTrackLimitationNotice(int numberOfLimitedStatuses) {
        }
    };

    TwitterStream twitterStream = new TwitterStreamFactory().getInstance();

    twitterStream.addListener(listener);

    twitterStream.sample();
}
```

我们可以放一些`println()`语句来检查所有方法中输出的 tweet 流。所有推文都有与之相关的位置元数据。

请注意，API 获取的所有 tweets 数据都是 UTF-8 格式，由于 Twitter 是一个多语言平台，一些数据格式可能无法识别。

## 6。结论

本文快速而全面地介绍了如何在 Java 中使用 Twitter4J。

所示示例[的实现可以在 GitHub](https://web.archive.org/web/20220814014500/https://github.com/eugenp/tutorials/tree/master/twitter4j) 上找到——这是一个基于 Maven 的项目，因此它应该很容易导入和运行。**我们需要做的唯一改变是插入我们自己的 OAuth 凭证。**