# Reddit 应用程序的第五轮改进

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-web-app-improvements-5>

## 1。概述

让我们从正在进行的案例研究开始，继续推进 Reddit 应用程序。

## 2。就帖子评论发送电子邮件通知

Reddit 丢失了电子邮件通知——简单明了。我希望看到的是——每当有人对我的帖子发表评论时，我都会收到一封附有评论的简短电子邮件通知。

所以，简单地说，这就是这个特性的目的，通过电子邮件通知评论。

我们将实现一个简单的调度程序来检查:

*   哪些用户应该收到帖子回复的电子邮件通知
*   如果用户在 Reddit 收件箱中收到任何帖子回复

然后，它会简单地发送一封带有未读帖子回复的电子邮件通知。

### 2.1。用户偏好

首先，我们需要通过添加以下内容来修改我们的首选实体和 d to:

```java
private boolean sendEmailReplies;
```

允许用户选择是否希望收到帖子回复的电子邮件通知。

### 2.2。通知调度器

接下来，这是我们的简单调度程序:

```java
@Component
public class NotificationRedditScheduler {

    @Autowired
    private INotificationRedditService notificationRedditService;

    @Autowired
    private PreferenceRepository preferenceRepository;

    @Scheduled(fixedRate = 60 * 60 * 1000)
    public void checkInboxUnread() {
        List<Preference> preferences = preferenceRepository.findBySendEmailRepliesTrue();
        for (Preference preference : preferences) {
            notificationRedditService.checkAndNotify(preference);
        }
    }
}
```

请注意，调度程序每小时运行一次——但是如果我们愿意，我们当然可以以更短的频率运行。

### 2.3。通知服务

现在，让我们讨论一下我们的通知服务:

```java
@Service
public class NotificationRedditService implements INotificationRedditService {
    private Logger logger = LoggerFactory.getLogger(getClass());
    private static String NOTIFICATION_TEMPLATE = "You have %d unread post replies.";
    private static String MESSAGE_TEMPLATE = "%s replied on your post %s : %s";

    @Autowired
    @Qualifier("schedulerRedditTemplate")
    private OAuth2RestTemplate redditRestTemplate;

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Autowired
    private UserRepository userRepository;

    @Override
    public void checkAndNotify(Preference preference) {
        try {
            checkAndNotifyInternal(preference);
        } catch (Exception e) {
            logger.error(
              "Error occurred while checking and notifying = " + preference.getEmail(), e);
        }
    }

    private void checkAndNotifyInternal(Preference preference) {
        User user = userRepository.findByPreference(preference);
        if ((user == null) || (user.getAccessToken() == null)) {
            return;
        }

        DefaultOAuth2AccessToken token = new DefaultOAuth2AccessToken(user.getAccessToken());
        token.setRefreshToken(new DefaultOAuth2RefreshToken((user.getRefreshToken())));
        token.setExpiration(user.getTokenExpiration());
        redditRestTemplate.getOAuth2ClientContext().setAccessToken(token);

        JsonNode node = redditRestTemplate.getForObject(
          "https://oauth.reddit.com/message/selfreply?mark=false", JsonNode.class);
        parseRepliesNode(preference.getEmail(), node);
    }

    private void parseRepliesNode(String email, JsonNode node) {
        JsonNode allReplies = node.get("data").get("children");
        int unread = 0;
        for (JsonNode msg : allReplies) {
            if (msg.get("data").get("new").asBoolean()) {
                unread++;
            }
        }
        if (unread == 0) {
            return;
        }

        JsonNode firstMsg = allReplies.get(0).get("data");
        String author = firstMsg.get("author").asText();
        String postTitle = firstMsg.get("link_title").asText();
        String content = firstMsg.get("body").asText();

        StringBuilder builder = new StringBuilder();
        builder.append(String.format(NOTIFICATION_TEMPLATE, unread));
        builder.append("\n");
        builder.append(String.format(MESSAGE_TEMPLATE, author, postTitle, content));
        builder.append("\n");
        builder.append("Check all new replies at ");
        builder.append("https://www.reddit.com/message/unread/");

        eventPublisher.publishEvent(new OnNewPostReplyEvent(email, builder.toString()));
    }
}
```

请注意:

*   我们调用 Reddit API，获得所有回复，然后逐一检查，看看是否是新的“未读”。
*   如果有未读回复，我们将触发一个事件，向该用户发送电子邮件通知。

### 2.4。新回复事件

这是我们的简单事件:

```java
public class OnNewPostReplyEvent extends ApplicationEvent {
    private String email;
    private String content;

    public OnNewPostReplyEvent(String email, String content) {
        super(email);
        this.email = email;
        this.content = content;
    }
}
```

### 2.5。回复听众

最后，这是我们的听众:

```java
@Component
public class ReplyListener implements ApplicationListener<OnNewPostReplyEvent> {
    @Autowired
    private JavaMailSender mailSender;

    @Autowired
    private Environment env;

    @Override
    public void onApplicationEvent(OnNewPostReplyEvent event) {
        SimpleMailMessage email = constructEmailMessage(event);
        mailSender.send(email);
    }

    private SimpleMailMessage constructEmailMessage(OnNewPostReplyEvent event) {
        String recipientAddress = event.getEmail();
        String subject = "New Post Replies";
        SimpleMailMessage email = new SimpleMailMessage();
        email.setTo(recipientAddress);
        email.setSubject(subject);
        email.setText(event.getContent());
        email.setFrom(env.getProperty("support.email"));
        return email;
    }
}
```

