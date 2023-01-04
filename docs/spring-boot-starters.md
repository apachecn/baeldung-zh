# Spring Boot 初学者入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-starters>

## 1。概述

依赖性管理是任何复杂项目的关键方面。并且手动地这样做是不理想的；你在它上面花的时间越多，你在项目其他重要方面的时间就越少。

Spring Boot 起动机就是为了解决这个问题而制造的。Starter POMs 是一组方便的依赖描述符，可以包含在应用程序中。您可以一站式获得您需要的所有 Spring 和相关技术，而不必搜索样本代码和复制粘贴大量的依赖描述符。

我们有 30 多种可用的启动程序，让我们在接下来的章节中看看其中的一些。

## 2。Web Starter

首先，让我们看看如何开发 REST 服务；我们可以使用像 Spring MVC、Tomcat 和 Jackson 这样的库——一个应用程序有很多依赖项。

Spring Boot 启动器只需添加一个依赖项，就可以帮助减少手动添加依赖项的数量。因此，无需手动指定依赖项，只需添加一个启动器，如下例所示:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

现在我们可以创建一个休息控制器。为了简单起见，我们不使用数据库，而是专注于 REST 控制器:

```
@RestController
public class GenericEntityController {
    private List<GenericEntity> entityList = new ArrayList<>();

    @RequestMapping("/entity/all")
    public List<GenericEntity> findAll() {
        return entityList;
    }

    @RequestMapping(value = "/entity", method = RequestMethod.POST)
    public GenericEntity addEntity(GenericEntity entity) {
        entityList.add(entity);
        return entity;
    }

    @RequestMapping("/entity/findby/{id}")
    public GenericEntity findById(@PathVariable Long id) {
        return entityList.stream().
                 filter(entity -> entity.getId().equals(id)).
                   findFirst().get();
    }
}
```

*GenericEntity* 是一个简单的 bean，具有类型 *Long* 的 *id* 和类型 *String* 的*值*。

