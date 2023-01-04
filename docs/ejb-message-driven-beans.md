# EJB 消息驱动 Beans 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ejb-message-driven-beans>

## 1。简介

简单地说，企业 JavaBean (EJB)是运行在应用服务器上的 JEE 组件。

在本教程中，我们将讨论消息驱动 bean(MDB ),它负责在异步上下文中处理消息。

自 EJB 2.0 规范以来，MDB 是 JEE 的一部分；EJB 3.0 引入了注释的使用，使得创建这些对象变得更加容易。在这里，我们将重点关注注释。

## 2。一些背景

在我们深入到消息驱动 Beans 的细节之前，让我们回顾一下与消息传递相关的一些概念。

### 2.1。消息传递

消息传递是一种通信机制。通过使用消息传递，程序可以交换数据，即使它们是用不同的编程语言编写的或者驻留在不同的操作系统中。

它提供了一个松散耦合的解决方案；**信息的生产者和消费者都不需要知道彼此的细节**。

因此，它们甚至不必同时连接到消息传递系统(异步通信)。

### 2.2。同步和异步通信

在同步通信期间，请求者一直等到响应返回。同时，请求者进程保持阻塞状态。

另一方面，在异步通信中，请求者发起操作，但不会被操作阻塞；请求者可以继续执行其他任务，稍后接收响应。

### 2.3。JMS

Java 消息服务(“JMS”)是一个支持消息传递的 Java API。

JMS 提供对等和发布/订阅消息传递模型。

## 3。消息驱动 bean

MDB 是每当消息到达消息传递系统时由容器调用的组件。因此，这个事件触发了这个 bean 内部的代码。

我们可以在 MDB `onMessage()`方法中执行许多任务，因为要在浏览器上显示接收到的数据，或者解析并保存到数据库中。

另一个例子是在一些处理之后将数据提交给另一个队列。这完全取决于我们的商业规则。

### 3.1。消息驱动 Beans 生命周期

MDB 只有两种状态:

1.  集装箱上没有
2.  已创建并准备好接收消息

依赖项(如果存在的话)会在 MDB 创建后立即注入。

为了在接收消息之前执行指令，我们需要用`@javax.ejb.` `PostConstruct`注释一个方法。

依赖注入和`@javax.ejb.` `PostConstruct`执行都只发生一次。

之后，MDB 就可以接收消息了。

### 3.2。交易

消息可以被传递到事务上下文中的 MDB。

这意味着`onMessage()`方法中的所有操作都是单个事务的一部分。

因此，如果发生回滚，消息系统会重新传递数据。

## 4。使用消息驱动 bean

### 4.1。创建消费者

为了创建消息驱动 Bean，我们在类名声明之前使用了`@javax.ejb.MessageDriven`注释。

为了处理传入的消息，我们必须实现`MessageListener`接口的`onMessage()`方法:

```java
@MessageDriven(activationConfig = { 
    @ActivationConfigProperty(
      propertyName = "destination", 
      propertyValue = "tutorialQueue"), 
    @ActivationConfigProperty(
      propertyName = "destinationType", 
      propertyValue = "javax.jms.Queue")
})
public class ReadMessageMDB implements MessageListener {

    public void onMessage(Message message) {
        TextMessage textMessage = (TextMessage) message;
        try {
            System.out.println("Message received: " + textMessage.getText());
        } catch (JMSException e) {
            System.out.println(
              "Error while trying to consume messages: " + e.getMessage());
        }
    }
}
```

因为本文关注的是注释而不是。xml 描述符我们将使用`@ActivationConfigProperty` 而不是<激活-配置-属性> `.`

`@ActivationConfigProperty` 是代表该配置的键值属性。我们将在`activationConfig`中使用两个属性，设置 MDB 将使用的队列和对象类型。

在`onMessage()`方法中，我们可以将消息参数转换为`TextMessage, BytesMessage, MapMessage StreamMessage` 或 `ObjectMessage`。

然而，对于本文，我们将只查看标准输出的消息内容。

### 4.2。创建生产者

如 2.1 节所述，**生产者和消费者服务是完全独立的，甚至可以用不同的编程语言编写**！

我们将使用 Java Servlets 生成消息:

```java
@Override
protected void doGet(
  HttpServletRequest req, 
  HttpServletResponse res) 
  throws ServletException, IOException {

    String text = req.getParameter("text") != null ? req.getParameter("text") : "Hello World";

    try (
        Context ic = new InitialContext();

        ConnectionFactory cf = (ConnectionFactory) ic.lookup("/ConnectionFactory");
        Queue queue = (Queue) ic.lookup("queue/tutorialQueue");

        Connection connection = cf.createConnection();
    ) {
        Session session = connection.createSession(
          false, Session.AUTO_ACKNOWLEDGE);
        MessageProducer publisher = session
          .createProducer(queue);

        connection.start();

        TextMessage message = session.createTextMessage(text);
        publisher.send(message);

    } catch (NamingException | JMSException e) {
        res.getWriter()
          .println("Error while trying to send <" + text + "> message: " + e.getMessage());
    } 

    res.getWriter()
      .println("Message sent: " + text);
}
```

获得`ConnectionFactory` 和`Queue` 实例后，我们必须创建一个`Connection`和`Session`。

为了创建一个会话，我们调用`createSession`方法。

`createSession`中的第一个参数是`boolean`，它定义了会话是否是事务的一部分。

第二个参数仅在第一个参数为`false`时使用。它允许我们描述适用于传入消息的确认方法，并接受`Session.AUTO_ACKNOWLEDGE, Session.CLIENT_ACKNOWLEDGE`和`Session.DUPS_OK_ACKNOWLEDGE`的值。

我们现在可以开始连接，在会话对象上创建一个文本消息并发送我们的消息。

绑定到同一队列的使用者将接收消息并执行其异步任务。

此外，除了查找`JNDI`对象之外，我们的 try-with-resources 块中的所有操作都确保在`JMSException`遇到错误时关闭连接，比如尝试连接到不存在的队列或指定错误的端口号进行连接。

## 5。测试消息驱动 Bean

通过`SendMessageServlet`上的`GET`方法发送消息，如:

`http://127.0.0.1:8080/producer/SendMessageServlet?text=Text to send`

同样，如果我们不发送任何参数，servlet 会将`“Hello World”`发送到队列，如`http://127.0.0.1:8080/producer/SendMessageServlet.`所示

## 6。结论

消息驱动 Beans 允许简单地创建基于队列的应用程序。

因此，**MDB 允许我们将我们的应用解耦到具有本地化职责的更小的服务中**，允许一个更加模块化和增量的系统，该系统可以从系统故障中恢复。

像往常一样，代码在 GitHub 上[结束。](https://web.archive.org/web/20220926190729/https://github.com/eugenp/tutorials/tree/master/spring-ejb-modules/wildfly/wildfly-mdb)