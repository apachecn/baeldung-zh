# 用 Java 管理亚马逊 SQS 队列

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aws-queues-java>

## 1。概述

在本教程中，我们将探索如何使用 Java SDK 来**使用亚马逊的 [SQS](https://web.archive.org/web/20220524015952/https://aws.amazon.com/sqs/) (简单队列服务)。**

## 2。先决条件

使用 Amazon AWS SDK for SQS 所需的 Maven 依赖项、AWS 帐户设置和客户端连接与本文中的[相同。](/web/20220524015952/https://www.baeldung.com/aws-s3-java)

假设我们已经创建了前一篇文章中描述的`AWSCredentials,`的实例，我们可以继续创建我们的 SQS 客户端:

```
AmazonSQS sqs = AmazonSQSClientBuilder.standard()
  .withCredentials(new AWSStaticCredentialsProvider(credentials))
  .withRegion(Regions.US_EAST_1)
  .build(); 
```

## 3。创建队列

一旦我们建立了 SQS 客户端，**创建队列相当简单。**

### 3.1。创建标准队列

让我们看看如何创建一个标准队列。为此，**我们需要创建一个`CreateQueueRequest:`** 的实例

```
CreateQueueRequest createStandardQueueRequest = new CreateQueueRequest("baeldung-queue");
String standardQueueUrl = sqs.createQueue(createStandardQueueRequest).getQueueUrl(); 
```

### 3.2。创建 FIFO 队列

创建 FIFO 类似于创建标准队列。我们仍将使用`CreateQueueRequest`的一个实例，就像我们之前做的那样。只是这一次，**我们必须传入队列属性，并将`FifoQueue`属性设置为`true` :**

```
Map<String, String> queueAttributes = new HashMap<>();
queueAttributes.put("FifoQueue", "true");
queueAttributes.put("ContentBasedDeduplication", "true");
CreateQueueRequest createFifoQueueRequest = new CreateQueueRequest(
  "baeldung-queue.fifo").withAttributes(queueAttributes);
String fifoQueueUrl = sqs.createQueue(createFifoQueueRequest)
  .getQueueUrl(); 
```

## 4。向队列发布消息

一旦我们设置好队列，我们就可以开始发送消息了。

### 4.1。向标准队列发送消息

为了向标准队列发送消息，我们将**必须创建一个`SendMessageRequest.`** 的实例

然后，我们将消息属性映射附加到该请求:

```
Map<String, MessageAttributeValue> messageAttributes = new HashMap<>();
messageAttributes.put("AttributeOne", new MessageAttributeValue()
  .withStringValue("This is an attribute")
  .withDataType("String"));  

SendMessageRequest sendMessageStandardQueue = new SendMessageRequest()
  .withQueueUrl(standardQueueUrl)
  .withMessageBody("A simple message.")
  .withDelaySeconds(30)
  .withMessageAttributes(messageAttributes);

sqs.sendMessage(sendMessageStandardQueue); 
```

`withDelaySeconds() `指定消息应该在多长时间后到达队列。

### 4.2。向 FIFO 队列发送消息

在这种情况下，唯一的区别是**我们必须指定消息所属的[`group`](https://web.archive.org/web/20220524015952/https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html):**

```
SendMessageRequest sendMessageFifoQueue = new SendMessageRequest()
  .withQueueUrl(fifoQueueUrl)
  .withMessageBody("Another simple message.")
  .withMessageGroupId("baeldung-group-1")
  .withMessageAttributes(messageAttributes);
```

正如您在上面的代码示例中看到的，我们使用`withMessageGroupId().`来指定组

### 4.3。向队列发布多条消息

我们也可以**使用一个请求向一个队列发送多条消息。**我们将创建一个`SendMessageBatchRequestEntry` 列表，我们将使用`SendMessageBatchRequest`的实例发送该列表:

```
List <SendMessageBatchRequestEntry> messageEntries = new ArrayList<>();
messageEntries.add(new SendMessageBatchRequestEntry()
  .withId("id-1")
  .withMessageBody("batch-1")
  .withMessageGroupId("baeldung-group-1"));
messageEntries.add(new SendMessageBatchRequestEntry()
  .withId("id-2")
  .withMessageBody("batch-2")
  .withMessageGroupId("baeldung-group-1"));

SendMessageBatchRequest sendMessageBatchRequest
 = new SendMessageBatchRequest(fifoQueueUrl, messageEntries);
sqs.sendMessageBatch(sendMessageBatchRequest);
```

## 5。从队列中读取消息

我们可以通过**调用`ReceiveMessageRequest:`** 实例上的`receiveMessage() `方法来接收队列中的消息

```
ReceiveMessageRequest receiveMessageRequest = new ReceiveMessageRequest(fifoQueueUrl)
  .withWaitTimeSeconds(10)
  .withMaxNumberOfMessages(10);

List<Message> sqsMessages = sqs.receiveMessage(receiveMessageRequest).getMessages(); 
```

使用`withMaxNumberOfMessages(),` 我们指定从队列中获取多少消息——尽管应该注意最大值是`10`。

方法`withWaitTimeSeconds()`启用`[long-polling](https://web.archive.org/web/20220524015952/https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-long-polling.html).`长轮询是一种限制我们发送到 SQS 的接收消息请求数量的方法。

简单地说，这意味着我们将等待指定的秒数来检索消息。如果在这段时间内队列中没有消息，那么请求将返回空值。如果消息在此期间到达队列，它将被返回。

我们可以**获得给定消息的属性和正文:**

```
sqsMessages.get(0).getAttributes();
sqsMessages.get(0).getBody();
```

## 6。从队列中删除消息

要删除消息，我们将使用一个`DeleteMessageRequest`:

```
sqs.deleteMessage(new DeleteMessageRequest()
  .withQueueUrl(fifoQueueUrl)
  .withReceiptHandle(sqsMessages.get(0).getReceiptHandle())); 
```

## 7 .**。死信队列**

**一个[死信队列](https://web.archive.org/web/20220524015952/https://en.wikipedia.org/wiki/Dead_letter_queue)必须和它的基本队列类型相同—** 如果基本队列是 FIFO，它必须是 FIFO，如果基本队列是标准队列，它必须是标准队列。对于这个例子，我们将使用一个标准队列。

我们需要做的第一件事是**创建将成为死信队列的内容:**

```
String deadLetterQueueUrl = sqs.createQueue("baeldung-dead-letter-queue").getQueueUrl(); 
```

接下来，我们将**获取我们新创建的队列的 [ARN(亚马逊资源名称)](https://web.archive.org/web/20220524015952/https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) :**

```
GetQueueAttributesResult deadLetterQueueAttributes = sqs.getQueueAttributes(
  new GetQueueAttributesRequest(deadLetterQueueUrl)
    .withAttributeNames("QueueArn"));

String deadLetterQueueARN = deadLetterQueueAttributes.getAttributes()
  .get("QueueArn"); 
```

最后，我们**将这个新创建的队列设置为我们原来的标准队列的死信队列:**

```
SetQueueAttributesRequest queueAttributesRequest = new SetQueueAttributesRequest()
  .withQueueUrl(standardQueueUrl)
  .addAttributesEntry("RedrivePolicy",
    "{\"maxReceiveCount\":\"2\", "
      + "\"deadLetterTargetArn\":\"" + deadLetterQueueARN + "\"}");

sqs.setQueueAttributes(queueAttributesRequest); 
```

**在构建我们的`SetQueueAttributesRequest `实例时，我们在`addAttributesEntry()`方法中设置的 JSON 包包含了我们需要的信息**:`maxReceiveCount`是`2`，这意味着如果一个消息被接收了这么多次，它就被认为没有被正确处理，并被发送到我们的死信队列。

属性将我们的标准队列指向我们新创建的死信队列。

## 8。监控

我们可以**检查给定队列中当前有多少消息，以及有多少[正在使用 SDK](https://web.archive.org/web/20220524015952/https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)进行处理。**首先，我们需要创建一个`GetQueueAttributesRequest. `

从这里，我们将检查队列的状态:

```
GetQueueAttributesRequest getQueueAttributesRequest 
  = new GetQueueAttributesRequest(standardQueueUrl)
    .withAttributeNames("All");
GetQueueAttributesResult getQueueAttributesResult 
  = sqs.getQueueAttributes(getQueueAttributesRequest);
System.out.println(String.format("The number of messages on the queue: %s", 
  getQueueAttributesResult.getAttributes()
    .get("ApproximateNumberOfMessages")));
System.out.println(String.format("The number of messages in flight: %s", 
  getQueueAttributesResult.getAttributes()
    .get("ApproximateNumberOfMessagesNotVisible")));
```

使用[亚马逊云手表](https://web.archive.org/web/20220524015952/https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-monitoring-using-cloudwatch.html)可以实现更深入的监控。

## 9。结论

在本文中，我们看到了如何使用 AWS Java SDK 管理 SQS 队列。

像往常一样，本文中使用的所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20220524015952/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-miscellaneous)