就这样——随着应用程序的运行，您可以访问[http://localhost:8080/entity/all](https://web.archive.org/web/20220125082427/http://localhost:8080/springbootapp/entity/all)并检查控制器是否工作。

我们已经创建了一个配置非常简单的 REST 应用程序。

## 3。测试启动器

为了测试，我们通常使用下面的库:Spring Test、JUnit、Hamcrest 和 Mockito。我们可以手动包含所有这些库，但是可以使用 Spring Boot 启动程序以下列方式自动包含这些库:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

注意，您不需要指定工件的版本号。Spring Boot 会计算出使用什么版本——你所需要指定的就是`spring-boot-starter-parent`工件的版本。如果以后您需要升级引导库和依赖项，只需在一个地方升级引导版本，其余的就交给它了。

让我们实际测试我们在前一个例子中创建的控制器。

有两种方法可以测试控制器:

*   使用模拟环境
*   使用嵌入式 Servlet 容器(如 Tomcat 或 Jetty)

在这个例子中，我们将使用一个模拟环境:

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
@WebAppConfiguration
public class SpringBootApplicationIntegrationTest {
    @Autowired
    private WebApplicationContext webApplicationContext;
    private MockMvc mockMvc;

    @Before
    public void setupMockMvc() {
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    }

    @Test
    public void givenRequestHasBeenMade_whenMeetsAllOfGivenConditions_thenCorrect()
      throws Exception { 
        MediaType contentType = new MediaType(MediaType.APPLICATION_JSON.getType(),
        MediaType.APPLICATION_JSON.getSubtype(), Charset.forName("utf8"));
        mockMvc.perform(MockMvcRequestBuilders.get("/entity/all")).
        andExpect(MockMvcResultMatchers.status().isOk()).
        andExpect(MockMvcResultMatchers.content().contentType(contentType)).
        andExpect(jsonPath("$", hasSize(4))); 
    } 
}
```

上面的测试调用了`/entity/all`端点，并验证 JSON 响应包含 4 个元素。为了通过这个测试，我们还必须在控制器类中初始化我们的列表:

```
public class GenericEntityController {
    private List<GenericEntity> entityList = new ArrayList<>();

    {
        entityList.add(new GenericEntity(1l, "entity_1"));
        entityList.add(new GenericEntity(2l, "entity_2"));
        entityList.add(new GenericEntity(3l, "entity_3"));
        entityList.add(new GenericEntity(4l, "entity_4"));
    }
    //...
}
```

这里重要的是， *@WebAppConfiguration* 注释和 *MockMVC* 是 *spring-test* 模块的一部分， *hasSize* 是一个 Hamcrest 匹配器，之前的*@是一个 JUnit 注释。这些都可以通过导入一个 starter 依赖项来实现。*

## 4。数据 JPA 启动程序

大多数 web 应用程序都有某种持久性——通常是 JPA。

与其手动定义所有相关的依赖项，不如让我们从入门开始:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

注意，开箱即用，我们至少自动支持以下数据库:H2、Derby 和 Hsqldb。在我们的例子中，我们将使用 H2。

现在让我们为我们的实体创建存储库:

```
public interface GenericEntityRepository extends JpaRepository<GenericEntity, Long> {}
```

测试代码的时间到了。下面是 JUnit 测试:

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
public class SpringBootJPATest {

    @Autowired
    private GenericEntityRepository genericEntityRepository;

    @Test
    public void givenGenericEntityRepository_whenSaveAndRetreiveEntity_thenOK() {
        GenericEntity genericEntity = 
          genericEntityRepository.save(new GenericEntity("test"));
        GenericEntity foundedEntity = 
          genericEntityRepository.findOne(genericEntity.getId());

        assertNotNull(foundedEntity);
        assertEquals(genericEntity.getValue(), foundedEntity.getValue());
    }
}
```

我们没有花时间指定数据库供应商、URL 连接和凭证。没有额外的配置是必要的，因为我们受益于坚实的启动默认设置；当然，如果需要的话，所有这些细节仍然可以配置。

## 5。邮件启动器

企业开发中一个非常常见的任务是发送电子邮件，直接处理 Java 邮件 API 通常很困难。

Spring Boot 启动器隐藏了这种复杂性——邮件依赖关系可以通过以下方式指定:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

现在我们可以直接使用 *JavaMailSender* ，所以让我们写一些测试。

出于测试目的，我们需要一个简单的 SMTP 服务器。在这个例子中，我们将使用 Wiser。这就是我们将它纳入 POM 的方式:

```
<dependency>
    <groupId>org.subethamail</groupId>
    <artifactId>subethasmtp</artifactId>
    <version>3.1.7</version>
    <scope>test</scope>
</dependency> 
```

Wiser 的最新版本可以在 [Maven central repository](https://web.archive.org/web/20220125082427/https://search.maven.org/classic/#search%7Cga%7C1%7Csubethasmtp) 上找到。

下面是测试的源代码:

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
public class SpringBootMailTest {
    @Autowired
    private JavaMailSender javaMailSender;

    private Wiser wiser;

    private String userTo = "[[email protected]](/web/20220125082427/https://www.baeldung.com/cdn-cgi/l/email-protection)";
    private String userFrom = "[[email protected]](/web/20220125082427/https://www.baeldung.com/cdn-cgi/l/email-protection)";
    private String subject = "Test subject";
    private String textMail = "Text subject mail";

    @Before
    public void setUp() throws Exception {
        final int TEST_PORT = 25;
        wiser = new Wiser(TEST_PORT);
        wiser.start();
    }

    @After
    public void tearDown() throws Exception {
        wiser.stop();
    }

    @Test
    public void givenMail_whenSendAndReceived_thenCorrect() throws Exception {
        SimpleMailMessage message = composeEmailMessage();
        javaMailSender.send(message);
        List<WiserMessage> messages = wiser.getMessages();

        assertThat(messages, hasSize(1));
        WiserMessage wiserMessage = messages.get(0);
        assertEquals(userFrom, wiserMessage.getEnvelopeSender());
        assertEquals(userTo, wiserMessage.getEnvelopeReceiver());
        assertEquals(subject, getSubject(wiserMessage));
        assertEquals(textMail, getMessage(wiserMessage));
    }

    private String getMessage(WiserMessage wiserMessage)
      throws MessagingException, IOException {
        return wiserMessage.getMimeMessage().getContent().toString().trim();
    }

    private String getSubject(WiserMessage wiserMessage) throws MessagingException {
        return wiserMessage.getMimeMessage().getSubject();
    }

    private SimpleMailMessage composeEmailMessage() {
        SimpleMailMessage mailMessage = new SimpleMailMessage();
        mailMessage.setTo(userTo);
        mailMessage.setReplyTo(userFrom);
        mailMessage.setFrom(userFrom);
        mailMessage.setSubject(subject);
        mailMessage.setText(textMail);
        return mailMessage;
    }
}
```

在测试中，前的*@和*后的*@方法负责启动和停止邮件服务器。*

注意，我们连接了*javamail sender*bean——该 bean 是由 Spring Boot 自动创建的**。**

就像 Boot 中的任何其他默认设置一样， *JavaMailSender* 的电子邮件设置可以在*应用程序中定制。*

```
spring.mail.host=localhost
spring.mail.port=25
spring.mail.properties.mail.smtp.auth=false
```

所以我们在`localhost:25`上配置了邮件服务器，并且我们不需要认证。

## 6。结论

在这篇文章中，我们给出了启动器的概述，解释了为什么我们需要它们，并提供了如何在您的项目中使用它们的例子。

让我们回顾一下使用 Spring Boot 起动机的好处:

*   提高 pom 可管理性
*   生产就绪、经过测试和支持的依赖关系配置
*   减少项目的总体配置时间

真正的首发名单可以在[这里](https://web.archive.org/web/20220125082427/https://github.com/spring-projects/spring-boot/tree/main/spring-boot-project/spring-boot-starters)找到。例子的源代码可以在[这里](https://web.archive.org/web/20220125082427/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-artifacts-2)找到。