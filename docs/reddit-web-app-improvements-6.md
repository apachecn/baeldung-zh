# Reddit 应用程序的第六轮改进

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-web-app-improvements-6>

## 1。概述

在这篇文章中，我们将总结对 Reddit 应用程序的改进。

## 2。命令 API 安全性

首先，我们将做一些工作来保护命令 API，以防止除所有者之外的用户操纵资源。

### 2.1。配置

我们将从在配置中启用`@Preauthorize`开始:

```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
```

### 2.2。授权命令

接下来，让我们借助一些 Spring 安全表达式在控制器层中授权我们的命令:

```java
@PreAuthorize("@resourceSecurityService.isPostOwner(#postDto.id)")
@RequestMapping(value = "/{id}", method = RequestMethod.PUT)
@ResponseStatus(HttpStatus.OK)
public void updatePost(@RequestBody ScheduledPostUpdateCommandDto postDto) {
    ...
}

@PreAuthorize("@resourceSecurityService.isPostOwner(#id)")
@RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deletePost(@PathVariable("id") Long id) {
    ...
}
```

```java
@PreAuthorize("@resourceSecurityService.isRssFeedOwner(#feedDto.id)")
@RequestMapping(value = "/{id}", method = RequestMethod.PUT)
@ResponseStatus(HttpStatus.OK)
public void updateFeed(@RequestBody FeedUpdateCommandDto feedDto) {
    ..
}

@PreAuthorize("@resourceSecurityService.isRssFeedOwner(#id)")
@RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deleteFeed(@PathVariable("id") Long id) {
    ...
}
```

请注意:

*   我们使用“#”来访问方法参数——正如我们在`#id`中所做的
*   我们使用“@”来访问一个 bean——就像我们在`@resourceSecurityService`中所做的那样

### 2.3。资源安全服务

负责检查所有权的服务如下所示:

```java
@Service
public class ResourceSecurityService {

    @Autowired
    private PostRepository postRepository;

    @Autowired
    private MyFeedRepository feedRepository;

    public boolean isPostOwner(Long postId) {
        UserPrincipal userPrincipal = (UserPrincipal) 
          SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        User user = userPrincipal.getUser();
        Post post = postRepository.findOne(postId);
        return post.getUser().getId() == user.getId();
    }

    public boolean isRssFeedOwner(Long feedId) {
        UserPrincipal userPrincipal = (UserPrincipal) 
          SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        User user = userPrincipal.getUser();
        MyFeed feed = feedRepository.findOne(feedId);
        return feed.getUser().getId() == user.getId();
    }
}
```

请注意:

*   `isPostOwner()`:检查当前用户是否拥有给定`postId`的`Post`
*   `isRssFeedOwner()`:检查当前用户是否拥有给定`feedId`的`MyFeed`

### 2.4。异常处理

接下来，我们将简单地处理`AccessDeniedException`–如下所示:

```java
@ExceptionHandler({ AuthenticationCredentialsNotFoundException.class, AccessDeniedException.class })
public ResponseEntity<Object> handleAccessDeniedException(final Exception ex, final WebRequest request) {
    logger.error("403 Status Code", ex);
    ApiError apiError = new ApiError(HttpStatus.FORBIDDEN, ex);
    return new ResponseEntity<Object>(apiError, new HttpHeaders(), HttpStatus.FORBIDDEN);
}
```

### 2.5。授权测试

最后，我们将测试我们的命令授权:

