# REST vs. gRPC

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-vs-grpc>

## 1.概观

在本文中，我们将比较 REST 和 gRPC 这两种 web APIs 的架构风格。

## 2.什么是休息？

REST(表述性状态转移)是一种架构风格，为设计 web APIs 提供了指南。

**它使用标准的 HTTP 1.1 方法，如`GET`、`POST`、`PUT`和`DELETE`来处理服务器端资源**。此外，[REST API 提供了预定义的 URL](/web/20220617075715/https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration)，客户端必须使用这些 URL 来连接服务器。

## 3\. What Is gRPC?

[gRPC (Remote Procedure Call)是 Google 利用](/web/20220617075715/https://www.baeldung.com/grpc-introduction) [HTTP/2 协议](/web/20220617075715/https://www.baeldung.com/netty-http2)开发的一种开源数据交换技术。

**使用[协议缓冲二进制格式(Protobuf)进行数据交换](/web/20220617075715/https://www.baeldung.com/spring-rest-api-with-protocol-buffers#1-introduction-to-protocol-buffers)** 。此外，这种架构风格实施了开发人员在开发或使用 web APIs 时必须遵循的规则。

## 4\. REST vs. gRPC

### 4.1.准则与规则

REST 是一套设计 web APIs 的指导原则，不需要强制任何东西。另一方面， **gRPC 通过定义一个客户端和服务器都必须遵守的`.proto`文件来执行规则，以便进行数据交换**。

### 4.2.底层 HTTP 协议

REST 提供了一个基于 HTTP 1.1 协议的请求-响应通信模型。因此，当多个请求到达服务器时，它必须一次处理一个请求。

然而，gRPC 遵循客户端响应通信模型来设计依赖于 HTTP/2 的 web APIs。因此， **gRPC 允许流式通信并同时服务多个请求**。除此之外， **gRPC 还支持类似 REST** 的一元通信。

### 4.3.数据交换格式

REST 通常使用 JSON 和 XML 格式进行数据传输。然而，gRPC 依赖 Protobuf 通过 HTTP/2 协议进行数据交换。

### 4.4.序列化与强类型

在大多数情况下，REST 使用 [JSON 或](/web/20220617075715/https://www.baeldung.com/java-json) [XML，这需要序列化](/web/20220617075715/https://www.baeldung.com/jackson-xml-serialization-and-deserialization)并转换成客户机和服务器的目标编程语言，从而增加了响应时间和解析请求/响应时出错的可能性。

但是，gRPC 提供了使用 Protobuf 交换格式自动转换为所选编程语言的强类型消息。

### 4.5.潜伏

利用 HTTP 1.1 的 REST 要求每个请求都有一个 TCP 握手。因此，HTTP 1.1 中的 REST APIs 可能会遇到延迟问题。

另一方面，gRPC 依赖于使用多路复用流的 HTTP/2 协议。因此，几个客户端可以同时发送多个请求，而无需为每个客户端建立新的 TCP 连接。此外，服务器可以通过已建立的连接向客户端发送推送通知。

### 4.6.浏览器支持

HTTP 1.1 上的 REST APIs 具有通用浏览器支持。

然而，gRPC 对浏览器的支持有限，因为许多浏览器(通常是旧版本)对 HTTP/2 没有成熟的支持。因此，它可能需要 gRPC-web 和代理层来执行 HTTP 1.1 和 HTTP/2 之间的转换。因此，目前 gRPC 主要用于内部服务。

### 4.7.**代码生成功能**

REST 没有提供内置的代码生成特性。但是，我们可以使用像 Swagger 或 Postman 这样的第三方工具来为 API 请求生成代码。

另一方面， **[gRPC，使用其`protoc`编译器](/web/20220617075715/https://www.baeldung.com/grpc-introduction#1-using-protocol-buffer-compiler)，自带本机代码生成特性**，兼容多种编程语言。

## 5.结论

在本文中，我们比较了 API 的两种架构风格，REST 和 gRPC。

我们的结论是 **REST 在将微服务和第三方应用与核心系统**集成时非常方便。

然而，gRPC 可以在各种系统中找到它的应用，如需要轻量级消息传输的**物联网系统、没有浏览器支持的移动应用以及需要多路复用流的应用**。