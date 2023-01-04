# 使用 Spring Security 防止暴力验证尝试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-block-brute-force-authentication-attempts>

## 1。概述

在这个快速教程中，我们将使用 Spring Security 实现一个基本的解决方案来防止[暴力认证尝试](/web/20221005021715/https://www.baeldung.com/cs/brute-force-cybersecurity-string-search) 。

简而言之，我们将记录从单个 IP 地址发起的失败尝试的次数。如果该特定 IP 超过了设定的请求数量，它将被阻止 24 小时。

## 延伸阅读:

## [Spring 方法安全性介绍](/web/20221005021715/https://www.baeldung.com/spring-security-method-security)

A guide to method-level security using the Spring Security framework.[Read more](/web/20221005021715/https://www.baeldung.com/spring-security-method-security) →

## [Spring 安全过滤器链中的自定义过滤器](/web/20221005021715/https://www.baeldung.com/spring-security-custom-filter)

A quick guide to show steps to add custom filter in Spring Security context.[Read more](/web/20221005021715/https://www.baeldung.com/spring-security-custom-filter) →

## [用于反应式应用的弹簧安全 5](/web/20221005021715/https://www.baeldung.com/spring-security-5-reactive)

A quick and practical example of Spring Security 5 framework's features for securing reactive applications.[Read more](/web/20221005021715/https://www.baeldung.com/spring-security-5-reactive) →

## 2。一个`AuthenticationFailureListener`

让我们从定义一个`AuthenticationFailureListener`开始——监听`AuthenticationFailureBadCredentialsEvent`事件并通知我们认证失败:

```
@Component
public class AuthenticationFailureListener implements 
  ApplicationListener<AuthenticationFailureBadCredentialsEvent> {

    @Autowired
    private HttpServletRequest request;

    @Autowired
    private LoginAttemptService loginAttemptService;

    @Override
    public void onApplicationEvent(AuthenticationFailureBadCredentialsEvent e) {
        final String xfHeader = request.getHeader("X-Forwarded-For");
        if (xfHeader == null) {
            loginAttemptService.loginFailed(request.getRemoteAddr());
        } else {
            loginAttemptService.loginFailed(xfHeader.split(",")[0]);
        }
    }
}
```

请注意，当认证失败时，我们如何通知`LoginAttemptService`不成功尝试的来源 IP 地址。这里，我们从`HttpServletRequest` bean 中获得 IP 地址，这也为我们提供了由代理服务器等转发的请求在`X-Forwarded-For`报头中的起始地址。

## 3。一个`AuthenticationSuccessEventListener`

让我们也定义一个`AuthenticationSuccessEventListener` ——它监听`AuthenticationSuccessEvent`事件并通知我们认证成功:

```
@Component
public class AuthenticationSuccessEventListener implements 
  ApplicationListener<AuthenticationSuccessEvent> {

    @Autowired
    private HttpServletRequest request;

    @Autowired
    private LoginAttemptService loginAttemptService;

    @Override
    public void onApplicationEvent(final AuthenticationSuccessEvent e) {
        final String xfHeader = request.getHeader("X-Forwarded-For");
        if (xfHeader == null) {
            loginAttemptService.loginSucceeded(request.getRemoteAddr());
        } else {
            loginAttemptService.loginSucceeded(xfHeader.split(",")[0]);
        }
    }
}
```

请注意——与失败监听器类似，我们通知`LoginAttemptService`身份验证请求所来自的 IP 地址。

## 4。`LoginAttemptService`

现在，让我们讨论一下我们的`LoginAttemptService`实现；简而言之，我们将每个 IP 地址的错误尝试次数保留 24 小时:

```
@Service
public class LoginAttemptService {

    private final int MAX_ATTEMPT = 10;
    private LoadingCache<String, Integer> attemptsCache;

    public LoginAttemptService() {
        super();
        attemptsCache = CacheBuilder.newBuilder().
          expireAfterWrite(1, TimeUnit.DAYS).build(new CacheLoader<String, Integer>() {
            public Integer load(String key) {
                return 0;
            }
        });
    }

    public void loginSucceeded(String key) {
        attemptsCache.invalidate(key);
    }

    public void loginFailed(String key) {
        int attempts = 0;
        try {
            attempts = attemptsCache.get(key);
        } catch (ExecutionException e) {
            attempts = 0;
        }
        attempts++;
        attemptsCache.put(key, attempts);
    }

    public boolean isBlocked(String key) {
        try {
            return attemptsCache.get(key) >= MAX_ATTEMPT;
        } catch (ExecutionException e) {
            return false;
        }
    }
}
```

请注意**一次不成功的认证尝试会增加该 IP** 的尝试次数，而成功的认证会重置该计数器。

从这一点来看，当我们认证时，这只是一个简单的**检查计数器的问题。**

## 5。`UserDetailsService`

现在，让我们在自定义的`UserDetailsService`实现中添加额外的检查；当我们加载`UserDetails`、**时，我们首先需要检查这个 IP 地址是否被阻止**:

```
@Service("userDetailsService")
@Transactional
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private RoleRepository roleRepository;

    @Autowired
    private LoginAttemptService loginAttemptService;

    @Autowired
    private HttpServletRequest request;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        String ip = getClientIP();
        if (loginAttemptService.isBlocked(ip)) {
            throw new RuntimeException("blocked");
        }

        try {
            User user = userRepository.findByEmail(email);
            if (user == null) {
                return new org.springframework.security.core.userdetails.User(
                  " ", " ", true, true, true, true, 
                  getAuthorities(Arrays.asList(roleRepository.findByName("ROLE_USER"))));
            }

            return new org.springframework.security.core.userdetails.User(
              user.getEmail(), user.getPassword(), user.isEnabled(), true, true, true, 
              getAuthorities(user.getRoles()));
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

这里是`getClientIP()`方法:

```
private String getClientIP() {
    String xfHeader = request.getHeader("X-Forwarded-For");
    if (xfHeader == null){
        return request.getRemoteAddr();
    }
    return xfHeader.split(",")[0];
}
```

注意，我们有一些额外的逻辑来**识别客户端**的原始 IP 地址。在大多数情况下，这是不必要的，但在一些网络场景中，这是必要的。

对于这些罕见的场景，我们使用`X-Forwarded-For`头来获取原始 IP；这个标题的语法如下:

```
X-Forwarded-For: clientIpAddress, proxy1, proxy2
```

另外，请注意 Spring 的另一个非常有趣的功能—**我们需要 HTTP 请求，所以我们只是简单地连接它。**

现在，这很酷。我们将不得不在我们的`web.xml`中添加一个快速监听器，这将使事情变得容易得多。

```
<listener>
    <listener-class>
        org.springframework.web.context.request.RequestContextListener
    </listener-class>
</listener>
```

就这样——我们已经在我们的`web.xml`中定义了这个新的`RequestContextListener`,以便能够访问来自`UserDetailsService`的请求。

## 6。修改`AuthenticationFailureHandler`

最后，让我们修改`CustomAuthenticationFailureHandler`来定制新的错误消息。

我们正在处理用户实际上被阻止 24 小时的情况，我们通知用户，他的 IP 被阻止，因为他超过了允许的最大错误身份验证尝试次数:

```
@Component
public class CustomAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    @Autowired
    private MessageSource messages;

    @Override
    public void onAuthenticationFailure(...) {
        ...

        String errorMessage = messages.getMessage("message.badCredentials", null, locale);
        if (exception.getMessage().equalsIgnoreCase("blocked")) {
            errorMessage = messages.getMessage("auth.message.blocked", null, locale);
        }

        ...
    }
}
```

## 7 .**。结论**

重要的是要明白这是**应对暴力破解密码尝试**的良好开端，但也有改进的空间。生产级暴力防御策略可能涉及比 IP 块更多的元素。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20221005021715/https://github.com/Baeldung/spring-security-registration "The Full Brute-Force Prevention Example Project on Github ")中找到。