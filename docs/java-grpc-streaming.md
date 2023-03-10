# 使用 Java 中的 gRPC 进行流式传输

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-grpc-streaming>

## 1.概观

gRPC 是一个进行进程间远程过程调用(RPC)的平台。它遵循客户机-服务器模型，具有高性能，并支持最重要的计算机语言。查看我们的文章[gRPC 简介](/web/20221205160713/https://www.baeldung.com/grpc-introduction)获得好评。

在本教程中，我们将重点关注 gRPC 流。**流允许在服务器和客户端之间复用消息，创建非常高效和灵活的进程间通信**。

## 2.gRPC 流的基础知识

gRPC 使用 [HTTP/2](https://web.archive.org/web/20221205160713/https://en.wikipedia.org/wiki/HTTP/2) 网络协议进行服务间通信**。** **一个****HTTP/2 的关键优势是它支持流。**每个流可以复用共享单个连接的多个双向消息。

在 gRPC 中，我们有三种功能呼叫类型的流:

1.  服务器流 RPC:客户端向服务器发送一个请求，并返回它顺序读取的几个消息。
2.  客户端流 RPC:客户端向服务器发送一系列消息。客户端等待服务器处理消息并读取返回的响应。
3.  双向流 RPC:客户端和服务器可以来回发送多条消息。消息的接收顺序与发送顺序相同。但是，服务器或客户端可以按照自己选择的顺序响应收到的消息。

为了演示如何使用这些过程调用，我们将编写一个简单的客户机-服务器应用程序示例，用于交换股票信息。

## 3.服务定义

我们使用`stock_quote.proto`来定义服务接口和有效载荷消息的结构:

```java
service StockQuoteProvider {

  rpc serverSideStreamingGetListStockQuotes(Stock) returns (stream StockQuote) {}

  rpc clientSideStreamingGetStatisticsOfStocks(stream Stock) returns (StockQuote) {}

  rpc bidirectionalStreamingGetListsStockQuotes(stream Stock) returns (stream StockQuote) {}
}
message Stock {
   string ticker_symbol = 1;
   string company_name = 2;
   string description = 3;
}
message StockQuote {
   double price = 1;
   int32 offer_number = 2;
   string description = 3;
}
```

**`StockQuoteProvider`服务有三种支持消息流的方法类型。**在下一节，我们将讨论它们的实现。

我们从服务的方法签名中看到，客户端通过发送`Stock`消息来查询服务器。服务器使用`StockQuote`消息发回响应。

**我们使用`pom.xml`文件中定义的`protobuf-maven-plugin`从`stock-quote.proto` IDL 文件**中生成 Java 代码。

该插件在`target/generated-sources/protobuf/java`和`/grpc-java`目录中生成客户端存根和服务器端代码。

**我们将利用生成的代码来实现我们的服务器和客户端**。

## 4.服务器实现

`StockServer`构造函数使用 gRPC `Server`来监听和调度传入的请求:

```java
public class StockServer {
    private int port;
    private io.grpc.Server server;

    public StockServer(int port) throws IOException {
        this.port = port;
        server = ServerBuilder.forPort(port)
          .addService(new StockService())
          .build();
    }
    //...
}
```

我们将`StockService`加到`io.grpc.Server`上。`StockService`扩展了`StockQuoteProviderImplBase`，这是`protobuf`插件从我们的原型文件中生成的。因此， **`StockQuoteProviderImplBase`具有用于三种流服务方法**的存根。

**`StockService`需要覆盖这些存根方法来实际实现我们的服务**。

接下来，我们将看看这是如何在三种流情况下实现的。

### 4.1.服务器端流

**客户发送一个报价请求，得到几个响应，每个响应都有不同的商品报价**:

```java
@Override
public void serverSideStreamingGetListStockQuotes(Stock request, StreamObserver<StockQuote> responseObserver) {
    for (int i = 1; i <= 5; i++) {
        StockQuote stockQuote = StockQuote.newBuilder()
          .setPrice(fetchStockPriceBid(request))
          .setOfferNumber(i)
          .setDescription("Price for stock:" + request.getTickerSymbol())
          .build();
        responseObserver.onNext(stockQuote);
    }
    responseObserver.onCompleted();
}
```

该方法创建一个`StockQuote`，获取价格，并标记报价编号。对于每一个提议，它都向调用`responseObserver::onNext`的客户端发送一条消息。它使用`reponseObserver::onCompleted`来表示它已经完成了 RPC。

### 4.2.客户端流

**客户端发送多只股票，服务器返回一只`StockQuote`** :

```java
@Override
public StreamObserver<Stock> clientSideStreamingGetStatisticsOfStocks(StreamObserver<StockQuote> responseObserver) {
    return new StreamObserver<Stock>() {
        int count;
        double price = 0.0;
        StringBuffer sb = new StringBuffer();

        @Override
        public void onNext(Stock stock) {
            count++;
            price = +fetchStockPriceBid(stock);
            sb.append(":")
                .append(stock.getTickerSymbol());
        }

        @Override
        public void onCompleted() {
            responseObserver.onNext(StockQuote.newBuilder()
                .setPrice(price / count)
                .setDescription("Statistics-" + sb.toString())
                .build());
            responseObserver.onCompleted();
        }

        // handle onError() ...
    };
}
```

该方法获取一个`StreamObserver<StockQuote>`作为参数来响应客户端。它返回一个`StreamObserver<Stock>`，在这里它处理客户端请求消息。

返回的`StreamObserver<Stock>`覆盖了`onNext()`以在每次客户端发送请求时得到通知。

当客户端发送完所有消息时，调用方法`StreamObserver<Stock>.onCompleted()`。利用我们收到的所有`Stock`消息，我们找到获取的股票价格的平均值，创建一个`StockQuote`，并调用`responseObserver::onNext`将结果交付给客户端。

最后，我们覆盖`StreamObserver<Stock>.onError()`来处理异常终止。

### 4.3.双向流

**客户端发送几只股票，服务器为每个请求返回一组价格**:

```java
@Override
public StreamObserver<Stock> bidirectionalStreamingGetListsStockQuotes(StreamObserver<StockQuote> responseObserver) {
    return new StreamObserver<Stock>() {
        @Override
        public void onNext(Stock request) {
            for (int i = 1; i <= 5; i++) {
                StockQuote stockQuote = StockQuote.newBuilder()
                  .setPrice(fetchStockPriceBid(request))
                  .setOfferNumber(i)
                  .setDescription("Price for stock:" + request.getTickerSymbol())
                  .build();
                responseObserver.onNext(stockQuote);
            }
        }

        @Override
        public void onCompleted() {
            responseObserver.onCompleted();
        }

        //handle OnError() ...
    };
}
```

我们有与上一个例子相同的方法签名。改变的是实现:我们不等待客户端发送所有的消息才做出响应。

在这种情况下，我们在接收到每个传入消息后立即调用`responseObserver::onNext`，并且按照接收的顺序。

重要的是要注意，如果需要，我们可以很容易地改变响应的顺序。

## 5.客户端实现

`StockClient`的构造函数接受一个 gRPC 通道，并实例化由 gRPC Maven 插件生成的存根类:

```java
public class StockClient {
    private StockQuoteProviderBlockingStub blockingStub;
    private StockQuoteProviderStub nonBlockingStub;

    public StockClient(Channel channel) {
        blockingStub = StockQuoteProviderGrpc.newBlockingStub(channel);
        nonBlockingStub = StockQuoteProviderGrpc.newStub(channel);
    }
    // ...
}
```

**`StockQuoteProviderBlockingStub``StockQuoteProviderStub`支持同步和异步客户端方法请求**。

接下来，我们将看到三个流 RPC 的客户端实现。

### 5.1.带有服务器端流的客户端 RPC

客户机向服务器发出一个请求股票价格的调用，并返回一个报价列表:

```java
public void serverSideStreamingListOfStockPrices() {
    Stock request = Stock.newBuilder()
      .setTickerSymbol("AU")
      .setCompanyName("Austich")
      .setDescription("server streaming example")
      .build();
    Iterator<StockQuote> stockQuotes;
    try {
        logInfo("REQUEST - ticker symbol {0}", request.getTickerSymbol());
        stockQuotes = blockingStub.serverSideStreamingGetListStockQuotes(request);
        for (int i = 1; stockQuotes.hasNext(); i++) {
            StockQuote stockQuote = stockQuotes.next();
            logInfo("RESPONSE - Price #" + i + ": {0}", stockQuote.getPrice());
        }
    } catch (StatusRuntimeException e) {
        logInfo("RPC failed: {0}", e.getStatus());
    }
}
```

我们使用`blockingStub::serverSideStreamingGetListStock` s 来发出同步请求。我们用`Iterator`得到一个`StockQuotes`的列表。

### 5.2.具有客户端流的客户端 RPC

客户端向服务器发送一个`Stock`流，并返回一个带有一些统计信息的`StockQuote`:

```java
public void clientSideStreamingGetStatisticsOfStocks() throws InterruptedException {
    StreamObserver<StockQuote> responseObserver = new StreamObserver<StockQuote>() {
        @Override
        public void onNext(StockQuote summary) {
            logInfo("RESPONSE, got stock statistics - Average Price: {0}, description: {1}", summary.getPrice(), summary.getDescription());
        }

        @Override
        public void onCompleted() {
            logInfo("Finished clientSideStreamingGetStatisticsOfStocks");
        }

        // Override OnError ...
    };

    StreamObserver<Stock> requestObserver = nonBlockingStub.clientSideStreamingGetStatisticsOfStocks(responseObserver);
    try {
        for (Stock stock : stocks) {
            logInfo("REQUEST: {0}, {1}", stock.getTickerSymbol(), stock.getCompanyName());
            requestObserver.onNext(stock);
        }
    } catch (RuntimeException e) {
        requestObserver.onError(e);
        throw e;
    }
    requestObserver.onCompleted();
}
```

正如我们在服务器示例中所做的那样，我们使用`StreamObservers`来发送和接收消息。

`requestObserver`使用非阻塞存根将`Stock`的列表发送到服务器。

对于`responseObserver`，我们用一些统计数据取回`StockQuote`。

### 5.3.具有双向流的客户端 RPC

客户端发送一个`Stock`流，并返回每个`Stock`的价格列表。

```java
public void bidirectionalStreamingGetListsStockQuotes() throws InterruptedException{
    StreamObserver<StockQuote> responseObserver = new StreamObserver<StockQuote>() {
        @Override
        public void onNext(StockQuote stockQuote) {
            logInfo("RESPONSE price#{0} : {1}, description:{2}", stockQuote.getOfferNumber(), stockQuote.getPrice(), stockQuote.getDescription());
        }

        @Override
        public void onCompleted() {
            logInfo("Finished bidirectionalStreamingGetListsStockQuotes");
        }

        //Override onError() ...
    };

    StreamObserver<Stock> requestObserver = nonBlockingStub.bidirectionalStreamingGetListsStockQuotes(responseObserver);
    try {
        for (Stock stock : stocks) {
            logInfo("REQUEST: {0}, {1}", stock.getTickerSymbol(), stock.getCompanyName());
            requestObserver.onNext(stock);
            Thread.sleep(200);
        }
    } catch (RuntimeException e) {
        requestObserver.onError(e);
        throw e;
    }
    requestObserver.onCompleted();
}
```

该实现非常类似于客户端流的情况。我们用`requestObserver`发送`Stock`——唯一的不同是，现在我们用`responseObserver`得到多个响应。响应与请求是分离的——它们可以以任何顺序到达。

## 6.运行服务器和客户端

使用 Maven 编译完代码后，我们只需要打开两个命令窗口。

要运行服务器，请执行以下操作:

```java
mvn exec:java -Dexec.mainClass=com.baeldung.grpc.streaming.StockServer
```

要运行客户端:

```java
mvn exec:java -Dexec.mainClass=com.baeldung.grpc.streaming.StockClient
```

## 7.结论

在本文中，我们看到了如何在 gRPC 中使用流。流是一项强大的功能，它允许客户端和服务器通过在单个连接上发送多条消息来进行通信。此外，**消息的接收顺序与发送顺序相同，但是任何一方都可以按照自己希望的顺序读取或写入消息**。

这些例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221205160713/https://github.com/eugenp/tutorials/tree/master/grpc)