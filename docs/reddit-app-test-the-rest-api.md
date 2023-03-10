# 测试 Reddit 应用程序的 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-app-test-the-rest-api>

## 1。概述

我们已经为简单的 Reddit 应用构建了 REST API 一段时间了——是时候认真对待了，T2 开始测试它了。

现在我们[终于将](/web/20220120083301/https://www.baeldung.com/reddit-app-replace-reddit-auth-with-form-based-login)切换到一个更简单的认证机制，这样做也更容易。我们将使用[强大的`rest-assured`库](https://web.archive.org/web/20220120083301/https://github.com/rest-assured/rest-assured)进行所有这些现场测试。

## 2。初始设置

API 测试需要用户来运行；为了简化针对 API 的测试，我们将预先在应用程序引导上创建一个测试用户:

```java
@Component
public class Setup {
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PreferenceRepository preferenceRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @PostConstruct
    private void createTestUser() {
        User userJohn = userRepository.findByUsername("john");
        if (userJohn == null) {
            userJohn = new User();
            userJohn.setUsername("john");
            userJohn.setPassword(passwordEncoder.encode("123"));
            userJohn.setAccessToken("token");
            userRepository.save(userJohn);
            final Preference pref = new Preference();
            pref.setTimezone(TimeZone.getDefault().getID());
            pref.setEmail("[[email protected]](/web/20220120083301/https://www.baeldung.com/cdn-cgi/l/email-protection)");
            preferenceRepository.save(pref);
            userJohn.setPreference(pref);
            userRepository.save(userJohn);
        }
    }
}
```

注意`Setup`是一个普通的 bean，我们使用`@PostConstruct`注释来挂钩实际的设置逻辑。

## 3。支持现场测试

在我们开始实际编写测试之前，**让我们首先设置一些基本的支持功能**，然后我们可以利用这些功能。

我们需要诸如身份验证、URL 路径之类的东西，可能还需要一些 JSON 标记和解组功能:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = { TestConfig.class }, 
  loader = AnnotationConfigContextLoader.class)
public class AbstractLiveTest {
    public static final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm");

    @Autowired
    private CommonPaths commonPaths;

    protected String urlPrefix;

    protected ObjectMapper objectMapper = new ObjectMapper().setDateFormat(dateFormat);

    @Before
    public void setup() {
        urlPrefix = commonPaths.getServerRoot();
    }

    protected RequestSpecification givenAuth() {
        FormAuthConfig formConfig 
          = new FormAuthConfig(urlPrefix + "/j_spring_security_check", "username", "password");
        return RestAssured.given().auth().form("john", "123", formConfig);
    }

    protected RequestSpecification withRequestBody(RequestSpecification req, Object obj) 
      throws JsonProcessingException {
        return req.contentType(MediaType.APPLICATION_JSON_VALUE)
          .body(objectMapper.writeValueAsString(obj));
    }
}
```

我们只是定义了一些简单的助手方法和字段，以使实际测试更容易:

*   `givenAuth()`:进行认证
*   `withRequestBody()`:发送一个`Object`的 JSON 表示作为 HTTP 请求的主体

这里是我们的简单 bean—`CommonPaths`——为系统的 URL 提供了一个清晰的抽象:

```java
@Component
@PropertySource({ "classpath:web-${envTarget:local}.properties" })
public class CommonPaths {

    @Value("${http.protocol}")
    private String protocol;

    @Value("${http.port}")
    private String port;

    @Value("${http.host}")
    private String host;

    @Value("${http.address}")
    private String address;

    public String getServerRoot() {
        if (port.equals("80")) {
            return protocol + "://" + host + "/" + address;
        }
        return protocol + "://" + host + ":" + port + "/" + address;
    }
}
```

和本地版本的属性文件:`web-local.properties`:

```java
http.protocol=http
http.port=8080
http.host=localhost
http.address=reddit-scheduler
```

最后，非常简单的测试弹簧配置:

```java
@Configuration
@ComponentScan({ "org.baeldung.web.live" })
public class TestConfig {
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
}
```

## 4。测试 API

我们要测试的第一个 API 是`/scheduledPosts` API:

```java
public class ScheduledPostLiveTest extends AbstractLiveTest {
    private static final String date = "2016-01-01 00:00";

