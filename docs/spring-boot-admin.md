# Spring Boot 管理指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-admin>

## 1。概述

[Spring Boot 管理](https://web.archive.org/web/20220926192843/https://github.com/codecentric/spring-boot-admin)是一个网络应用，用于管理和监控 Spring Boot 应用。每个应用程序都被视为一个客户机，并注册到管理服务器。在幕后，魔术是由 Spring Boot 致动器端点。

在本文中，我们将描述配置 Spring Boot 管理服务器的步骤，以及应用程序如何成为客户机。

## 2。管理服务器设置

首先，我们需要创建一个简单的 Spring Boot web 应用程序，并添加以下 [Maven 依赖项](https://web.archive.org/web/20220926192843/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-boot-admin-starter-server):

```
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
    <version>2.4.1</version>
</dependency>
```

此后，`@EnableAdminServer` 将可用，因此我们将把它添加到主类中，如下例所示:

```
@EnableAdminServer
@SpringBootApplication
public class SpringBootAdminServerApplication(exclude = AdminServerHazelcastAutoConfiguration.class) {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootAdminServerApplication.class, args);
    }
}
```

此时，我们已经准备好启动服务器并注册客户机应用程序。

## 3。设置客户端

现在，在我们设置好管理服务器之后，我们可以将我们的第一个 Spring Boot 应用程序注册为客户机。我们必须添加下面的 [Maven 依赖关系](https://web.archive.org/web/20220926192843/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-boot-admin-starter-client):

```
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.4.1</version>
</dependency>
```

接下来，我们需要配置客户机来了解管理服务器的基本 URL。为此，我们只需添加以下属性:

```
spring.boot.admin.client.url=http://localhost:8080
```

**从《Spring Boot 2》开始，`health`和`info`以外的端点默认不公开。**

让我们公开所有的端点:

```
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
```

## 4。安全配置

Spring Boot 管理服务器可以访问应用程序的敏感端点，因此建议我们为管理和客户端应用程序添加一些安全配置。

首先，我们将着重于配置管理服务器的安全性。我们必须添加以下 [Maven 依赖关系](https://web.archive.org/web/20220926192843/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-security%22):

```
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-server-ui-login</artifactId>
    <version>1.5.7</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.4.0</version>
</dependency>
```

这将启用安全性并向管理应用程序添加一个登录界面。

接下来，我们将添加一个安全配置类，如下所示:

```
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {
    private final AdminServerProperties adminServer;

    public WebSecurityConfig(AdminServerProperties adminServer) {
        this.adminServer = adminServer;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        SavedRequestAwareAuthenticationSuccessHandler successHandler = 
          new SavedRequestAwareAuthenticationSuccessHandler();
        successHandler.setTargetUrlParameter("redirectTo");
        successHandler.setDefaultTargetUrl(this.adminServer.getContextPath() + "/");

        http
            .authorizeRequests()
                .antMatchers(this.adminServer.getContextPath() + "/assets/**").permitAll()
                .antMatchers(this.adminServer.getContextPath() + "/login").permitAll()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage(this.adminServer.getContextPath() + "/login")
                .successHandler(successHandler)
                .and()
            .logout()
                .logoutUrl(this.adminServer.getContextPath() + "/logout")
                .and()
            .httpBasic()
                .and()
            .csrf()
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                .ignoringRequestMatchers(
                  new AntPathRequestMatcher(this.adminServer.getContextPath() + 
                   "/instances", HttpMethod.POST.toString()), 
                  new AntPathRequestMatcher(this.adminServer.getContextPath() + 
                   "/instances/*", HttpMethod.DELETE.toString()),
                  new AntPathRequestMatcher(this.adminServer.getContextPath() + "/actuator/**"))
                .and()
            .rememberMe()
                .key(UUID.randomUUID().toString())
                .tokenValiditySeconds(1209600);
        return http.build();
    }
}
```

有一个简单的安全配置，但是添加之后，我们会注意到客户机不能再注册到服务器。

为了将客户机注册到新的安全服务器，我们必须在客户机的属性文件中添加更多的配置:

```
spring.boot.admin.client.username=admin
spring.boot.admin.client.password=admin
```

我们在这一点上，我们保护了我们的管理服务器。在生产系统中，我们试图监控的应用程序自然会受到保护。因此，我们也将为客户端添加安全性，并且我们将在管理服务器的 UI 界面中注意到客户端信息不再可用。

我们必须添加一些元数据，我们将发送到管理服务器。服务器使用此信息连接到客户端的端点:

```
spring.security.user.name=client
spring.security.user.password=client
spring.boot.admin.client.instance.metadata.user.name=${spring.security.user.name}
spring.boot.admin.client.instance.metadata.user.password=${spring.security.user.password}
```

当然，通过 HTTP 发送凭证是不安全的——所以通信需要经过 HTTPS。

## 5。监控和管理功能

Spring Boot 管理员可以配置为只显示我们认为有用的信息。我们只需改变默认配置并添加我们自己需要的指标:

```
spring.boot.admin.routes.endpoints=env, metrics, trace, jolokia, info, configprops
```

随着我们的深入，我们会看到还有一些其他的功能可以探索。我们谈论的是`JMX bean management` 使用`Jolokia`以及 `Loglevel` 管理。

Spring Boot 管理员还支持使用 Hazelcast 进行集群复制。我们只需添加下面的 [Maven 依赖关系](https://web.archive.org/web/20220926192843/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.hazelcast%22%20AND%20a%3A%22hazelcast%22)，让自动配置完成剩下的工作:

```
<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast</artifactId>
    <version>4.0.3</version>
</dependency>
```

如果我们想要一个持久的 Hazelcast 实例，我们将使用一个自定义配置:

```
@Configuration
public class HazelcastConfig {

    @Bean
    public Config hazelcast() {
        MapConfig eventStoreMap = new MapConfig("spring-boot-admin-event-store")
          .setInMemoryFormat(InMemoryFormat.OBJECT)
          .setBackupCount(1)
          .setEvictionConfig(new EvictionConfig().setEvictionPolicy(EvictionPolicy.NONE))
          .setMergePolicyConfig(new MergePolicyConfig(PutIfAbsentMergePolicy.class.getName(), 100));

        MapConfig sentNotificationsMap = new MapConfig("spring-boot-admin-application-store")
          .setInMemoryFormat(InMemoryFormat.OBJECT)
          .setBackupCount(1)
          .setEvictionConfig(new EvictionConfig().setEvictionPolicy(EvictionPolicy.LRU))
          .setMergePolicyConfig(new MergePolicyConfig(PutIfAbsentMergePolicy.class.getName(), 100));

        Config config = new Config();
        config.addMapConfig(eventStoreMap);
        config.addMapConfig(sentNotificationsMap);
        config.setProperty("hazelcast.jmx", "true");

        config.getNetworkConfig()
          .getJoin()
          .getMulticastConfig()
          .setEnabled(false);
        TcpIpConfig tcpIpConfig = config.getNetworkConfig()
          .getJoin()
          .getTcpIpConfig();
        tcpIpConfig.setEnabled(true);
        tcpIpConfig.setMembers(Collections.singletonList("127.0.0.1"));
        return config;
    }
}
```

## 6。通知

接下来，让我们讨论一下，如果我们注册的客户端出现问题，是否有可能从管理服务器收到通知。以下通告程序可用于配置:

*   电子邮件
*   PagerDuty
*   OpsGenie
*   Hipchat
*   松弛的
*   我们聊聊吧

### 6.1。电子邮件通知

我们将首先关注为我们的管理服务器配置邮件通知。为此，我们必须添加如下所示的[邮件启动器依赖项](https://web.archive.org/web/20220926192843/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-mail%22):

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
    <version>2.4.0</version>
</dependency>
```

在这之后，我们必须添加一些邮件配置:

```
spring.mail.host=smtp.example.com
spring.mail.username=smtp_user
spring.mail.password=smtp_password
[[email protected]](/web/20220926192843/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

现在，每当我们的注册客户将他的状态从“在线”更改为“离线”或其他状态时，就会有一封电子邮件发送到上面配置的地址。对于其他通告程序，配置是相似的。

### 6.2。Hipchat 通知

正如我们将看到的，与 Hipchat 的集成非常简单；只有几个强制属性需要设置:

```
spring.boot.admin.notify.hipchat.auth-token=<generated_token>
spring.boot.admin.notify.hipchat.room-id=<room-id>
spring.boot.admin.notify.hipchat.url=https://yourcompany.hipchat.com/v2/
```

定义好这些之后，我们会在 Hipchat room 中注意到，每当客户端的状态改变时，我们都会收到通知。

### 6.3。定制通知配置

我们可以配置一个定制的通知系统，拥有一些强大的工具。我们可以使用一个`reminding notifier`来发送一个预定的通知，直到客户端的状态改变。

或者，我们可能希望向一组经过筛选的客户端发送通知。为此，我们可以使用一个`filtering notifier:`

```
@Configuration
public class NotifierConfiguration {
    private final InstanceRepository repository;
    private final ObjectProvider<List<Notifier>> otherNotifiers;

    public NotifierConfiguration(InstanceRepository repository, 
      ObjectProvider<List<Notifier>> otherNotifiers) {
        this.repository = repository;
        this.otherNotifiers = otherNotifiers;
    }

    @Bean
    public FilteringNotifier filteringNotifier() {
        CompositeNotifier delegate = 
          new CompositeNotifier(this.otherNotifiers.getIfAvailable(Collections::emptyList));
        return new FilteringNotifier(delegate, this.repository);
    }

    @Bean
    public LoggingNotifier notifier() {
        return new LoggingNotifier(repository);
    }

    @Primary
    @Bean(initMethod = "start", destroyMethod = "stop")
    public RemindingNotifier remindingNotifier() {
        RemindingNotifier remindingNotifier = new RemindingNotifier(filteringNotifier(), repository);
        remindingNotifier.setReminderPeriod(Duration.ofMinutes(5));
        remindingNotifier.setCheckReminderInverval(Duration.ofSeconds(60));
        return remindingNotifier;
    }
}
```

## 7。结论

这个介绍教程涵盖了简单的步骤，一个人必须做的，以监测和管理他的 Spring Boot 应用程序使用 Spring Boot 管理。

自动配置允许我们只添加一些小的配置，最后，拥有一个完全工作的管理服务器。

和往常一样，本指南的示例代码可以在 Github 上找到[。](https://web.archive.org/web/20220926192843/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-admin)