```java
public class CommandAuthorizationLiveTest extends ScheduledPostLiveTest {

    @Test
    public void givenPostOwner_whenUpdatingScheduledPost_thenUpdated() throws ParseException, IOException {
        ScheduledPostDto post = newDto();
        post.setTitle("new title");
        Response response = withRequestBody(givenAuth(), post).put(urlPrefix + "/api/scheduledPosts/" + post.getId());

        assertEquals(200, response.statusCode());
    }

    @Test
    public void givenUserOtherThanOwner_whenUpdatingScheduledPost_thenForbidden() throws ParseException, IOException {
        ScheduledPostDto post = newDto();
        post.setTitle("new title");
        Response response = withRequestBody(givenAnotherUserAuth(), post).put(urlPrefix + "/api/scheduledPosts/" + post.getId());

        assertEquals(403, response.statusCode());
    }

    private RequestSpecification givenAnotherUserAuth() {
        FormAuthConfig formConfig = new FormAuthConfig(
          urlPrefix + "/j_spring_security_check", "username", "password");
        return RestAssured.given().auth().form("test", "test", formConfig);
    }
}
```

请注意`givenAuth()`实现是如何使用用户“john”的，而`givenAnotherUserAuth()`是如何使用用户“test”的——这样我们就可以测试出这些涉及两个不同用户的复杂场景。

## 3。更多重新提交选项

接下来，我们将添加一个有趣的选项——**在一两天后**向 Reddit 重新提交文章，而不是立即提交。

我们将从修改预定的 post 重新提交选项开始，我们将拆分`timeInterval`。这曾经有两个独立的职责；那是:

*   提交帖子和分数检查之间的时间
*   分数检查和下一次提交时间之间的时间

我们不会把这两个责任分开:`checkAfterInterval`和`submitAfterInterval`。

### 3.1。发布实体

我们将通过删除以下内容来修改帖子和首选项实体:

```java
private int timeInterval;
```

并补充道:

```java
private int checkAfterInterval;

private int submitAfterInterval;
```

注意，我们将对相关的 dto 做同样的事情。

### 3.2。调度器

接下来，我们将修改我们的调度程序以使用新的时间间隔，如下所示:

```java
private void checkAndReSubmitInternal(Post post) {
    if (didIntervalPass(post.getSubmissionDate(), post.getCheckAfterInterval())) {
        PostScores postScores = getPostScores(post);
        ...
}

private void checkAndDeleteInternal(Post post) {
    if (didIntervalPass(post.getSubmissionDate(), post.getCheckAfterInterval())) {
        PostScores postScores = getPostScores(post);
        ...
}

private void resetPost(Post post, String failReason) {
    long time = new Date().getTime();
    time += TimeUnit.MILLISECONDS.convert(post.getSubmitAfterInterval(), TimeUnit.MINUTES);
    post.setSubmissionDate(new Date(time))
    ...
}
```

注意，对于带有`submissionDate` **T** 和`checkAfterInterval` **t1** 和`submitAfterInterval` **t2** 以及尝试次数> 1 的预定 post，我们将有:

1.  在 **T** 第一次提交帖子
2.  调度程序在 **T+t1** 检查 post 分数
3.  假设帖子没有达到目标分数，则在 **T+t1+t2** 第二次提交帖子

## 4。对 OAuth2 访问令牌的额外检查

接下来，我们将围绕访问令牌添加一些额外的检查。

有时，用户访问令牌可能会被破坏，从而导致应用程序出现意外行为。如果发生这种情况，我们将通过允许用户将其帐户重新连接到 Reddit 来解决这个问题，从而获得一个新的访问令牌。

### 4.1。Reddit 控制器

下面是简单的控制器级别检查—`isAccessTokenValid()`:

```java
@RequestMapping(value = "/isAccessTokenValid")
@ResponseBody
public boolean isAccessTokenValid() {
    return redditService.isCurrentUserAccessTokenValid();
}
```

### 4.2。Reddit 服务

下面是服务级别实现:

```java
@Override
public boolean isCurrentUserAccessTokenValid() {
    UserPrincipal userPrincipal = (UserPrincipal) 
      SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    User currentUser = userPrincipal.getUser();
    if (currentUser.getAccessToken() == null) {
        return false;
    }
    try {
        redditTemplate.needsCaptcha();
    } catch (Exception e) {
        redditTemplate.setAccessToken(null);
        currentUser.setAccessToken(null);
        currentUser.setRefreshToken(null);
        currentUser.setTokenExpiration(null);
        userRepository.save(currentUser);
        return false;
    }
    return true;
}
```