## 3。会话并发控制

接下来，让我们设置一些关于应用程序允许的并发会话数量的更严格的规则。更重要的是—**我们不允许并发会话**:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.sessionManagement()
          .maximumSessions(1)
          .maxSessionsPreventsLogin(true);
}
```

注意——因为我们正在使用定制的`UserDetails`实现——我们需要覆盖`equals()`和`hashcode()`,因为会话控制策略将所有主体存储在一个映射中，并且需要能够检索它们:

```java
public class UserPrincipal implements UserDetails {

    private User user;

    @Override
    public int hashCode() {
        int prime = 31;
        int result = 1;
        result = (prime * result) + ((user == null) ? 0 : user.hashCode());
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (obj == null) {
            return false;
        }
        if (getClass() != obj.getClass()) {
            return false;
        }
        UserPrincipal other = (UserPrincipal) obj;
        if (user == null) {
            if (other.user != null) {
                return false;
            }
        } else if (!user.equals(other.user)) {
            return false;
        }
        return true;
    }
}
```

## 4。独立的 API Servlet

应用程序现在在同一个 servlet 中同时服务于前端和 API 这并不理想。

现在让我们将这两个主要职责分开，并把它们放入两个不同的 servlets 中:

```java
@Bean
public ServletRegistrationBean frontendServlet() {
    ServletRegistrationBean registration = 
      new ServletRegistrationBean(new DispatcherServlet(), "/*");

    Map<String, String> params = new HashMap<String, String>();
    params.put("contextClass", 
      "org.springframework.web.context.support.AnnotationConfigWebApplicationContext");
    params.put("contextConfigLocation", "org.baeldung.config.frontend");
    registration.setInitParameters(params);

    registration.setName("FrontendServlet");
    registration.setLoadOnStartup(1);
    return registration;
}

@Bean
public ServletRegistrationBean apiServlet() {
    ServletRegistrationBean registration = 
      new ServletRegistrationBean(new DispatcherServlet(), "/api/*");

    Map<String, String> params = new HashMap<String, String>();
    params.put("contextClass", 
      "org.springframework.web.context.support.AnnotationConfigWebApplicationContext");
    params.put("contextConfigLocation", "org.baeldung.config.api");

    registration.setInitParameters(params);
    registration.setName("ApiServlet");
    registration.setLoadOnStartup(2);
    return registration;
}

@Override
protected SpringApplicationBuilder configure(final SpringApplicationBuilder application) {
    application.sources(Application.class);
    return application;
}
```

请注意，我们现在有了一个前端 servlet 来处理所有前端请求，并且只启动特定于前端的 Spring 上下文；然后我们有了 API Servlet——为 API 引导一个完全不同的 Spring 上下文。

同样非常重要的是，这两个 servlet Spring 上下文是子上下文。父上下文——由`SpringApplicationBuilder`创建——扫描`root`包的常见配置，如持久性、服务等。

这是我们的`WebFrontendConfig`:

```java
@Configuration
@EnableWebMvc
@ComponentScan({ "org.baeldung.web.controller.general" })
public class WebFrontendConfig implements WebMvcConfigurer {

    @Bean
    public static PropertySourcesPlaceholderConfigurer 
      propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/home");
        ...
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
    }
}
```

和`WebApiConfig`:

```java
@Configuration
@EnableWebMvc
@ComponentScan({ "org.baeldung.web.controller.rest", "org.baeldung.web.dto" })
public class WebApiConfig implements WebMvcConfigurer {

    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}
```

## 5。取消订阅源 URL

最后——我们将使 RSS 工作得更好。

有时，RSS 提要会通过 Feedburner 等外部服务被缩短或重定向——因此当我们在应用程序中加载提要的 URL 时——我们需要确保在所有重定向中遵循该 URL，直到到达我们真正关心的主 URL。

因此，当我们将文章的链接发布到 Reddit 时，我们实际上发布了正确的原始 URL:

```java
@RequestMapping(value = "/url/original")
@ResponseBody
public String getOriginalLink(@RequestParam("url") String sourceUrl) {
    try {
        List<String> visited = new ArrayList<String>();
        String currentUrl = sourceUrl;
        while (!visited.contains(currentUrl)) {
            visited.add(currentUrl);
            currentUrl = getOriginalUrl(currentUrl);
        }
        return currentUrl;
    } catch (Exception ex) {
        // log the exception
        return sourceUrl;
    }
}

private String getOriginalUrl(String oldUrl) throws IOException {
    URL url = new URL(oldUrl);
    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
    connection.setInstanceFollowRedirects(false);
    String originalUrl = connection.getHeaderField("Location");
    connection.disconnect();
    if (originalUrl == null) {
        return oldUrl;
    }
    if (originalUrl.indexOf("?") != -1) {
        return originalUrl.substring(0, originalUrl.indexOf("?"));
    }
    return originalUrl;
}
```

对于这种实现，需要注意以下几点:

*   我们正在处理多层次的重定向
*   我们还跟踪所有访问过的网址，以避免重定向循环

## 6。结论

仅此而已——一些坚实的改进让 Reddit 应用程序变得更好。下一步是对 API 进行一些性能测试，看看它在生产场景中的表现如何。