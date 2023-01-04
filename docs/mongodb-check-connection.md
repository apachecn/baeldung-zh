# 正在检查与 MongoDB 的连接

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-check-connection>

## 1.概观

在本教程中，我们将学习检查与 [MongoDB](https://web.archive.org/web/20220814143201/https://www.mongodb.com/) 的连接。

重要的是，要连接到单个 MongoDB 实例，我们需要指定 MongoDB 实例的 URI。

## 2.使用 Mongo Shell 检查连接

在这一节中，我们将使用 mongo shell 命令连接到 MongoDB 服务器。我们将探索连接到 MongoDB 的不同情况。

### 2.1.检查默认端口上的连接

默认情况下，MongoDB 运行在端口`27017,` 上，但是我们也可以在其他端口上运行它。我们可以使用简单的 mongo 命令连接到 MongoDB 服务器:

```java
$ mongo
MongoDB shell version v4.4.2
connecting to: mongodb://localhost:27017/?compressors=disabled&gssapiServiceName;=mongodb
Implicit session: session { "id" : UUID("b7f80a0c-c7b9-4aea-b34c-605b85e601dd") }
MongoDB server version: 4.0.1-rc0-2-g54f1582fc6
```

在上面的命令中，默认情况下，MongoDB 假定端口为`27017`。如果 MongoDB 服务器关闭，我们会得到以下错误:

```java
$ mongo --host localhost --port 27017 admin
MongoDB shell version v4.4.2
connecting to: mongodb://localhost:27017/admin?compressors=disabled&gssapiServiceName;=mongodb
Error: couldn't connect to server localhost:27017, connection attempt failed:
  SocketException: Error connecting to localhost:27017 :: caused by :: Connection refused :
[[email protected]](/web/20220814143201/https://www.baeldung.com/cdn-cgi/l/email-protection)/mongo/shell/mongo.js:374:17
@(connect):2:6
exception: connect failed
exiting with code 1
```

在这种情况下，我们得到了错误，因为我们无法连接到服务器。

### 2.2.在安全的 MongoDB 数据库上检查连接

MongoDB 可以通过身份验证来保护。在这种情况下，我们需要在命令中传递用户名和密码:

```java
$ mongo mongodb://baeldung:[[email protected]](/web/20220814143201/https://www.baeldung.com/cdn-cgi/l/email-protection):27017
```

这里我们使用了`username`“bael dung”和`password`“bael dung”来连接运行在本地主机上的 MongoDB。

### 2.3.检查自定义端口上的连接

我们还可以在自定义端口上运行 MongoDB。我们需要做的就是在`mongod.conf`文件中进行修改。如果 MongoDB 运行在其他端口上，我们需要在命令中提供该端口:

```java
$ mongo mongodb://localhost:27001
```

这里，在 mongo shell 中，我们还可以检查数据库服务器的当前活动连接。

```java
var status = db.serverStatus();
status.connections
{
    "current" : 21,
    "available" : 15979
}
```

`serverStatus`返回一个概述数据库进程当前状态的文档。定期运行`serverStatus`命令将收集关于 MongoDB 实例的统计信息。

## 3.使用 Java 驱动程序代码检查连接

到目前为止，我们已经学会了使用 shell 检查与 MongoDB 的连接。现在让我们使用 Java 驱动程序代码来研究同样的问题:

```java
MongoClientOptions.Builder builder = MongoClientOptions.builder();
// builder settings
ServerAddress ServerAddress = new ServerAddress("localhost", 27017);
MongoClient mongoClient = new MongoClient(ServerAddress, builder.build());

try {
    System.out.println("MongoDB Server is Up:- "+mongoClient.getAddress());
    System.out.println(mongoClient.getConnectPoint());
    System.out.println(db.getStats());
} catch (Exception e) {
    System.out.println("MongoDB Server is Down");
} finally{
    mongoClient.close();
}
```

在上面的代码中，我们首先创建了`MongoClientOption`构建器来定制`MongoClient`连接的配置，然后使用服务器地址创建了`MongoClient`连接。假设 MongoDB 服务器运行在本地主机的`27017`端口上。否则，`MongoClient `会抛出一个错误。

## 4.结论

在本教程中，我们学习了在不同的实时情况下检查 MongoDB 服务器的连接。

首先，我们用 mongo 默认命令检查连接，然后使用 authenticated 命令并连接到运行在定制端口上的 MongoDB 服务器。接下来，我们检查了 MongoDB shell 和 Java 驱动程序代码的连接。

所有案例的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220814143201/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-2)