这里发生的事情很简单。如果用户已经有了访问令牌，我们将尝试使用简单的`needsCaptcha`调用来访问 Reddit API。

如果调用失败，则当前令牌无效，因此我们将重置它。当然，这会导致用户被提示重新连接他们的帐户到 Reddit。

### 4.3。前端

最后，我们将在主页上展示这一点:

```java
<div id="connect" style="display:none">
    <a href="redditLogin">Connect your Account to Reddit</a>
</div>

<script>
$.get("api/isAccessTokenValid", function(data){
    if(!data){
        $("#connect").show();
    }
});
</script>
```

请注意，如果访问令牌无效，将如何向用户显示“连接到 Reddit”链接。

## 5。分成多个模块

接下来，我们将应用程序分成模块。我们将采用 4 个模块: **reddit-common** 、 **reddit-rest** 、 **reddit-ui** 和 **reddit-web** 。

### 5.1。父级

首先，让我们从包装所有子模块父模块开始。

父模块 **reddit-scheduler** 包含子模块和一个简单的`pom.xml`，如下所示:

```java
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.baeldung</groupId>
    <artifactId>reddit-scheduler</artifactId>
    <version>0.2.0-SNAPSHOT</version>
    <name>reddit-scheduler</name>
    <packaging>pom</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.2.7.RELEASE</version>
    </parent>

    <modules>
        <module>reddit-common</module>
        <module>reddit-rest</module>
        <module>reddit-ui</module>
        <module>reddit-web</module>
    </modules>

    <properties>
        <!-- dependency versions and properties -->
    </properties>

</project>
```

所有的属性和依赖版本都将在这里声明，在父模块`pom.xml`中，供所有子模块使用。

### 5.2。公共模块

现在，让我们来谈谈我们的 **reddit-common** 模块。该模块将包含持久性、服务和 reddit 相关资源。它还包含持久性和集成测试。

本模块包含的配置类有`CommonConfig`、 `PersistenceJpaConfig, RedditConfig`、 `ServiceConfig`、 `WebGeneralConfig`。

下面是简单的`pom.xml`:

```java
<project>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>reddit-common</artifactId>
    <name>reddit-common</name>
    <packaging>jar</packaging>

    <parent>
        <groupId>org.baeldung</groupId>
        <artifactId>reddit-scheduler</artifactId>
        <version>0.2.0-SNAPSHOT</version>
    </parent>

</project>
```

### 5.3。休息模块

我们的 **reddit-rest** 模块包含 rest 控制器和 dto。

该模块中唯一的配置类是`WebApiConfig`。

下面是`pom.xml`:

```java
<project>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>reddit-rest</artifactId>
    <name>reddit-rest</name>
    <packaging>jar</packaging>

    <parent>
        <groupId>org.baeldung</groupId>
        <artifactId>reddit-scheduler</artifactId>
        <version>0.2.0-SNAPSHOT</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.baeldung</groupId>
            <artifactId>reddit-common</artifactId>
            <version>0.2.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
    ...
```

该模块还包含所有异常处理逻辑。

### 5.4。用户界面模块

**reddit-ui** 模块包含前端和 MVC 控制器。

包含的配置类有`WebFrontendConfig`和`ThymeleafConfig`。

我们需要更改百里香配置，从资源类路径而不是服务器上下文中加载模板:

```java
@Bean
public TemplateResolver templateResolver() {
    SpringResourceTemplateResolver templateResolver = new SpringResourceTemplateResolver();
    templateResolver.setPrefix("classpath:/");
    templateResolver.setSuffix(".html");
    templateResolver.setCacheable(false);
    return templateResolver;
}
```

