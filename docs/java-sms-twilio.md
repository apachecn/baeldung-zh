# 用 Twilio 用 Java 发送短信

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sms-twilio>

## 1.介绍

发送 SMS 消息是许多现代应用程序的重要组成部分。SMS 消息可以服务于多种用例:双因素身份验证、实时提醒、聊天机器人等等。

在本教程中，我们将构建一个使用 [Twilio](https://web.archive.org/web/20220627090128/https://www.twilio.com/) 发送 SMS 消息的简单 Java 应用程序。

提供短信功能的服务有很多，比如 [Vonage](https://web.archive.org/web/20220627090128/https://www.vonage.com/) 、 [Plivo](https://web.archive.org/web/20220627090128/https://www.plivo.com/) 、亚马逊[简单通知服务](https://web.archive.org/web/20220627090128/https://aws.amazon.com/sns/) (SNS)、 [Zapier](https://web.archive.org/web/20220627090128/https://www.zapier.com/) 等等。

使用 Twilio Java 客户端，**我们只需几行代码**就可以发送短信。

## 2.设置 Twilio

首先，我们需要一个 Twilio 账户。他们提供一个试用账户，足以测试他们平台的每一项功能。

作为帐户设置的一部分，我们还必须创建一个电话号码。这一点很重要，因为试用帐户需要一个经过验证的电话号码来发送邮件。

Twilio 为新账户提供了[快速设置教程](https://web.archive.org/web/20220627090128/https://www.twilio.com/docs/sms/quickstart/java)。完成帐户设置并验证电话号码后，我们就可以开始发送消息了。

## 3.TwiML 简介

在编写我们的示例应用程序之前，让我们快速了解一下用于 Twilio 服务的数据交换格式。

TwiML 是一种基于 XML 的专有标记语言。TwiML 消息中的元素反映了我们可以采取的与电话相关的不同操作:打电话、记录消息、发送消息等等。

以下是发送 SMS 的 TwiML 消息示例:

```java
<Response>
    <Message>
        <Body>Sample Twilio SMS</Body>
    </Message>
</Response>
```

这是另一个打电话的 TwiML 消息的例子:

```java
<Response>
    <Dial>
        <Number>415-123-4567</Number>
    </Dial>
</Response>
```

这些都是微不足道的例子，但是它们让我们很好地理解了 TwiML 的样子。它由易于记忆的动词和名词组成，与我们用手机执行的操作直接相关。

## 4.用 Twilio 用 Java 发送短信

Twilio 提供了一个丰富的 Java 客户端，使得与他们的服务交互变得容易。我们可以使用现成的 Java 客户端，而不必从头开始编写构建 TwiML 消息的代码。

### 4.1.属国

我们可以直接从 [Maven Central](https://web.archive.org/web/20220627090128/https://search.maven.org/classic/#artifactdetails%7Ccom.twilio.sdk%7Ctwilio%7C7.20.0%7Cjar) 下载依赖项，或者将以下条目添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>com.twilio.sdk</groupId>
    <artifactId>twilio</artifactId>
    <version>7.20.0</version>
</dependency>
```

### 4.2.发送短信

首先，让我们看一些示例代码:

```java
Twilio.init(ACCOUNT_SID, AUTH_TOKEN);
Message message = Message.creator(
    new PhoneNumber("+12225559999"),
    new PhoneNumber(TWILIO_NUMBER),
    "Sample Twilio SMS using Java")
.create();
```

让我们将上面示例中的代码分解成几个关键部分:

*   使用我们唯一的帐户 Sid 和令牌设置 Twilio 环境需要一次`Twilio.init()`调用
*   `Message`对象是相当于我们前面看到的 TwiML `<Message>`元素的 Java
*   `Message.creator()`需要 3 个参数:收件人电话号码、发件人电话号码和消息正文
*   `create()`方法处理发送消息

### 4.3.发送彩信

Twilio API 还支持发送彩信。我们可以混合搭配文字和图像，为此，接收手机必须支持媒体信息:

```java
Twilio.init(ACCOUNT_SID, AUTH_TOKEN);
Message message = Message.creator(
    new PhoneNumber("+12225559999"),
    new PhoneNumber(TWILIO_NUMBER),
    "Sample Twilio MMS using Java")
.setMediaUrl(
    Promoter.listOfOne(URI.create("http://www.domain.com/image.png")))
.create();
```

## 5.跟踪邮件状态

在前面的例子中，我们没有确认消息是否真的被传递了。然而 **Twilio 为我们提供了一种机制来确定消息是否被成功传递**。

### 5.1.消息状态代码

发送消息时，它将随时处于以下状态之一:

*   `Queued`–Twilio 已收到消息，并将其排队等待发送
*   `Sending`–服务器正在将您的信息发送到网络中最近的上游运营商
*   `Sent`–消息被最近的上游运营商成功接收
*   `Delivered`–Twilio 已收到来自上游运营商的消息传递确认，可能还有目的手机(如果可用)
*   `Failed`–无法发送信息
*   `Undelivered`–服务器收到一张送达回执，表明邮件未送达

请注意，对于最后两种状态，我们可以找到一个包含更具体细节的错误代码，以帮助我们解决交付问题。

Twilio Java 客户端提供了同步和异步方法来获取状态。让我们看一看。

### 5.2.检查交付状态(同步)

一旦我们创建了一个`Message`对象，我们可以调用`Message.getStatus()`来查看它当前的状态:

```java
Twilio.init(ACCOUNT_SID, AUTH_TOKEN);
ResourceSet messages = Message.reader().read();
for (Message message : messages) {
    System.out.println(message.getSid() + " : " + message.getStatus());
}
```

注意,`Message.reader().read()`进行远程 API 调用，所以要谨慎使用。**默认情况下，它会返回我们发送给**的所有消息，但我们可以根据电话号码或日期范围过滤返回的消息。

### 5.3.检查交付状态(异步)

因为检索消息状态需要远程 API 调用，所以可能需要很长时间。为了避免阻塞当前线程，Twilio Java 客户端还提供了异步版本的`Message.getStatus().read()`。

```java
Twilio.init(ACCOUNT_SID, AUTH_TOKEN);
ListenableFuture<ResourceSet<Message>> future = Message.reader().readAsync();
Futures.addCallback(
    future,
    new FutureCallback<ResourceSet<Message>>() {
        public void onSuccess(ResourceSet<Message> messages) {
            for (Message message : messages) {
                System.out.println(message.getSid() + " : " + message.getStatus());
             }
         }
         public void onFailure(Throwable t) {
             System.out.println("Failed to get message status: " + t.getMessage());
         }
     });
```

它使用 Guava `ListenableFuture`接口在不同的线程上处理来自 Twilio 的响应。

## 6.结论

在本文中，我们学习了如何使用 Twilio 和 Java 发送短信和彩信。

虽然 TwiML 是所有往来于 Twilio 服务器的消息的基础，但是 Twilio Java 客户端使发送消息变得非常容易。

和往常一样，这个例子的完整代码库可以在我们的 [GitHub 库](https://web.archive.org/web/20220627090128/https://github.com/eugenp/tutorials/tree/master/twilio)中找到。