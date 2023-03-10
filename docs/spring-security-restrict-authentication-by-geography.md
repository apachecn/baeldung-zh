# 仅允许来自接受位置的具有 Spring 安全性的身份验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-restrict-authentication-by-geography>

## 1。概述

在本教程中，我们将重点介绍一个非常有趣的安全特性——根据用户的位置来保护他们的帐户。

简而言之，**我们将阻止任何来自不寻常或非标准位置的登录**，并允许用户以安全的方式启用新位置。

这是注册系列的[部分，自然是建立在现有代码基础之上的。](/web/20220822105732/https://www.baeldung.com/spring-security-registration)

## 2。用户位置模型

首先，让我们看看我们的`UserLocation`模型——它保存了关于用户登录位置的信息；每个用户至少有一个与其帐户相关联的位置:

```java
@Entity
public class UserLocation {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String country;

    private boolean enabled;

    @ManyToOne(targetEntity = User.class, fetch = FetchType.EAGER)
    @JoinColumn(nullable = false, name = "user_id")
    private User user;

    public UserLocation() {
        super();
        enabled = false;
    }

    public UserLocation(String country, User user) {
        super();
        this.country = country;
        this.user = user;
        enabled = false;
    }
    ...
}
```

我们将向我们的存储库添加一个简单的检索操作:

```java
public interface UserLocationRepository extends JpaRepository<UserLocation, Long> {
    UserLocation findByCountryAndUser(String country, User user);
}
```

注意到

*   默认情况下，新的`UserLocation`被禁用
*   每个用户至少有一个与其帐户相关联的位置，这是他们注册时访问应用程序的第一个位置

## 3。注册

现在，让我们讨论如何修改注册过程以添加默认用户位置:

```java
@PostMapping("/user/registration")
public GenericResponse registerUserAccount(@Valid UserDto accountDto, 
  HttpServletRequest request) {

    User registered = userService.registerNewUserAccount(accountDto);
    userService.addUserLocation(registered, getClientIP(request));
    ...
}
```

在服务实现中，我们将通过用户的 IP 地址获取国家:

```java
public void addUserLocation(User user, String ip) {
    InetAddress ipAddress = InetAddress.getByName(ip);
    String country 
      = databaseReader.country(ipAddress).getCountry().getName();
    UserLocation loc = new UserLocation(country, user);
    loc.setEnabled(true);
    loc = userLocationRepo.save(loc);
}
```

注意，我们使用 [GeoLite2](https://web.archive.org/web/20220822105732/https://dev.maxmind.com/geoip/geoip2/geolite2/) 数据库从 IP 地址获取国家。为了使用 [GeoLite2](https://web.archive.org/web/20220822105732/https://dev.maxmind.com/geoip/geoip2/geolite2/) ，我们需要 maven 依赖关系:

```java
<dependency>
    <groupId>com.maxmind.geoip2</groupId>
    <artifactId>geoip2</artifactId>
    <version>2.15.0</version>
</dependency>
```

我们还需要定义一个简单的 bean:

```java
@Bean
public DatabaseReader databaseReader() throws IOException, GeoIp2Exception {
    File resource = new File("src/main/resources/GeoLite2-Country.mmdb");
    return new DatabaseReader.Builder(resource).build();
}
```

我们已经从 MaxMind 这里加载了 [GeoLite2 国家](https://web.archive.org/web/20220822105732/https://dev.maxmind.com/geoip/geolite2-free-geolocation-data?lang=en)数据库。

## 4。安全登录

现在我们已经有了用户的默认国家，我们将在身份验证后添加一个简单的位置检查器:

```java
@Autowired
private DifferentLocationChecker differentLocationChecker;

@Bean
public DaoAuthenticationProvider authProvider() {
    CustomAuthenticationProvider authProvider = new CustomAuthenticationProvider();
    authProvider.setUserDetailsService(userDetailsService);
    authProvider.setPasswordEncoder(encoder());
    authProvider.setPostAuthenticationChecks(differentLocationChecker);
    return authProvider;
}
```

这是我们的`DifferentLocationChecker`:

```java
@Component
public class DifferentLocationChecker implements UserDetailsChecker {

    @Autowired
    private IUserService userService;

    @Autowired
    private HttpServletRequest request;

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Override
    public void check(UserDetails userDetails) {
        String ip = getClientIP();
        NewLocationToken token = userService.isNewLoginLocation(userDetails.getUsername(), ip);
        if (token != null) {
            String appUrl = 
              "http://" 
              + request.getServerName() 
              + ":" + request.getServerPort() 
              + request.getContextPath();

            eventPublisher.publishEvent(
              new OnDifferentLocationLoginEvent(
                request.getLocale(), userDetails.getUsername(), ip, token, appUrl));
            throw new UnusualLocationException("unusual location");
        }
    }

    private String getClientIP() {
        String xfHeader = request.getHeader("X-Forwarded-For");
        if (xfHeader == null) {
            return request.getRemoteAddr();
        }
        return xfHeader.split(",")[0];
    }
}
```

请注意，我们使用了 s `etPostAuthenticationChecks()`以便**仅在成功认证**后运行检查——当用户提供正确的凭证时。

同样，我们的自定义`UnusualLocationException`是一个简单的`AuthenticationException`。

我们还需要修改我们的`AuthenticationFailureHandler`来定制错误消息:

```java
@Override
public void onAuthenticationFailure(...) {
    ...
    else if (exception.getMessage().equalsIgnoreCase("unusual location")) {
        errorMessage = messages.getMessage("auth.message.unusual.location", null, locale);
    }
}
```

现在，让我们深入了解一下`isNewLoginLocation()` 的实现:

```java
@Override
public NewLocationToken isNewLoginLocation(String username, String ip) {
    try {
        InetAddress ipAddress = InetAddress.getByName(ip);
        String country 
          = databaseReader.country(ipAddress).getCountry().getName();

        User user = repository.findByEmail(username);
        UserLocation loc = userLocationRepo.findByCountryAndUser(country, user);
        if ((loc == null) || !loc.isEnabled()) {
            return createNewLocationToken(country, user);
        }
    } catch (Exception e) {
        return null;
    }
    return null;
}
```

请注意，当用户提供正确的凭证时，我们如何检查他们的位置。如果该位置已经与该用户帐户相关联，则用户能够成功地进行身份验证。

如果没有，我们创建一个`NewLocationToken`和一个禁用的`UserLocation`，允许用户启用这个新位置。在接下来的章节中会有更多的介绍。

```java
private NewLocationToken createNewLocationToken(String country, User user) {
    UserLocation loc = new UserLocation(country, user);
    loc = userLocationRepo.save(loc);
    NewLocationToken token = new NewLocationToken(UUID.randomUUID().toString(), loc);
    return newLocationTokenRepository.save(token);
}
```

最后，下面是简单的`NewLocationToken`实现——允许用户将新位置与他们的帐户关联起来:

```java
@Entity
public class NewLocationToken {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String token;

    @OneToOne(targetEntity = UserLocation.class, fetch = FetchType.EAGER)
    @JoinColumn(nullable = false, name = "user_location_id")
    private UserLocation userLocation;

    ...
}
```

## 5。异地登录事件

当用户从不同的位置登录时，我们创建了一个`NewLocationToken` 并使用它来触发一个`OnDifferentLocationLoginEvent`:

```java
public class OnDifferentLocationLoginEvent extends ApplicationEvent {
    private Locale locale;
    private String username;
    private String ip;
    private NewLocationToken token;
    private String appUrl;
}
```

`DifferentLocationLoginListener`如下处理我们的事件:

```java
@Component
public class DifferentLocationLoginListener 
  implements ApplicationListener<OnDifferentLocationLoginEvent> {

    @Autowired
    private MessageSource messages;

    @Autowired
    private JavaMailSender mailSender;

    @Autowired
    private Environment env;

    @Override
    public void onApplicationEvent(OnDifferentLocationLoginEvent event) {
        String enableLocUri = event.getAppUrl() + "/user/enableNewLoc?token=" 
          + event.getToken().getToken();
        String changePassUri = event.getAppUrl() + "/changePassword.html";
        String recipientAddress = event.getUsername();
        String subject = "Login attempt from different location";
        String message = messages.getMessage("message.differentLocation", new Object[] { 
          new Date().toString(), 
          event.getToken().getUserLocation().getCountry(), 
          event.getIp(), enableLocUri, changePassUri 
          }, event.getLocale());

        SimpleMailMessage email = new SimpleMailMessage();
        email.setTo(recipientAddress);
        email.setSubject(subject);
        email.setText(message);
        email.setFrom(env.getProperty("support.email"));
        mailSender.send(email);
    }
}
```

请注意，**当用户从不同的位置登录时，我们将发送电子邮件通知他们**。

如果其他人试图登录他们的帐户，他们当然会更改密码。如果他们识别出身份验证尝试，他们将能够将新的登录位置与他们的帐户相关联。

## 6。启用新的登录位置

最后，既然用户已经得到了可疑活动的通知，让我们看看**应用程序将如何处理启用新位置**:

```java
@RequestMapping(value = "/user/enableNewLoc", method = RequestMethod.GET)
public String enableNewLoc(Locale locale, Model model, @RequestParam("token") String token) {
    String loc = userService.isValidNewLocationToken(token);
    if (loc != null) {
        model.addAttribute(
          "message", 
          messages.getMessage("message.newLoc.enabled", new Object[] { loc }, locale)
        );
    } else {
        model.addAttribute(
          "message", 
          messages.getMessage("message.error", null, locale)
        );
    }
    return "redirect:/login?lang=" + locale.getLanguage();
}
```

而我们的`isValidNewLocationToken()`方法:

```java
@Override
public String isValidNewLocationToken(String token) {
    NewLocationToken locToken = newLocationTokenRepository.findByToken(token);
    if (locToken == null) {
        return null;
    }
    UserLocation userLoc = locToken.getUserLocation();
    userLoc.setEnabled(true);
    userLoc = userLocationRepo.save(userLoc);
    newLocationTokenRepository.delete(locToken);
    return userLoc.getCountry();
}
```

简单地说，我们将启用与令牌相关联的`UserLocation`,然后删除令牌。

## 7.限制

为了结束这篇文章，我们需要提到上述实现的一个限制。我们用来确定客户端 IP 的方法是:

```java
private final String getClientIP(HttpServletRequest request) 
```

并不总是返回客户端的正确 IP 地址。如果 Spring Boot 应用程序部署在本地，则返回的 IP 地址是(除非有不同的配置)0.0.0.0。由于 MaxMind 数据库中没有该地址，因此无法注册和登录。如果客户端的 IP 地址不在数据库中，也会出现同样的问题。

## 8。结论

在本教程中，我们重点介绍了一种强大的新机制来为我们的应用程序增加安全性—**根据用户的位置限制意外的用户活动。**

和往常一样，完整的实现可以在 GiHub 上找到[。](https://web.archive.org/web/20220822105732/https://github.com/Baeldung/spring-security-registration)