下面是简单的`pom.xml`:

```java
<project>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>reddit-ui</artifactId>
    <name>reddit-ui</name>
    <packaging>jar</packaging>

    <parent>
        <groupId>org.baeldung</groupId>
        <artifactId>reddit-scheduler</artifactId>
        <version>0.2.0-SNAPSHOT</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.baeldung</groupId>
            <artifactId>reddit-common</artifactId>
            <version>0.2.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
...
```

我们现在也有了一个更简单的异常处理程序，用于处理前端异常:

```java
@ControllerAdvice
public class RestExceptionHandler extends ResponseEntityExceptionHandler implements Serializable {

    private static final long serialVersionUID = -3365045939814599316L;

    @ExceptionHandler({ UserApprovalRequiredException.class, UserRedirectRequiredException.class })
    public String handleRedirect(RuntimeException ex, WebRequest request) {
        logger.info(ex.getLocalizedMessage());
        throw ex;
    }

    @ExceptionHandler({ Exception.class })
    public String handleInternal(RuntimeException ex, WebRequest request) {
        logger.error(ex);
        String response = "Error Occurred: " + ex.getMessage();
        return "redirect:/submissionResponse?msg=" + response;
    }
}
```

### 5.5。网络模块

最后，这是我们的 reddit-web 模块。

该模块包含资源、安全配置和`SpringBootApplication`配置——如下:

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {
    @Bean
    public ServletRegistrationBean frontendServlet() {
        AnnotationConfigWebApplicationContext dispatcherContext = 
          new AnnotationConfigWebApplicationContext();
        dispatcherContext.register(WebFrontendConfig.class, ThymeleafConfig.class);
        ServletRegistrationBean registration = new ServletRegistrationBean(
          new DispatcherServlet(dispatcherContext), "/*");
        registration.setName("FrontendServlet");
        registration.setLoadOnStartup(1);
        return registration;
    }

    @Bean
    public ServletRegistrationBean apiServlet() {
        AnnotationConfigWebApplicationContext dispatcherContext = 
          new AnnotationConfigWebApplicationContext();
        dispatcherContext.register(WebApiConfig.class);
        ServletRegistrationBean registration = new ServletRegistrationBean(
          new DispatcherServlet(dispatcherContext), "/api/*");
        registration.setName("ApiServlet");
        registration.setLoadOnStartup(2);
        return registration;
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        application.sources(Application.class, CommonConfig.class, 
          PersistenceJpaConfig.class, RedditConfig.class, 
          ServiceConfig.class, WebGeneralConfig.class);
        return application;
    }

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        super.onStartup(servletContext);
        servletContext.addListener(new SessionListener());
        servletContext.addListener(new RequestContextListener());
        servletContext.addListener(new HttpSessionEventPublisher());
    }

    public static void main(String... args) {
        SpringApplication.run(Application.class, args);
    }
}
```

这里是`pom.xml`:

```java
<project>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>reddit-web</artifactId>
    <name>reddit-web</name>
    <packaging>war</packaging>

    <parent>
        <groupId>org.baeldung</groupId>
        <artifactId>reddit-scheduler</artifactId>
        <version>0.2.0-SNAPSHOT</version>
    </parent>

    <dependencies>
	<dependency>
            <groupId>org.baeldung</groupId>
            <artifactId>reddit-common</artifactId>
            <version>0.2.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.baeldung</groupId>
            <artifactId>reddit-rest</artifactId>
            <version>0.2.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.baeldung</groupId>
            <artifactId>reddit-ui</artifactId>
            <version>0.2.0-SNAPSHOT</version>
        </dependency>
...
```

请注意，这是唯一的 war、可部署的模块——所以应用程序现在已经很好地模块化了，但是仍然作为一个整体进行部署。

## 6。结论

我们即将完成 Reddit 案例研究。这是一个非常酷的应用程序，它是根据我的个人需求开发的，而且运行得非常好。