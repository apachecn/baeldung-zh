# Spring 集成中的安全性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-integration-security>

## 1。简介

在本文中，我们将关注如何在集成流程中一起使用 Spring 集成和 Spring 安全。

因此，我们将建立一个简单的安全消息流来演示 Spring 安全在 Spring 集成中的使用。此外，我们将提供多线程消息通道中的`SecurityContext`传播的例子。

关于使用框架的更多细节，可以参考我们的[Spring Integration 简介](/web/20220627074627/https://www.baeldung.com/spring-integration)。

## 2。弹簧集成配置

### 2.1。依赖性

首先，我们需要将 Spring 集成依赖项添加到我们的项目中。

因为我们将建立一个简单的消息流，包含`DirectChannel`、`PublishSubscribeChannel`和`ServiceActivator,` ，所以我们需要`spring-integration-core`依赖。

此外，我们还需要`spring-integration-security`依赖项，以便能够在 Spring 集成中使用 Spring 安全性:

```java
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-security</artifactId>
    <version>5.0.3.RELEASE</version>
</dependency>
```

我们还使用了 Spring Security，所以我们将把`spring-security-config`添加到我们的项目中:

```java
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.0.3.RELEASE</version>
</dependency>
```

我们可以在 Maven Central 查看以上所有依赖项的最新版本: `[spring-integration-security](https://web.archive.org/web/20220627074627/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.integration%22%20AND%20a%3A%22spring-integration-security%22), [spring-security-config](https://web.archive.org/web/20220627074627/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.security%22%20AND%20a%3A%22spring-security-config%22).`

### 2.2。基于 Java 的配置

我们的例子将使用基本的 Spring 集成组件。因此，我们只需要通过使用`@EnableIntegration`注释在我们的项目中启用 Spring 集成:

```java
@Configuration
@EnableIntegration
public class SecuredDirectChannel {
    //...
}
```

## 3。安全消息通道

首先，**我们需要一个`ChannelSecurityInterceptor`的实例，它将拦截一个通道上的所有`send`和`receive`调用，并决定该调用是否可以被执行或拒绝**:

```java
@Autowired
@Bean
public ChannelSecurityInterceptor channelSecurityInterceptor(
  AuthenticationManager authenticationManager, 
  AccessDecisionManager customAccessDecisionManager) {

    ChannelSecurityInterceptor 
      channelSecurityInterceptor = new ChannelSecurityInterceptor();

    channelSecurityInterceptor
      .setAuthenticationManager(authenticationManager);

    channelSecurityInterceptor
      .setAccessDecisionManager(customAccessDecisionManager);

    return channelSecurityInterceptor;
}
```

`AuthenticationManager`和`AccessDecisionManager`bean 被定义为:

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends GlobalMethodSecurityConfiguration {

    @Override
    @Bean
    public AuthenticationManager 
      authenticationManager() throws Exception {
        return super.authenticationManager();
    }

    @Bean
    public AccessDecisionManager customAccessDecisionManager() {
        List<AccessDecisionVoter<? extends Object>> 
          decisionVoters = new ArrayList<>();
        decisionVoters.add(new RoleVoter());
        decisionVoters.add(new UsernameAccessDecisionVoter());
        AccessDecisionManager accessDecisionManager
          = new AffirmativeBased(decisionVoters);
        return accessDecisionManager;
    }
}
```

这里，我们使用两个`AccessDecisionVoter` : `RoleVoter`和一个自定义`UsernameAccessDecisionVoter.`

现在，我们可以使用那个`ChannelSecurityInterceptor`来保护我们的频道。我们需要做的是通过`@SecureChannel`注释来装饰通道:

```java
@Bean(name = "startDirectChannel")
@SecuredChannel(
  interceptor = "channelSecurityInterceptor", 
  sendAccess = { "ROLE_VIEWER","jane" })
public DirectChannel startDirectChannel() {
    return new DirectChannel();
}

@Bean(name = "endDirectChannel")
@SecuredChannel(
  interceptor = "channelSecurityInterceptor", 
  sendAccess = {"ROLE_EDITOR"})
public DirectChannel endDirectChannel() {
    return new DirectChannel();
}
```

`@SecureChannel`接受三个属性:

*   `interceptor`属性:引用一个`ChannelSecurityInterceptor` bean。
*   `sendAccess`和`receiveAccess` 属性:包含在通道上调用`send`或`receive`动作的策略。

在上面的例子中，我们期望只有拥有`ROLE_VIEWER`或用户名`jane`的用户可以从`startDirectChannel`发送消息。

此外，只有拥有`ROLE_EDITOR`的用户才能向`endDirectChannel`发送消息。

我们在客户`AccessDecisionManager:`的支持下实现了这一点，无论是`RoleVoter`还是`UsernameAccessDecisionVoter`都返回了肯定的响应，访问被授予。

## 4。`ServiceActivator` 担保

值得一提的是，我们也可以通过 Spring 方法安全保护我们的`ServiceActivator`。因此，我们需要启用方法安全性注释:

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends GlobalMethodSecurityConfiguration {
    //....
}
```

为了简单起见，在本文中，我们将只使用 Spring `pre` 和 `post`注释，所以我们将把`@EnableGlobalMethodSecurity`注释添加到我们的配置类中，并将`prePostEnabled`设置为`true`。

现在我们可以用一个`@PreAuthorization`注释来保护我们的`ServiceActivator`:

```java
@ServiceActivator(
  inputChannel = "startDirectChannel", 
  outputChannel = "endDirectChannel")
@PreAuthorize("hasRole('ROLE_LOGGER')")
public Message<?> logMessage(Message<?> message) {
    Logger.getAnonymousLogger().info(message.toString());
    return message;
}
```

这里的`ServiceActivator`接收来自`startDirectChannel`的消息，并将消息输出给`endDirectChannel`。

此外，该方法只有在当前的`Authentication`主体拥有角色`ROLE_LOGGER`时才可访问。

## 5。安全上下文传播

**弹簧`SecurityContext`默认为线绕**。这意味着`SecurityContext`不会传播到子线程。

对于上面所有的例子，我们都使用了`DirectChannel`和`ServiceActivator`——它们都运行在一个线程中；因此，`SecurityContext`在整个流程中都是可用的。

然而，**与`QueueChannel`、`ExecutorChannel`和`PublishSubscribeChannel`一起使用`Executor,` 时，消息会从一个线程转移到其他线程**。在这种情况下，我们需要将`SecurityContext`传播给所有接收消息的线程。

让我们创建另一个消息流，它以一个`PublishSubscribeChannel` 通道开始，两个`ServiceActivator`订阅了该通道:

```java
@Bean(name = "startPSChannel")
@SecuredChannel(
  interceptor = "channelSecurityInterceptor", 
  sendAccess = "ROLE_VIEWER")
public PublishSubscribeChannel startChannel() {
    return new PublishSubscribeChannel(executor());
}

@ServiceActivator(
  inputChannel = "startPSChannel", 
  outputChannel = "finalPSResult")
@PreAuthorize("hasRole('ROLE_LOGGER')")
public Message<?> changeMessageToRole(Message<?> message) {
    return buildNewMessage(getRoles(), message);
}

@ServiceActivator(
  inputChannel = "startPSChannel", 
  outputChannel = "finalPSResult")
@PreAuthorize("hasRole('ROLE_VIEWER')")
public Message<?> changeMessageToUserName(Message<?> message) {
    return buildNewMessage(getUsername(), message);
}
```

在上面的例子中，我们有两个`ServiceActivator`订阅了`startPSChannel.` ,通道需要一个角色为`ROLE_VIEWER`的`Authentication`主体能够向它发送消息。

同样，只有当`Authentication`主体拥有`ROLE_LOGGER` 角色时，我们才能调用`changeMessageToRole`服务。

此外，只有当`Authentication`主体拥有角色`ROLE_VIEWER`时，才能调用`changeMessageToUserName`服务。

同时，`startPSChannel` 将在`ThreadPoolTaskExecutor:`的支持下运行

```java
@Bean
public ThreadPoolTaskExecutor executor() {
    ThreadPoolTaskExecutor pool = new ThreadPoolTaskExecutor();
    pool.setCorePoolSize(10);
    pool.setMaxPoolSize(10);
    pool.setWaitForTasksToCompleteOnShutdown(true);
    return pool;
}
```

因此，两个`ServiceActivator`将在两个不同的线程中运行。**为了将`SecurityContext`传播给那些线程，我们需要向我们的消息通道添加一个`SecurityContextPropagationChannelInterceptor`** :

```java
@Bean
@GlobalChannelInterceptor(patterns = { "startPSChannel" })
public ChannelInterceptor securityContextPropagationInterceptor() {
    return new SecurityContextPropagationChannelInterceptor();
}
```

注意我们是如何用`@GlobalChannelInterceptor`注释来修饰`SecurityContextPropagationChannelInterceptor` 的。我们还将我们的`startPSChannel`添加到它的`patterns`属性中。

因此，上面的配置声明来自当前线程的`SecurityContext`将被传播到从`startPSChannel`派生的任何线程。

## 6。测试

让我们开始使用一些 JUnit 测试来验证我们的消息流。

### 6.1。依赖性

当然，我们在这一点上需要`spring-security-test`依赖关系:

```java
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <version>5.0.3.RELEASE</version>
    <scope>test</scope>
</dependency>
```

同样，可以从 Maven Central 查看最新版本: `[spring-security-test](https://web.archive.org/web/20220627074627/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-security-test).`

### 6.2。测试安全通道

首先，我们试图向我们的`startDirectChannel:`发送一条消息

```java
@Test(expected = AuthenticationCredentialsNotFoundException.class)
public void 
  givenNoUser_whenSendToDirectChannel_thenCredentialNotFound() {

    startDirectChannel
      .send(new GenericMessage<String>(DIRECT_CHANNEL_MESSAGE));
}
```

因为通道是安全的，所以当发送消息而不提供身份验证对象时，我们预计会出现`AuthenticationCredentialsNotFoundException`异常。

接下来，我们提供一个角色为`ROLE_VIEWER,`的用户，并向我们的`startDirectChannel`发送一条消息:

```java
@Test
@WithMockUser(roles = { "VIEWER" })
public void 
  givenRoleViewer_whenSendToDirectChannel_thenAccessDenied() {
    expectedException.expectCause
      (IsInstanceOf.<Throwable> instanceOf(AccessDeniedException.class));

    startDirectChannel
      .send(new GenericMessage<String>(DIRECT_CHANNEL_MESSAGE));
 }
```

现在，尽管我们的用户可以向`startDirectChannel`发送消息，因为他有角色`ROLE_VIEWER`，但是他不能调用请求角色为`ROLE_LOGGER`的用户的`logMessage`服务。

在这种情况下，将抛出一个原因为`AcessDeniedException`的`MessageHandlingException`。

测试将抛出原因为`AccessDeniedExcecption`的`MessageHandlingException`。因此，我们使用一个`ExpectedException`规则的实例来验证异常的原因。

接下来，我们为用户提供用户名`jane`和两个角色:`ROLE_LOGGER`和`ROLE_EDITOR.`

然后再次尝试向`startDirectChannel` 发送消息`:`

```java
@Test
@WithMockUser(username = "jane", roles = { "LOGGER", "EDITOR" })
public void 
  givenJaneLoggerEditor_whenSendToDirectChannel_thenFlowCompleted() {
    startDirectChannel
      .send(new GenericMessage<String>(DIRECT_CHANNEL_MESSAGE));
    assertEquals
      (DIRECT_CHANNEL_MESSAGE, messageConsumer.getMessageContent());
}
```

消息将在我们的流程中成功传播，从`startDirectChannel` 到`logMessage`激活器，然后到`endDirectChannel`。这是因为所提供的身份验证对象拥有访问这些组件所需的所有权限。

### 6.3。测试`SecurityContext`传播

在声明测试用例之前，我们可以用`PublishSubscribeChannel`回顾一下我们例子的整个流程:

*   流程从具有策略`sendAccess = “ROLE_VIEWER”`的`startPSChannel`开始
*   两个`ServiceActivator`订阅该频道:一个具有安全注释`@PreAuthorize(“hasRole(‘ROLE_LOGGER')”)`，一个具有安全注释`@PreAuthorize(“hasRole(‘ROLE_VIEWER')”)`

因此，首先我们为用户提供角色`ROLE_VIEWER`,并尝试向我们的渠道发送消息:

```java
@Test
@WithMockUser(username = "user", roles = { "VIEWER" })
public void 
  givenRoleUser_whenSendMessageToPSChannel_thenNoMessageArrived() 
  throws IllegalStateException, InterruptedException {

    startPSChannel
      .send(new GenericMessage<String>(DIRECT_CHANNEL_MESSAGE));

    executor
      .getThreadPoolExecutor()
      .awaitTermination(2, TimeUnit.SECONDS);

    assertEquals(1, messageConsumer.getMessagePSContent().size());
    assertTrue(
      messageConsumer
      .getMessagePSContent().values().contains("user"));
}
```

**由于我们的用户只有角色`ROLE_VIEWER`，消息只能通过`startPSChannel`和一个`ServiceActivator`** 。

因此，在流程结束时，我们只收到一条消息。

让我们为用户提供两个角色`ROLE_VIEWER`和`ROLE_LOGGER`:

```java
@Test
@WithMockUser(username = "user", roles = { "LOGGER", "VIEWER" })
public void 
  givenRoleUserAndLogger_whenSendMessageToPSChannel_then2GetMessages() 
  throws IllegalStateException, InterruptedException {
    startPSChannel
      .send(new GenericMessage<String>(DIRECT_CHANNEL_MESSAGE));

    executor
      .getThreadPoolExecutor()
      .awaitTermination(2, TimeUnit.SECONDS);

    assertEquals(2, messageConsumer.getMessagePSContent().size());
    assertTrue
      (messageConsumer
      .getMessagePSContent()
      .values().contains("user"));
    assertTrue
      (messageConsumer
      .getMessagePSContent()
      .values().contains("ROLE_LOGGER,ROLE_VIEWER"));
}
```

现在，我们可以在流程结束时接收到这两个消息，因为用户拥有所需的所有权限。

## 7。结论

在本教程中，我们探索了在 Spring Integration 中使用 Spring Security 来保护消息通道和`ServiceActivator`的可能性。

和往常一样，我们可以在 Github 上找到所有的例子[。](https://web.archive.org/web/20220627074627/https://github.com/eugenp/tutorials/tree/master/spring-integration)