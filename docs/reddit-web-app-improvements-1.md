# Reddit 应用程序的第一轮改进

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-web-app-improvements-1>

## 1。概述

Reddit 网络应用程序[案例研究](/web/20211127053454/https://www.baeldung.com/case-study-a-reddit-app-with-spring) 进展顺利——小型网络应用程序正在成形并慢慢变得可用。

在这一部分中，我们将对现有的功能做一些小的改进——一些面向外部，一些不面向外部——总体来说**让应用程序变得更好**。

## 2。设置检查

让我们从一些简单但有用的检查开始，这些检查需要在应用程序启动时运行:

```
@Autowired
private UserRepository repo;

@PostConstruct
public void startupCheck() {
    if (StringUtils.isBlank(accessTokenUri) || 
      StringUtils.isBlank(userAuthorizationUri) || 
      StringUtils.isBlank(clientID) || StringUtils.isBlank(clientSecret)) {
        throw new RuntimeException("Incomplete reddit properties");
    }
    repo.findAll();
}
```

请注意，在依赖注入过程结束后，我们如何使用`@PostConstruct`注释来挂钩应用程序的生命周期。

简单的目标是:

*   检查我们是否拥有访问 Reddit API 所需的所有属性
*   检查持久层是否在工作(通过发出一个简单的`findAll`调用)

如果我们失败了——我们很早就失败了。

## 3。Reddit 的“请求过多”问题

Reddit API 在不发送唯一的“`User-Agent`”的速率限制请求方面非常积极。

因此，我们需要使用自定义的`Interceptor`将这个唯一的`User-Agent` 头添加到我们的`redditRestTemplate`中:

### 3.1。`Interceptor`创建自定义

这是我们定制的拦截器-`UserAgentInterceptor`:

```
public class UserAgentInterceptor implements ClientHttpRequestInterceptor {

    @Override
    public ClientHttpResponse intercept(
      HttpRequest request, byte[] body, 
      ClientHttpRequestExecution execution) throws IOException {

        HttpHeaders headers = request.getHeaders();
        headers.add("User-Agent", "Schedule with Reddit");
        return execution.execute(request, body);
    }
}
```

### 3.2。配置`redditRestTemplate`

我们当然需要用我们正在使用的`redditRestTemplate` 来设置这个拦截器:

```
@Bean
public OAuth2RestTemplate redditRestTemplate(OAuth2ClientContext clientContext) {
    OAuth2RestTemplate template = new OAuth2RestTemplate(reddit(), clientContext);
    List<ClientHttpRequestInterceptor> list = new ArrayList<ClientHttpRequestInterceptor>();
    list.add(new UserAgentInterceptor());
    template.setInterceptors(list);
    return template;
}
```

## 4。配置用于测试的 H2 数据库

接下来，让我们继续设置一个内存中的数据库 H2 进行测试。我们需要将这种依赖性添加到我们的`pom.xml`:

```
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.187</version>
</dependency>
```

并定义一个 `persistence-test.properties`:

```
## DataSource Configuration ###
jdbc.driverClassName=org.h2.Driver
jdbc.url=jdbc:h2:mem:oauth_reddit;DB_CLOSE_DELAY=-1
jdbc.user=sa
jdbc.pass=
## Hibernate Configuration ##
hibernate.dialect=org.hibernate.dialect.H2Dialect
hibernate.hbm2ddl.auto=update
```

## 5。切换到百里香叶

JSP 出来了，百里香进来了。

### 5.1。修改`pom.xml`

首先，我们需要将这些依赖项添加到 pom.xml 中:

```
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>2.1.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring4</artifactId>
    <version>2.1.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity3</artifactId>
    <version>2.1.2.RELEASE</version>
</dependency>
```

### 5.2。创造`ThymeleafConfig`

接下来——一个简单的`ThymeleafConfig`:

```
@Configuration
public class ThymeleafConfig {
    @Bean
    public TemplateResolver templateResolver() {
        ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver();
        templateResolver.setPrefix("/WEB-INF/jsp/");
        templateResolver.setSuffix(".jsp");
        return templateResolver;
    }

    @Bean
    public SpringTemplateEngine templateEngine() {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver());
        templateEngine.addDialect(new SpringSecurityDialect());
        return templateEngine;
    }

    @Bean
    public ViewResolver viewResolver() {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setTemplateEngine(templateEngine());
        viewResolver.setOrder(1);
        return viewResolver;
    }
}
```

并将其添加到我们的`ServletInitializer`:

```
@Override
protected WebApplicationContext createServletApplicationContext() {
    AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
    context.register(PersistenceJPAConfig.class, WebConfig.class, 
      SecurityConfig.class, ThymeleafConfig.class);
    return context;
}
```

### 5.3。修改`home.html`

以及主页的快速修改:

```
<html>
<head>
<title>Schedule to Reddit</title>
</head>
<body>
<div class="container">
        <h1>Welcome, <small><span sec:authentication="principal.username">Bob</span></small></h1>
        <br/>
        <a href="posts" >My Scheduled Posts</a>
        <a href="post" >Post to Reddit</a>
        <a href="postSchedule" >Schedule Post to Reddit</a>
</div>
</body>
</html>
```

## 6。注销

现在，让我们做一些实际上对应用程序的最终用户可见的改进。我们将从注销开始。

通过修改我们的安全配置，我们在应用程序中添加了一个简单的注销选项:

```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .....
        .and()
        .logout()
        .deleteCookies("JSESSIONID")
        .logoutUrl("/logout")
        .logoutSuccessUrl("/");
}
```

## 7 .**。子编辑自动完成**

接下来——让我们实现一个简单的自动完成功能,用于填充子编辑；手动编写不是一个好方法，因为很有可能出错。

让我们从客户端开始:

```
<input id="sr" name="sr"/>
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
<script src="https://ajax.googleapis.com/ajax/libs/jqueryui/1.11.2/jquery-ui.min.js"></script>
<script>
$(function() {
    $( "#sr" ).autocomplete({
        source: "/subredditAutoComplete"
    });
});
</script>
```

很简单。现在，服务器端:

```
@RequestMapping(value = "/subredditAutoComplete")
@ResponseBody
public String subredditAutoComplete(@RequestParam("term") String term) {
    MultiValueMap<String, String> param = new LinkedMultiValueMap<String, String>();
    param.add("query", term);
    JsonNode node = redditRestTemplate.postForObject(
      "https://oauth.reddit.com//api/search_reddit_names", param, JsonNode.class);
    return node.get("names").toString();
}
```

## 8。检查链接是否已经在 Reddit 上

接下来，让我们看看如何检查一个链接是否已经提交给 Reddit。

这是我们的`submissionForm.html`:

```
<input name="url" />
<input name="sr">

<a href="#" onclick="checkIfAlreadySubmitted()">Check if already submitted</a>
<span id="checkResult" style="display:none"></span>

<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
<script>
$(function() {
    $("input[name='url'],input[name='sr']").focus(function (){
        $("#checkResult").hide();
    });
});
function checkIfAlreadySubmitted(){
    var url = $("input[name='url']").val();
    var sr = $("input[name='sr']").val();
    if(url.length >3 && sr.length > 3){
        $.post("checkIfAlreadySubmitted",{url: url, sr: sr}, function(data){
            var result = JSON.parse(data);
            if(result.length == 0){
                $("#checkResult").show().html("Not submitted before");
            }else{
                $("#checkResult").show().html(
               'Already submitted <b><a target="_blank" href="http://www.reddit.com'
               +result[0].data.permalink+'">here</a></b>');
            }
        });
    }
    else{
        $("#checkResult").show().html("Too short url and/or subreddit");
    }
}           
</script>
```

这是我们的控制器方法:

```
@RequestMapping(value = "/checkIfAlreadySubmitted", method = RequestMethod.POST)
@ResponseBody
public String checkIfAlreadySubmitted(
  @RequestParam("url") String url, @RequestParam("sr") String sr) {
    JsonNode node = redditRestTemplate.getForObject(
      "https://oauth.reddit.com/r/" + sr + "/search?q=url:" + url + "&restrict;_sr=on", JsonNode.class);
    return node.get("data").get("children").toString();
}
```

## 9。部署到 Heroku

最后，我们将部署到 Heroku，并使用他们的免费层来支持示例应用程序。

### 9.1。修改`pom.xml`

首先，我们需要将这个 **Web Runner 插件**添加到`pom.xml`中:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>2.3</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals><goal>copy</goal></goals>
            <configuration>
                <artifactItems>
                    <artifactItem>
                        <groupId>com.github.jsimone</groupId>
                        <artifactId>webapp-runner</artifactId>
                        <version>7.0.57.2</version>
                        <destFileName>webapp-runner.jar</destFileName>
                    </artifactItem>
                </artifactItems>
            </configuration>
        </execution>
    </executions>
</plugin>
```

注意–我们将使用 Web Runner 在 Heroku 上发布我们的应用程序。

我们将在 Heroku 上使用 Postgresql，因此我们需要依赖驱动程序:

```
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>9.4-1201-jdbc41</version>
</dependency>
```

### 9.2。`Procfile`

我们需要定义将在服务器上运行的流程，如下所示:

```
web:    java $JAVA_OPTS -jar target/dependency/webapp-runner.jar --port $PORT target/*.war
```

### 9.3。创建 Heroku App

要从您的项目中创建 Heroku 应用程序，我们只需:

```
cd path_to_your_project
heroku login
heroku create
```

### 9.4。数据库配置

接下来，我们需要使用应用程序的`Postgres`数据库属性来配置数据库。

例如，以下是 persistence-prod.properties:

```
## DataSource Configuration ##
jdbc.driverClassName=org.postgresql.Driver
jdbc.url=jdbc:postgresql://hostname:5432/databasename
jdbc.user=xxxxxxxxxxxxxx
jdbc.pass=xxxxxxxxxxxxxxxxx

## Hibernate Configuration ##
hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
hibernate.hbm2ddl.auto=update
```

注意，我们需要从 Heroku dashborad 获取数据库细节[主机名、数据库名、用户和密码]。

同样——和大多数情况一样，关键字“user”是`Postgres` 中的一个[保留字，所以我们需要更改我们的`User`实体表名:](https://web.archive.org/web/20211127053454/https://www.postgresql.org/docs/7.3/static/sql-keywords-appendix.html)

```
@Entity
@Table(name = "APP_USER")
public class User { .... }
```

### 9.5。将代码推送到 Heoku

现在，让我们将代码推送给 Heroku:

```
git add .
git commit -m "init"
git push heroku master
```

## 10。结论

在我们案例研究的第四部分，重点是微小但重要的改进。如果你一直在关注，你可以看到这是如何成为一个有趣和有用的小应用程序。