    private Post createPost() throws ParseException, IOException {
        Post post = new Post();
        post.setTitle("test");
        post.setUrl("test.com");
        post.setSubreddit("test");
        post.setSubmissionDate(dateFormat.parse(date));

        Response response = withRequestBody(givenAuth(), post)
          .post(urlPrefix + "/api/scheduledPosts?date=" + date);

        return objectMapper.reader().forType(Post.class).readValue(response.asString());
    }
}
```

首先，让我们测试一下**安排一个新帖子**:

```java
@Test
public void whenScheduleANewPost_thenCreated() 
  throws ParseException, IOException {
    Post post = new Post();
    post.setTitle("test");
    post.setUrl("test.com");
    post.setSubreddit("test");
    post.setSubmissionDate(dateFormat.parse(date));

    Response response = withRequestBody(givenAuth(), post)
      .post(urlPrefix + "/api/scheduledPosts?date=" + date);

    assertEquals(201, response.statusCode());
    Post result = objectMapper.reader().forType(Post.class).readValue(response.asString());
    assertEquals(result.getUrl(), post.getUrl());
}
```

接下来，让我们测试**检索用户的所有预定帖子**:

```java
@Test
public void whenGettingUserScheduledPosts_thenCorrect() 
  throws ParseException, IOException {
    createPost();

    Response response = givenAuth().get(urlPrefix + "/api/scheduledPosts?page=0");

    assertEquals(201, response.statusCode());
    assertTrue(response.as(List.class).size() > 0);
}
```

接下来，让我们测试一下**编辑预定的帖子**:

```java
@Test
public void whenUpdatingScheduledPost_thenUpdated() 
  throws ParseException, IOException {
    Post post = createPost();

    post.setTitle("new title");
    Response response = withRequestBody(givenAuth(), post).
      put(urlPrefix + "/api/scheduledPosts/" + post.getId() + "?date=" + date);

    assertEquals(200, response.statusCode());
    response = givenAuth().get(urlPrefix + "/api/scheduledPosts/" + post.getId());
    assertTrue(response.asString().contains(post.getTitle()));
}
```

最后，让我们测试一下 API 中的删除操作:

```java
@Test
public void whenDeletingScheduledPost_thenDeleted() 
  throws ParseException, IOException {
    Post post = createPost();
    Response response = givenAuth().delete(urlPrefix + "/api/scheduledPosts/" + post.getId());

    assertEquals(204, response.statusCode());
}
```

## 5。测试/ `sites` API

接下来，让我们测试发布站点资源(由用户定义的站点)的 API:

```java
public class MySitesLiveTest extends AbstractLiveTest {

    private Site createSite() throws ParseException, IOException {
        Site site = new Site("/feed/");
        site.setName("baeldung");

        Response response = withRequestBody(givenAuth(), site)
          .post(urlPrefix + "/sites");

        return objectMapper.reader().forType(Site.class).readValue(response.asString());
    }
}
```

让我们测试**检索用户的所有站点**:

```java
@Test
public void whenGettingUserSites_thenCorrect() 
  throws ParseException, IOException {
    createSite();
    Response response = givenAuth().get(urlPrefix + "/sites");

    assertEquals(200, response.statusCode());
    assertTrue(response.as(List.class).size() > 0);
}
```

并且还检索一个站点的文章:

```java
@Test
public void whenGettingSiteArticles_thenCorrect() 
  throws ParseException, IOException {
    Site site = createSite();
    Response response = givenAuth().get(urlPrefix + "/sites/articles?id=" + site.getId());

    assertEquals(200, response.statusCode());
    assertTrue(response.as(List.class).size() > 0);
}
```

接下来，让我们测试一下添加新站点的**:**

```java
@Test
public void whenAddingNewSite_thenCorrect() 
  throws ParseException, IOException {
    Site site = createSite();

    Response response = givenAuth().get(urlPrefix + "/sites");
    assertTrue(response.asString().contains(site.getUrl()));
}
```

并且**删除**它:

```java
@Test
public void whenDeletingSite_thenDeleted() throws ParseException, IOException {
    Site site = createSite();
    Response response = givenAuth().delete(urlPrefix + "/sites/" + site.getId());

    assertEquals(204, response.statusCode());
}
```

## 6。测试`/user/preferences` API

最后，让我们关注暴露用户偏好的 API。

首先，让我们测试**获取用户偏好**:

```java
@Test
public void whenGettingPrefernce_thenCorrect() {
    Response response = givenAuth().get(urlPrefix + "/user/preference");

    assertEquals(200, response.statusCode());
    assertTrue(response.as(Preference.class).getEmail().contains("john"));
}
```

和**编辑**它们:

```java
@Test
public void whenUpdattingPrefernce_thenCorrect() 
  throws JsonProcessingException {
    Preference pref = givenAuth().get(urlPrefix + "/user/preference").as(Preference.class);
    pref.setEmail("[[email protected]](/web/20220120083301/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    Response response = withRequestBody(givenAuth(), pref).
      put(urlPrefix + "/user/preference/" + pref.getId());

    assertEquals(200, response.statusCode());
    response = givenAuth().get(urlPrefix + "/user/preference");
    assertEquals(response.as(Preference.class).getEmail(), pref.getEmail());
}
```

## 7。结论

在这篇简短的文章中，我们为 REST API 做了一些基本的测试。

虽然没有什么可想象的——需要更高级的场景——但是**这不是关于完美，而是关于进步和公开迭代**。