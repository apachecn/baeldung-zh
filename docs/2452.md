# Java 中的 MQTT 客户端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mqtt-client>

## 1.概观

在本教程中，我们将看到如何使用 [Eclipse Paho 项目](https://web.archive.org/web/20220524010205/https://www.eclipse.org/paho/)提供的库在 Java 项目中添加 MQTT 消息传递。

## 2.MQTT 引物

**MQTT (MQ 遥测传输)是一种消息协议**,旨在提供一种简单、轻量的方法来传输低功耗设备的数据，例如工业应用中使用的设备。

随着 IoT(物联网)设备的日益普及，MQTT 的使用也越来越多，导致其被 OASIS 和 ISO 标准化。

该协议支持单一的消息传递模式，即发布-订阅模式:客户端发送的每条消息都包含一个关联的“主题”,代理使用该主题将其路由到订阅的客户端。主题名称可以是简单的字符串，如“`oiltemp`”或类似路径的字符串“`motor/1/rpm`”。

为了接收消息，客户端使用一个或多个主题的确切名称或包含支持的通配符之一的字符串(对于多级主题为“#”，对于单级主题为“+”)来订阅这些主题。

## 3.项目设置

为了在 Maven 项目中包含 Paho 库，我们必须添加以下依赖项:

```
<dependency>
  <groupId>org.eclipse.paho</groupId>
  <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
  <version>1.2.0</version>
</dependency>
```

可以从 Maven Central 下载最新版本的 [Eclipse Paho](https://web.archive.org/web/20220524010205/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.eclipse.paho%22%20AND%20a%3A%22org.eclipse.paho.client.mqttv3%22) Java 库模块。

## 4.客户端设置

当使用 Paho 库时，为了从 MQTT 代理发送和/或接收消息，我们需要做的第一件事是**获得`IMqttClient`接口** `. `的实现。该接口包含应用程序建立到服务器的连接、发送和接收消息所需的所有方法。

Paho 自带了该接口的两个实现，一个是异步的(`MqttAsyncClient`)，另一个是同步的(`MqttClient`)。在我们的例子中，我们将关注同步版本，它具有更简单的语义。

设置本身是一个两步过程:我们首先创建一个`MqttClient`类的实例，然后将它连接到我们的服务器。以下小节详细介绍了这些步骤。

### 4.1.创建新的`IMqttClient`实例

下面的代码片段展示了如何创建一个新的`IMqttClient`同步实例:

```
String publisherId = UUID.randomUUID().toString();
IMqttClient publisher = new MqttClient("tcp://iot.eclipse.org:1883",publisherId);
```

在本例中，**我们使用最简单的构造函数，它接受 MQTT 代理的端点地址和唯一标识我们的客户机的客户机标识符**。

在我们的例子中，我们使用了一个随机 UUID，所以每次运行都会生成一个新的客户端标识符。

Paho 还提供了额外的构造函数，我们可以使用它们来定制用于存储未确认消息的持久性机制和/或用于运行协议引擎实现所需的后台任务的`ScheduledExecutorService`。

**我们使用的服务器端点是由 Paho 项目**托管的公共 MQTT 代理，它允许任何有互联网连接的人测试客户机，而不需要任何认证。

### 4.2.连接到服务器

我们新创建的`MqttClient`实例没有连接到服务器。**我们通过调用它的`connect()`方法**，可选地传递一个`MqttConnectOptions `实例来定制协议的某些方面。

特别是，我们可以使用这些选项来传递附加信息，如安全凭证、会话恢复模式、重新连接模式等等。

`MqttConnectionOptions` 类将这些选项公开为简单的属性，我们可以使用普通的 setter 方法进行设置。我们只需要设置我们的场景所需的属性，其余的将采用默认值。

用于建立与服务器连接的代码通常如下所示:

```
MqttConnectOptions options = new MqttConnectOptions();
options.setAutomaticReconnect(true);
options.setCleanSession(true);
options.setConnectionTimeout(10);
publisher.connect(options);
```

在这里，我们定义我们的连接选项，以便:

*   如果网络出现故障，库将自动尝试重新连接到服务器
*   它将丢弃上次运行中未发送的消息
*   连接超时设置为 10 秒

## 5.发送消息

使用已经连接的`MqttClient`发送消息非常简单。**我们使用`publish()`方法的一个变体将有效载荷(总是一个字节数组)发送到给定的主题**，使用以下服务质量选项之一:

*   0—“最多一次”语义，也称为“一劳永逸”。当消息丢失可接受时，使用此选项，因为它不需要任何类型的确认或持久性
*   1-“至少一次”语义。当消息丢失不可接受时使用此选项`and`您的订户可以处理重复消息
*   2—“恰好一次”语义。当消息丢失不可接受时使用此选项`and`您的订户无法处理重复消息

在我们的示例项目中，`EngineTemperatureSensor `类扮演模拟传感器的角色，每次我们调用它的`call()`方法时，它都会产生一个新的温度读数。

这个类实现了`Callable`接口，所以我们可以很容易地将它与`java.util.concurrent`包中可用的`ExecutorService`实现之一一起使用:

```
public class EngineTemperatureSensor implements Callable<Void> {

    // ... private members omitted

    public EngineTemperatureSensor(IMqttClient client) {
        this.client = client;
    }

    @Override
    public Void call() throws Exception {        
        if ( !client.isConnected()) {
            return null;
        }           
        MqttMessage msg = readEngineTemp();
        msg.setQos(0);
        msg.setRetained(true);
        client.publish(TOPIC,msg);        
        return null;        
    }

    private MqttMessage readEngineTemp() {             
        double temp =  80 + rnd.nextDouble() * 20.0;        
        byte[] payload = String.format("T:%04.2f",temp)
          .getBytes();        
        return new MqttMessage(payload);           
    }
}
```

**`MqttMessage`封装了有效载荷本身、请求的服务质量以及消息的`retained`标志。**该标志向代理指示它应该保留该消息，直到被订户使用。

我们可以使用这个特性来实现一个“最近一次正确的”行为，这样当一个新的订阅者连接到服务器时，它会立即收到保留的消息。

## 6.接收消息

为了接收来自 MQTT 代理的消息，**我们需要使用`subscribe()`方法变体之一**，它允许我们指定:

*   我们希望接收的消息的一个或多个主题过滤器
*   相关联的 QoS
*   处理收到的消息的回调处理程序

在下面的例子中，我们展示了如何将消息监听器添加到现有的`IMqttClient` 实例中，以接收来自给定主题的消息。我们使用一个`CountDownLatch`作为回调和主执行线程之间的同步机制，每当新消息到达时，它就递减。

在示例代码中，我们使用了不同的`IMqttClient`实例来接收消息。我们这样做只是为了更清楚地说明哪个客户端做什么，但这不是 Paho 的限制-如果您愿意，您可以使用同一个客户端来发布和接收消息:

```
CountDownLatch receivedSignal = new CountDownLatch(10);
subscriber.subscribe(EngineTemperatureSensor.TOPIC, (topic, msg) -> {
    byte[] payload = msg.getPayload();
    // ... payload handling omitted
    receivedSignal.countDown();
});    
receivedSignal.await(1, TimeUnit.MINUTES);
```

上面使用的`subscribe()`变量将一个`IMqttMessageListener`实例作为它的第二个参数。

在我们的例子中，我们使用一个简单的 lambda 函数来处理有效负载并递减计数器。如果在指定的时间窗口(1 分钟)内没有足够的消息到达，`await()`方法将抛出一个异常。

当使用 Paho 时，我们不需要显式地确认消息收到。如果回调正常返回，Paho 认为这是一次成功的消费，并向服务器发送一个确认。

如果回调抛出一个`Exception`，客户端将被关闭。**请注意，这将导致 QoS 等级为 0** 的任何消息丢失。

一旦客户端重新连接并再次订阅主题，以 QoS 级别 1 或 2 发送的消息将由服务器重新发送。

## 7.结论

在本文中，我们展示了如何使用 Eclipse Paho 项目提供的库在 Java 应用程序中添加对 MQTT 协议的支持。

这个库处理所有底层协议细节，允许我们关注解决方案的其他方面，同时为定制其内部特性的重要方面(如消息持久性)留下良好的空间。

本文中显示的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220524010205/https://github.com/eugenp/tutorials/tree/master/libraries-server)