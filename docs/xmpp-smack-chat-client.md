# XMPP Smack 客户端指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/xmpp-smack-chat-client>

## 1。简介

XMPP 是一个丰富而复杂的即时通讯协议。

在本教程中，我们不是从头开始编写我们自己的客户端，而是看一看 Smack，一个用 Java 编写的模块化和可移植的开源 XMPP 客户端，它已经为我们做了很多繁重的工作。

## 2。依赖性

Smack 被组织成几个模块以提供更多的灵活性，所以我们可以很容易地包含我们需要的特性。

其中包括:

*   TCP 模块上的 XMPP
*   支持 XMPP 标准基金会定义的许多扩展的模块
*   传统扩展支持
*   要调试的模块

我们可以在 [XMPP 的文档](https://web.archive.org/web/20220523233813/https://download.igniterealtime.org/smack/docs/latest/javadoc/)中找到所有支持的模块。

然而，在本教程中，我们将只使用`tcp`、`im`、`extensions`和`java7`模块:

```java
<dependency>
    <groupId>org.igniterealtime.smack</groupId>
    <artifactId>smack-tcp</artifactId>
</dependency>
<dependency>
    <groupId>org.igniterealtime.smack</groupId>
    <artifactId>smack-im</artifactId>
</dependency>
<dependency>
    <groupId>org.igniterealtime.smack</groupId>
    <artifactId>smack-extensions</artifactId>
</dependency>
<dependency>
    <groupId>org.igniterealtime.smack</groupId>
    <artifactId>smack-java7</artifactId>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20220523233813/https://search.maven.org/search?q=g:org.igniterealtime.smack) 找到。

## 3。设置

为了测试客户端，我们需要一个 XMPP 服务器。为此，我们将在[jabber.hot-chilli.net](https://web.archive.org/web/20220523233813/https://jabber.hot-chilli.net/)上创建一个账户，这是一个面向所有人的免费 Jabber/XMPP 服务。

之后，我们可以使用`XMPPTCPConnectionConfiguration `类配置 Smack，该类提供了一个构建器来设置连接的参数:

```java
XMPPTCPConnectionConfiguration config = XMPPTCPConnectionConfiguration.builder()
  .setUsernameAndPassword("baeldung","baeldung")
  .setXmppDomain("jabb3r.org")
  .setHost("jabb3r.org")
  .build();
```

**生成器允许我们设置执行连接**所需的基本信息。如果需要，我们还可以设置其他参数，如端口、SSL 协议和超时。

## 4。连接

使用`XMPPTCPConnection`类可以简单地实现连接:

```java
AbstractXMPPConnection connection = new XMPPTCPConnection(config);
connection.connect(); //Establishes a connection to the server
connection.login(); //Logs in 
```

该类包含一个接受以前构建的配置的构造函数。它还提供了连接到服务器和登录的方法。

**一旦建立了连接，我们就可以使用 Smack 的特性**，就像`chat`，我们将在下一节描述。

在连接突然中断的情况下，默认情况下，Smack 会尝试重新连接。

`[ReconnectionManager](https://web.archive.org/web/20220523233813/https://download.igniterealtime.org/smack/docs/latest/javadoc/org/jivesoftware/smack/ReconnectionManager.html)`将尝试立即重新连接到服务器，并随着连续的重新连接不断失败而增加尝试之间的延迟。

## 5。聊天

该库的一大特色是支持聊天。

使用`Chat`类可以在两个用户之间创建新的消息线程:

```java
ChatManager chatManager = ChatManager.getInstanceFor(connection);
EntityBareJid jid = JidCreate.entityBareFrom("[[email protected]](/web/20220523233813/https://www.baeldung.com/cdn-cgi/l/email-protection)");
Chat chat = chatManager.chatWith(jid);
```

注意，为了构建一个`Chat`,我们使用了一个`ChatManager`,并且很明显，指定了与谁聊天。我们通过使用`EntityBareJid`对象实现了后者，该对象 **包装了一个 XMPP 地址—也称为 JID—** ，由本地部分(`baeldung2`)和域部分(`jabb3r.org`)组成。

之后，我们可以使用`send()`方法发送消息:

```java
chat.send("Hello!");
```

并通过设置侦听器来接收消息:

```java
chatManager.addIncomingListener(new IncomingChatMessageListener() {
  @Override
  public void newIncomingMessage(EntityBareJid from, Message message, Chat chat) {
      System.out.println("New message from " + from + ": " + message.getBody());
  }
});
```

### 5.1。房间

除了端到端的用户聊天， **Smack 还通过使用房间**来提供对群组聊天的支持。

有两种类型的房间，即时房间和预订房间。

即时聊天室可用于即时访问，并根据一些默认配置自动创建。另一方面，在允许任何人进入之前，预订的房间由房间所有者手动配置。

让我们来看看如何使用`MultiUserChatManager`创建一个即时房间:

```java
MultiUserChatManager manager = MultiUserChatManager.getInstanceFor(connection);
MultiUserChat muc = manager.getMultiUserChat(jid);
Resourcepart room = Resourcepart.from("baeldung_room");
muc.create(room).makeInstant();
```

以类似的方式，我们可以创建一个预订房间:

```java
Set<Jid> owners = JidUtil.jidSetFrom(
  new String[] { "[[email protected]](/web/20220523233813/https://www.baeldung.com/cdn-cgi/l/email-protection)", "[[email protected]](/web/20220523233813/https://www.baeldung.com/cdn-cgi/l/email-protection)" });

muc.create(room)
  .getConfigFormManger()
  .setRoomOwners(owners)
  .submitConfigurationForm();
```

## 6。花名册

Smack 提供的另一个功能是跟踪其他用户的存在。

使用`Roster.getInstanceFor(), `我们可以获得一个`Roster`实例:

```java
Roster roster = Roster.getInstanceFor(connection);
```

**[`Roster`](https://web.archive.org/web/20220523233813/https://github.com/igniterealtime/Smack/blob/master/documentation/roster.md)是一个联系人列表，将用户表示为`RosterEntry`对象，并允许我们将用户组织成组。**

我们可以使用`getEntries()`方法打印`Roster`中的所有条目:

```java
Collection<RosterEntry> entries = roster.getEntries();
for (RosterEntry entry : entries) {
    System.out.println(entry);
}
```

此外，它允许我们通过`RosterListener:`监听其条目和存在数据的变化

```java
roster.addRosterListener(new RosterListener() {
    public void entriesAdded(Collection<String> addresses) { // handle new entries }
    public void entriesDeleted(Collection<String> addresses) { // handle deleted entries }
    public void entriesUpdated(Collection<String> addresses) { // handle updated entries }
    public void presenceChanged(Presence presence) { // handle presence change }
});
```

它还提供了一种保护用户隐私的方法，确保只有经过批准的用户才能订阅名册。为了做到这一点，Smack 实现了一个基于权限的模型。

使用`Roster.setSubscriptionMode()`方法有三种处理在线订阅请求的方式:

*   `Roster.SubscriptionMode.accept_all`–接受所有订阅请求
*   `Roster.SubscriptionMode.reject_all – `拒绝所有订阅请求
*   `Roster.SubscriptionMode.manual – `手动处理在线订阅请求

如果我们选择手动处理订阅请求，我们将需要注册一个`StanzaListener` (在下一节中描述)并处理带有`Presence.Type.subscribe`类型的数据包。

## 7。房间

除了聊天之外，Smack 还提供了一个灵活的框架来发送一个节并监听传入的节。

澄清一下，在 XMPP 中，节是一个独立的语义单位。它是通过 XML 流从一个实体发送到另一个实体的结构化信息。

我们可以使用`send()`方法通过`Connection `传输`Stanza`:

```java
Stanza presence = new Presence(Presence.Type.subscribe);
connection.sendStanza(presence);
```

在上面的例子中，我们发送了一个`Presence`节来订阅花名册。

另一方面，为了处理传入的节，库提供了两个构造:

*   `StanzaCollector `
*   `StanzaListener`

特别是 **`StanzaCollector `让我们同步等待新的一节**:

```java
StanzaCollector collector
  = connection.createStanzaCollector(StanzaTypeFilter.MESSAGE);
Stanza stanza = collector.nextResult();
```

而 **`StanzaListener`是一个异步通知我们即将到来的段落**的接口:

```java
connection.addAsyncStanzaListener(new StanzaListener() {
    public void processStanza(Stanza stanza) 
      throws SmackException.NotConnectedException,InterruptedException, 
        SmackException.NotLoggedInException {
            // handle stanza
        }
}, StanzaTypeFilter.MESSAGE);
```

### 7.1。过滤器

此外，这个库提供了一组内置的过滤器来处理输入的节。

我们可以使用`StanzaTypeFilter`通过类型或者使用`StanzaIdFilter:`通过 ID 来过滤节

```java
StanzaFilter messageFilter = StanzaTypeFilter.MESSAGE;
StanzaFilter idFilter = new StanzaIdFilter("123456");
```

或者，通过特定地址识别:

```java
StanzaFilter fromFilter
  = FromMatchesFilter.create(JidCreate.from("[[email protected]](/web/20220523233813/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
StanzaFilter toFilter
  = ToMatchesFilter.create(JidCreate.from("[[email protected]](/web/20220523233813/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
```

我们可以使用逻辑过滤运算符(`AndFilter`、`OrFilter`、`NotFilter`)来创建复杂的过滤器:

```java
StanzaFilter filter
  = new AndFilter(StanzaTypeFilter.Message, FromMatchesFilter.create("[[email protected]](/web/20220523233813/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
```

## 8。结论

在本文中，我们介绍了 Smack 提供的最有用的类。

我们学习了如何配置库来发送和接收 XMPP 节。

随后，我们学习了如何使用`ChatManager`和`Roster`功能处理群聊。

像往常一样，本教程中显示的所有代码示例都可以在 GitHub 上获得。