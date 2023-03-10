# gRPC 中的错误处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/grpcs-error-handling>

## 1.概观

gRPC 是一个做进程间远程过程调用(RPC)的平台。它的性能很高，可以在任何环境下运行。

在本教程中，我们将关注使用 Java 的 gRPC 错误处理。gRPC 具有非常低的延迟和高吞吐量，因此非常适合在微服务架构等复杂环境中使用。在这些系统中，充分了解网络不同组件的状态、性能和故障至关重要。因此，良好的错误处理实现对于帮助我们实现前面的目标至关重要。

## 2.gRPC 中错误处理的基础

gRPC 中的错误是一级实体，即**gRPC 中的每个调用要么是有效载荷消息，要么是状态错误消息**。

**错误被编入状态消息，并在所有支持的语言中实现**。

一般来说，我们不应该在响应负载中包含错误。为此，始终使用`StreamObserver::` `OnError,` ，它在内部将状态错误添加到尾部标题。唯一的例外，正如我们将在下面看到的，是当我们处理流的时候。

所有客户端或服务器 gRPC 库都支持官方 gRPC 错误模型。Java 用类` **io.grpc.Status**`封装了这个错误模型。这个类**需要一个标准的[错误状态代码](https://web.archive.org/web/20220529021919/https://grpc.github.io/grpc/core/md_doc_statuscodes.html)和一个可选的字符串错误消息来提供附加信息**。这种错误模型的优点在于，它不受所使用的数据编码(协议缓冲区、REST 等)的支持。).但是，它非常有限，因为我们不能在状态中包含错误细节。

如果您的 gRPC 应用程序实现了用于数据编码的协议缓冲区，那么您可以为 Google API 使用更丰富的[错误模型。 **`com.google.rpc.Status`** 类封装了这个错误模型。这个类**提供了`com.google.rpc.Code` 值，一个错误消息，附加的错误细节**被附加为`protobuf`消息。此外，我们可以利用一组预定义的`protobuf`错误消息，这些消息在](https://web.archive.org/web/20220529021919/https://cloud.google.com/apis/design/errors#error_model)`[error_details.proto](https://web.archive.org/web/20220529021919/https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto)`中定义，涵盖了最常见的情况。在包`com.google.rpc`中，我们有类:`RetryInfo`、`DebugInfo`、`QuotaFailure`、`ErrorInfo`、`PrecondicionFailure`、`BadRequest`、`RequestInfo`、`ResourceInfo,`和`Help`，这些类封装了[、`error_details.proto`、](https://web.archive.org/web/20220529021919/https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto)中的所有错误信息。

**除了这两个错误模型，我们还可以定义定制的错误消息，作为键值对添加到 RPC 元数据**。

我们将编写一个非常简单的应用程序，展示如何将这些错误模型用于定价服务，其中客户端发送商品名称，服务器提供定价值。

## 3.一元 RPC 调用

让我们开始考虑下面在`commodity_price.proto`中定义的服务接口:

```java
service CommodityPriceProvider {
    rpc getBestCommodityPrice(Commodity) returns (CommodityQuote) {}
}

message Commodity {
    string access_token = 1;
    string commodity_name = 2;
}

message CommodityQuote {
    string commodity_name = 1;
    string producer_name = 2;
    double price = 3;
}

message ErrorResponse {
    string commodity_name = 1;
    string access_token = 2;
    string expected_token = 3;
    string expected_value = 4;
}
```

服务的输入是一个`Commodity`消息。在请求中，客户端必须提供一个`access_token`和一个`commodity_name`。

服务器用一个`CommodityQuote`同步响应，它为`Commodity`声明`comodity_name`、`producer_name,`和相关的`price`。

为了便于说明，我们还定义了一个自定义的`ErrorResponse`。这是一个自定义错误消息的例子，我们将把它作为元数据发送给客户端。

### 3.1.使用`io.grpc.Status`进行响应

在服务器的服务调用中，我们检查有效的`Commodity`请求:

```java
public void getBestCommodityPrice(Commodity request, StreamObserver<CommodityQuote> responseObserver) {

    if (commodityLookupBasePrice.get(request.getCommodityName()) == null) {

        Metadata.Key<ErrorResponse> errorResponseKey = ProtoUtils.keyForProto(ErrorResponse.getDefaultInstance());
        ErrorResponse errorResponse = ErrorResponse.newBuilder()
          .setCommodityName(request.getCommodityName())
          .setAccessToken(request.getAccessToken())
          .setExpectedValue("Only Commodity1, Commodity2 are supported")
          .build();
        Metadata metadata = new Metadata();
        metadata.put(errorResponseKey, errorResponse);
        responseObserver.onError(io.grpc.Status.INVALID_ARGUMENT.withDescription("The commodity is not supported")
          .asRuntimeException(metadata));
    } 
    // ...
}
```

在这个简单的例子中，如果`commodityLookupBasePrice` `HashTable`中不存在`Commodity`，我们将返回一个错误。

首先，我们构建一个定制的`ErrorResponse`并创建一个键值对，我们将它添加到`metadata.put(errorResponseKey, errorResponse)`中的元数据中。

我们使用`io.grpc.Status`来指定错误状态。函数 **`responseObserver::onError`以一个`Throwable` 作为参数，所以我们用`asRuntimeException(metadata)` 将`Status`转换成一个`Throwable`** 。`asRuntimeException`可以选择接受一个元数据参数(在我们的例子中，是一个`ErrorResponse`键-值对)，它添加到消息的尾部。

如果客户端发出无效请求，它将返回一个异常:

```java
@Test
public void whenUsingInvalidCommodityName_thenReturnExceptionIoRpcStatus() throws Exception {

    Commodity request = Commodity.newBuilder()
      .setAccessToken("123validToken")
      .setCommodityName("Commodity5")
      .build();

    StatusRuntimeException thrown = Assertions.assertThrows(StatusRuntimeException.class, () -> blockingStub.getBestCommodityPrice(request));

    assertEquals("INVALID_ARGUMENT", thrown.getStatus().getCode().toString());
    assertEquals("INVALID_ARGUMENT: The commodity is not supported", thrown.getMessage());
    Metadata metadata = Status.trailersFromThrowable(thrown);
    ErrorResponse errorResponse = metadata.get(ProtoUtils.keyForProto(ErrorResponse.getDefaultInstance()));
    assertEquals("Commodity5",errorResponse.getCommodityName());
    assertEquals("123validToken", errorResponse.getAccessToken());
    assertEquals("Only Commodity1, Commodity2 are supported", errorResponse.getExpectedValue());
}
```

对`blockingStub::getBestCommodityPrice`的调用抛出了一个`StatusRuntimeExeption`，因为请求有一个无效的商品名称。

我们使用`Status::trailerFromThrowable`来访问元数据。`ProtoUtils::keyForProto`给出了`ErrorResponse`的元数据键。

### 3.2.使用`com.google.rpc.Status`进行响应

让我们考虑下面的服务器代码示例:

```java
public void getBestCommodityPrice(Commodity request, StreamObserver<CommodityQuote> responseObserver) {
    // ...
    if (request.getAccessToken().equals("123validToken") == false) {

        com.google.rpc.Status status = com.google.rpc.Status.newBuilder()
          .setCode(com.google.rpc.Code.NOT_FOUND.getNumber())
          .setMessage("The access token not found")
          .addDetails(Any.pack(ErrorInfo.newBuilder()
            .setReason("Invalid Token")
            .setDomain("com.baeldung.grpc.errorhandling")
            .putMetadata("insertToken", "123validToken")
            .build()))
          .build();
        responseObserver.onError(StatusProto.toStatusRuntimeException(status));
    }
    // ...
}
```

在实现中，如果请求没有有效的令牌，`getBestCommodityPrice`将返回一个错误。

此外，我们将状态代码、消息和细节设置为`com.google.rpc.Status`。

在这个例子中，我们使用预定义的`com.google.rpc.ErrorInfo`而不是自定义的`ErrorDetails`(尽管如果需要的话，我们可以两者都使用)。我们用`Any::pack()`连载`ErrorInfo`。

类`StatusProto::toStatusRuntimeException`将`com.google.rpc.Status`转换成一个`Throwable`。

原则上，我们还可以添加在`error_details.proto`中定义的其他消息来进一步定制响应。

客户端实现非常简单:

```java
@Test
public void whenUsingInvalidRequestToken_thenReturnExceptionGoogleRPCStatus() throws Exception {

    Commodity request = Commodity.newBuilder()
      .setAccessToken("invalidToken")
      .setCommodityName("Commodity1")
      .build();

    StatusRuntimeException thrown = Assertions.assertThrows(StatusRuntimeException.class,
      () -> blockingStub.getBestCommodityPrice(request));
    com.google.rpc.Status status = StatusProto.fromThrowable(thrown);
    assertNotNull(status);
    assertEquals("NOT_FOUND", Code.forNumber(status.getCode()).toString());
    assertEquals("The access token not found", status.getMessage());
    for (Any any : status.getDetailsList()) {
        if (any.is(ErrorInfo.class)) {
            ErrorInfo errorInfo = any.unpack(ErrorInfo.class);
            assertEquals("Invalid Token", errorInfo.getReason());
            assertEquals("com.baeldung.grpc.errorhandling", errorInfo.getDomain());
            assertEquals("123validToken", errorInfo.getMetadataMap().get("insertToken"));
        }
    }
}
```

`StatusProto.fromThrowable`是直接从异常中获取`com.google.rpc.Status`的实用方法。

从`status::getDetailsList`我们得到`com.google.rpc.ErrorInfo`的细节。

## 4.gRPC 流的错误

gRPC 流允许服务器和客户端在一次 RPC 调用中发送多条消息。

**就错误传播而言，我们目前使用的方法对 gRPC 流**无效。**原因是`onError()`必须是 RPC** 中调用的最后一个方法，因为在这个调用之后，框架切断了客户端和服务器之间的通信。

当我们使用流时，这不是我们想要的行为。相反，我们希望保持连接开放，以响应可能通过 RPC 传来的其他消息。

这个问题的一个好的**解决方案是将错误添加到消息本身**，如我们在`commodity_price.proto`中所示:

```java
service CommodityPriceProvider {

    rpc getBestCommodityPrice(Commodity) returns (CommodityQuote) {}

    rpc bidirectionalListOfPrices(stream Commodity) returns (stream StreamingCommodityQuote) {}
}

message Commodity {
    string access_token = 1;
    string commodity_name = 2;
}

message StreamingCommodityQuote{
    oneof message{
        CommodityQuote comodity_quote = 1;
        google.rpc.Status status = 2;
   }   
}
```

函数`bidirectionalListOfPrices`返回一个`StreamingCommodityQuote`。这条消息有`oneof`关键字，表明它可以使用`CommodityQuote`或`google.rpc.Status`。

在以下示例中，如果客户端发送无效令牌，服务器会在响应正文中添加一个状态错误:

```java
public StreamObserver<Commodity> bidirectionalListOfPrices(StreamObserver<StreamingCommodityQuote> responseObserver) {

    return new StreamObserver<Commodity>() {
        @Override
        public void onNext(Commodity request) {

            if (request.getAccessToken().equals("123validToken") == false) {

                com.google.rpc.Status status = com.google.rpc.Status.newBuilder()
                  .setCode(Code.NOT_FOUND.getNumber())
                  .setMessage("The access token not found")
                  .addDetails(Any.pack(ErrorInfo.newBuilder()
                    .setReason("Invalid Token")
                    .setDomain("com.baeldung.grpc.errorhandling")
                    .putMetadata("insertToken", "123validToken")
                    .build()))
                  .build();
                StreamingCommodityQuote streamingCommodityQuote = StreamingCommodityQuote.newBuilder()
                  .setStatus(status)
                  .build();
                responseObserver.onNext(streamingCommodityQuote);
            }
            // ...
        }
    }
}
```

**代码创建一个`com.google.rpc.Status`的实例，并将其添加到`StreamingCommodityQuote`响应消息**中。它不调用`onError(),`，所以框架不会中断与客户端的连接。

让我们看看客户端实现:

```java
public void onNext(StreamingCommodityQuote streamingCommodityQuote) {

    switch (streamingCommodityQuote.getMessageCase()) {
        case COMODITY_QUOTE:
            CommodityQuote commodityQuote = streamingCommodityQuote.getComodityQuote();
            logger.info("RESPONSE producer:" + commodityQuote.getCommodityName() + " price:" + commodityQuote.getPrice());
            break;
        case STATUS:
            com.google.rpc.Status status = streamingCommodityQuote.getStatus();
            logger.info("Status code:" + Code.forNumber(status.getCode()));
            logger.info("Status message:" + status.getMessage());
            for (Any any : status.getDetailsList()) {
                if (any.is(ErrorInfo.class)) {
                    ErrorInfo errorInfo;
                    try {
                        errorInfo = any.unpack(ErrorInfo.class);
                        logger.info("Reason:" + errorInfo.getReason());
                        logger.info("Domain:" + errorInfo.getDomain());
                        logger.info("Insert Token:" + errorInfo.getMetadataMap().get("insertToken"));
                    } catch (InvalidProtocolBufferException e) {
                        logger.error(e.getMessage());
                    }
                }
            }
            break;
        // ...
    }
}
```

**客户端** **获取`onNext(StreamingCommodityQuote)`返回的消息，** **使用`switch`语句区分`CommodityQuote`或`com.google.rpc.Status`** 。

## 5.结论

在本教程中，我们已经展示了如何在 gRPC 中为一元和基于流的 RPC 调用实现错误处理。

gRPC 是用于分布式系统中远程通信的一个很好的框架。在这些系统中，有一个非常健壮的错误处理实现来帮助监控系统是很重要的。这在微服务这样的复杂架构中更为关键。

这些例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220529021919/https://github.com/eugenp/tutorials/tree/master/grpc)