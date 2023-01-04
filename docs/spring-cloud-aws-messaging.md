# spring Cloud AWS–消息支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-aws-messaging>

在最后一篇文章中，我们将继续讨论 AWS 消息支持。

Content Series:[This article is part of a series:](javascript:void(0);)[• Spring Cloud AWS – S3](/web/20221208143841/https://www.baeldung.com/spring-cloud-aws-s3)
[• Spring Cloud AWS – EC2](/web/20221208143841/https://www.baeldung.com/spring-cloud-aws-ec2)
[• Spring Cloud AWS – RDS](/web/20221208143841/https://www.baeldung.com/spring-cloud-aws-rds)
• Spring Cloud AWS – Messaging Support (current article)

## 1。AWS 消息支持

### 1.1。SQS(简单队列服务)

我们可以使用`QueueMessagingTemplate`向 SQS 队列发送消息。

要创建这个 bean，我们可以使用一个`AmazonSQSAsync`客户端，在使用 Spring Boot 启动器时，默认情况下这个客户端在应用程序上下文中可用:

```java
@Bean
public QueueMessagingTemplate queueMessagingTemplate(
  AmazonSQSAsync amazonSQSAsync) {
    return new QueueMessagingTemplate(amazonSQSAsync);
}
```

然后，我们可以使用`convertAndSend()`方法发送消息:

```java
@Autowired
QueueMessagingTemplate messagingTemplate;

public void send(String topicName, Object message) {
    messagingTemplate.convertAndSend(topicName, message);
}
```

由于亚马逊 SQS 只接受`String`有效载荷，Java 对象被自动序列化为 JSON。

我们还可以使用 `@SqsListener`来配置监听器:

```java
@SqsListener("spring-cloud-test-queue")
public void receiveMessage(String message, 
  @Header("SenderId") String senderId) {
    // ...
}
```

该方法将接收来自`spring-cloud-test-queue`的消息，然后处理它们。我们还可以在方法参数上使用`@Header`注释来检索消息头。

如果第一个参数是一个定制的 Java 对象而不是`String,`, Spring 将使用 JSON 转换把消息转换成那个类型。

### 1.2。SNS(简单通知服务)

类似于 SQS，我们可以使用`NotificationMessagingTemplate`来发布一个主题的消息。

要创建它，我们需要一个`AmazonSNS`客户端:

```java
@Bean
public NotificationMessagingTemplate notificationMessagingTemplate(
  AmazonSNS amazonSNS) {
    return new NotificationMessagingTemplate(amazonSNS);
}
```

然后，我们可以向主题发送通知:

```java
@Autowired
NotificationMessagingTemplate messagingTemplate;

public void send(String Object message, String subject) {
    messagingTemplate
      .sendNotification("spring-cloud-test-topic", message, subject);
}
```

在 AWS 支持的多个 SNS 端点(SQS、HTTP(S)、电子邮件和 SMS)中，**该项目仅支持 HTTP(S)** 。

我们可以在 MVC 控制器中配置端点:

```java
@Controller
@RequestMapping("/topic-subscriber")
public class SNSEndpointController {

    @NotificationSubscriptionMapping
    public void confirmUnsubscribeMessage(
      NotificationStatus notificationStatus) {
        notificationStatus.confirmSubscription();
    }

    @NotificationMessageMapping
    public void receiveNotification(@NotificationMessage String message, 
      @NotificationSubject String subject) {
        // handle message
    }

    @NotificationUnsubscribeConfirmationMapping
    public void confirmSubscriptionMessage(
      NotificationStatus notificationStatus) {
        notificationStatus.confirmSubscription();
    }
}
```

**我们需要在控制器级别将主题名称添加到`@RequestMapping`注释中。**该控制器启用 HTTP(s)端点–`/topic-subscriber`, SNS 主题使用该端点来创建订阅。

例如，我们可以通过调用 URL 来订阅某个主题:

```java
https://host:port/topic-subscriber/
```

请求中的头决定了调用三种方法中的哪一种。

当标题`[x-amz-sns-message-type=SubscriptionConfirmation]`出现并确认对主题的新订阅时，调用带有`@NotificationSubscriptionMapping`注释的方法。

一旦被订阅，主题将向端点发送带有标题`[x-amz-sns-message-type=Notification]`的通知。这将调用用`@NotificationMessageMapping`标注的方法。

最后，当端点取消订阅主题时，会收到一个带有标题`[x-amz-sns-message-type=UnsubscribeConfirmation]`的确认请求。

这将调用用`@NotificationUnsubscribeConfirmationMapping`标注的方法来确认取消订阅操作。

请注意，`@RequestMapping`中的值与它订阅的主题名称无关。

## 2。结论

在这最后一篇文章中，我们探讨了 Spring Cloud 对 AWS 消息传递的支持——这是关于 Spring Cloud 和 AWS 的快速系列的结论。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143841/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-aws)

**«** Previous[Spring Cloud AWS – RDS](/web/20221208143841/https://www.baeldung.com/spring-cloud-aws-rds)