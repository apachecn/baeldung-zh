# 与 JDBC 的春季会议

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-session-jdbc>

## 1。概述

在这个快速教程中，我们将学习如何使用 Spring 会话 JDBC 将会话信息持久化到数据库中。

出于演示目的，我们将使用内存中的 H2 数据库。

## 2。配置选项

创建我们的示例项目最简单快捷的方法是使用`Spring Boot`。然而，我们还将展示一种非引导的设置方式。

因此，你不需要完成第 3 和第 4 部分。根据我们是否使用`Spring Boot`来配置`Spring Session`来选择一个。

## 3。Spring Boot 配置

首先，让我们看看 JDBC 春季会议所需的配置。

### 3.1。Maven 依赖关系

首先，我们需要将这些依赖项添加到我们的项目中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency> 
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.197</version>
    <scope>runtime</scope>
</dependency>
```

我们的应用程序使用`Spring Boot`运行，父应用程序`pom.xml`为每个条目提供版本。每个依赖项的最新版本可以在这里找到: [spring-boot-starter-web](https://web.archive.org/web/20221208143845/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22) 、 [spring-boot-starter-test、](https://web.archive.org/web/20221208143845/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-test%22) [spring-session-jdbc](https://web.archive.org/web/20221208143845/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.session%22%20AND%20a%3A%22spring-session-jdbc%22) 和 h2。

令人惊讶的是，**我们需要**来启用由关系数据库支持的 Spring 会话**的唯一配置属性在**T0 中

```java
spring.session.store-type=jdbc
```

## 4。标准弹簧配置(无 Spring Boot)

让我们看看没有 spring Boot 的集成和配置 Spring-session——只有普通的 Spring。

### 4.1。Maven 依赖关系

首先，如果我们要将`spring-session-jdbc`添加到一个标准的 Spring 项目中，我们需要将 [spring-session-jdbc](https://web.archive.org/web/20221208143845/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.session%22%20AND%20a%3A%22spring-session-jdbc%22) 和 [h2](https://web.archive.org/web/20221208143845/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.h2database%22%20AND%20a%3A%22h2%22) 添加到我们的`pom.xml`(上一节代码片段中的最后两个依赖项)。

### 4.2。春季会议配置

现在让我们为`Spring Session JDBC`添加一个配置类:

```java
@Configuration
@EnableJdbcHttpSession
public class Config
  extends AbstractHttpSessionApplicationInitializer {

    @Bean
    public EmbeddedDatabase dataSource() {
        return new EmbeddedDatabaseBuilder()
          .setType(EmbeddedDatabaseType.H2)
          .addScript("org/springframework/session/jdbc/schema-h2.sql").build();
    }

    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

正如我们所见，差异很小。现在我们必须显式定义我们的`EmbeddedDatabase`和`PlatformTransactionManager `bean——在前面的配置中，Spring Boot 已经为我们完成了。

以上确保了名为`springSessionRepositoryFilter`的 Spring bean 为每个请求注册到我们的`Servlet Container`。

## 5。一个简单的应用程序

接下来，让我们看一个简单的 REST API，它保存并演示了会话持久性`. `

### 5.1。控制器

首先，让我们添加一个`Controller`类来存储和显示`HttpSession`中的信息:

```java
@Controller
public class SpringSessionJdbcController {

    @GetMapping("/")
    public String index(Model model, HttpSession session) {
        List<String> favoriteColors = getFavColors(session);
        model.addAttribute("favoriteColors", favoriteColors);
        model.addAttribute("sessionId", session.getId());
        return "index";
    }

    @PostMapping("/saveColor")
    public String saveMessage
      (@RequestParam("color") String color, 
      HttpServletRequest request) {

        List<String> favoriteColors 
          = getFavColors(request.getSession());
        if (!StringUtils.isEmpty(color)) {
            favoriteColors.add(color);
            request.getSession().
              setAttribute("favoriteColors", favoriteColors);
        }
        return "redirect:/";
    }

    private List<String> getFavColors(HttpSession session) {
        List<String> favoriteColors = (List<String>) session
          .getAttribute("favoriteColors");

        if (favoriteColors == null) {
            favoriteColors = new ArrayList<>();
        }
        return favoriteColors;
    }
} 
```

## 6。测试我们的实现

现在我们有了一个带有 GET 和 POST 方法的 API，让我们编写调用这两个方法的测试。

在每种情况下，我们都应该能够断言会话信息被持久存储在数据库中。为了验证这一点，我们将直接查询会话数据库。

让我们先来设置一下:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(
  webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class SpringSessionJdbcApplicationTests {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate testRestTemplate;

    private List<String> getSessionIdsFromDatabase() 
      throws SQLException {

        List<String> result = new ArrayList<>();
        ResultSet rs = getResultSet(
          "SELECT * FROM SPRING_SESSION");

        while (rs.next()) {
            result.add(rs.getString("SESSION_ID"));
        }
        return result;
    }

    private List<byte[]> getSessionAttributeBytesFromDb() 
      throws SQLException {

        List<byte[]> result = new ArrayList<>();
        ResultSet rs = getResultSet(
          "SELECT * FROM SPRING_SESSION_ATTRIBUTES");

        while (rs.next()) {
            result.add(rs.getBytes("ATTRIBUTE_BYTES"));
        }
        return result;
    }

    private ResultSet getResultSet(String sql) 
      throws SQLException {

        Connection conn = DriverManager
          .getConnection("jdbc:h2:mem:testdb", "sa", "");
        Statement stat = conn.createStatement();
        return stat.executeQuery(sql);
    }
}
```

注意使用`@FixMethodOrder(MethodSorters.NAME_ASCENDING)`来控制测试用例执行的顺序`. `阅读更多关于它的信息[这里](/web/20221208143845/https://www.baeldung.com/junit-5-test-order)。

让我们首先断言数据库中的会话表是空的:

```java
@Test
public void whenH2DbIsQueried_thenSessionInfoIsEmpty() 
  throws SQLException {

    assertEquals(
      0, getSessionIdsFromDatabase().size());
    assertEquals(
      0, getSessionAttributeBytesFromDatabase().size());
}
```

接下来，我们测试 GET 端点:

```java
@Test
public void whenH2DbIsQueried_thenOneSessionIsCreated() 
  throws SQLException {

    assertThat(this.testRestTemplate.getForObject(
      "http://localhost:" + port + "/", String.class))
      .isNotEmpty();
    assertEquals(1, getSessionIdsFromDatabase().size());
}
```

当第一次调用 API 时，会创建一个会话并保存在数据库中。正如我们所看到的，此时在`SPRING_SESSION `表中只有一行。

最后，我们通过提供一种喜爱的颜色来测试 POST 端点:

```java
@Test
public void whenH2DbIsQueried_thenSessionAttributeIsRetrieved()
  throws Exception {

    MultiValueMap<String, String> map = new LinkedMultiValueMap<>();
    map.add("color", "red");
    this.testRestTemplate.postForObject(
      "http://localhost:" + port + "/saveColor", map, String.class);
    List<byte[]> queryResponse = getSessionAttributeBytesFromDatabase();

    assertEquals(1, queryResponse.size());
    ObjectInput in = new ObjectInputStream(
      new ByteArrayInputStream(queryResponse.get(0)));
    List<String> obj = (List<String>) in.readObject();
    assertEquals("red", obj.get(0));
}
```

不出所料，`SPRING_SESSION_ATTRIBUTES `表坚持最喜欢的颜色。注意，我们必须将`ATTRIBUTE_BYTES `的内容反序列化为一系列`String`对象，因为 Spring 在数据库中保存会话属性时会进行对象序列化。

## 7。它是如何工作的？

查看控制器，没有迹象表明数据库保存了会话信息。所有的神奇都发生在我们在`application.properties`中添加的一行中。

也就是**当我们在幕后指定`spring.session.store-type=jdbc,`时，Spring Boot 会应用一个相当于手动添加`@EnableJdbcHttpSession`注释**的配置。

这创建了一个名为 s `pringSessionRepositoryFilter`的 Spring Bean，它实现了`a SessionRepositoryFilter`。

另一个关键点是，过滤器截取每一个`HttpServletRequest`，并将其包装成一个`SessionRepositoryRequestWrapper`。

它还调用`commitSession`方法来保存会话信息。

## 8。存储在 H2 数据库中的会话信息

通过添加以下属性，我们可以查看存储来自 URL 的会话信息的表—[http://localhost:8080/H2-console/](https://web.archive.org/web/20221208143845/http://localhost:8080/h2-console/):

```java
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

[![spring session jdbc](img/d0d70dbc2cde02cb64a918b24ad47c22.png)](/web/20221208143845/https://www.baeldung.com/wp-content/uploads/2018/07/bael-1911_1.png) [![spring session jdbc](img/6a9704adcd2dd170288a8e4097f65747.png)](/web/20221208143845/https://www.baeldung.com/wp-content/uploads/2018/07/bael-1911_2.png)

## 9.结论

Spring Session 是在分布式系统架构中管理 HTTP 会话的强大工具。Spring 通过提供具有最小配置的预定义模式来处理简单用例的繁重工作。同时，它提供了灵活性，可以根据我们希望如何存储会话信息来提出我们的设计。

最后，关于使用 Spring Session 管理认证信息，您可以参考本文—[Spring Session 指南](/web/20221208143845/https://www.baeldung.com/spring-session)。

和往常一样，你可以在 Github 上找到源代码[。](https://web.archive.org/web/20221208143845/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-session)