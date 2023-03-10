# 嵌入在 Spring Boot 应用程序中的键盘锁

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/keycloak-embedded-in-spring-boot-app>

## 1。概述

**Keycloak 是一个开源的身份和访问管理解决方案**，由 RedHat 管理，JBoss 用 Java 开发。

在本教程中，我们将学习**如何设置一个嵌入在 Spring Boot 应用**中的 Keycloak 服务器。这使得启动预先配置的 Keycloak 服务器变得很容易。

Keycloak 也可以作为一个独立的服务器运行，但是它需要通过管理控制台下载和设置。

## 2。键盘锁预配置

首先，让我们了解如何预配置一个 Keycloak 服务器。

该服务器包含一组领域，每个领域作为一个独立的用户管理单元。要预先配置它，我们需要指定一个 JSON 格式的领域定义文件。

使用 [Keycloak 管理控制台](https://web.archive.org/web/20220809012715/https://www.keycloak.org/docs/6.0/server_admin/#admin-console)可以配置的一切都保存在这个 JSON 中。

我们的授权服务器将预先配置有`baeldung-realm.json`。让我们看看文件中的一些相关配置:

*   `users`:我们默认的用户是`[[email protected]](/web/20220809012715/https://www.baeldung.com/cdn-cgi/l/email-protection)`和`[[email protected]](/web/20220809012715/https://www.baeldung.com/cdn-cgi/l/email-protection)`；他们在这里也会有证件
*   `clients`:我们将定义一个 id 为`newClient`的客户端
*   `standardFlowEnabled`:设置为真，激活`newClient`的授权码流
*   `redirectUris` : `newClient`认证成功后服务器将重定向到的 URL 在此列出
*   `webOrigins`:设置为`“+”`，允许 CORS 支持所有列为`redirectUris`的网址

默认情况下，Keycloak 服务器发布 JWT 令牌，因此不需要为此进行单独的配置。接下来让我们看看 Maven 的配置。

## 3。Maven 配置

因为我们将在 Spring Boot 应用程序中嵌入 Keycloak，所以没有必要单独下载它。

相反，**我们将设置以下一组依赖关系**:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>        
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

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

注意，这里我们使用的是 Spring Boot 的 2.6.7 版本。为了持久性，添加了依赖关系`spring-boot-starter-data-jpa`和 H2。其他的`springframework.boot`依赖是为了 web 支持，因为我们也需要能够运行 Keycloak 授权服务器以及作为 web 服务的管理控制台。

**我们还需要几个 Keycloak 和 RESTEasy 的依赖项**:

```java
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-jackson2-provider</artifactId>
    <version>3.15.1.Final</version>
</dependency>

<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-dependencies-server-all</artifactId>
    <version>18.0.0</version>
    <type>pom</type>
</dependency> 
```

在 Maven 网站上查看最新版本的 [Keycloak](https://web.archive.org/web/20220809012715/https://mvnrepository.com/artifact/org.keycloak/keycloak-dependencies-server-all) 和 [RESTEasy](https://web.archive.org/web/20220809012715/https://mvnrepository.com/artifact/org.jboss.resteasy/resteasy-jackson2-provider) 。

最后，我们必须覆盖`<infinispan.version>` 属性，使用 Keycloak 声明的版本，而不是 Spring Boot 定义的版本:

```java
<properties>
    <infinispan.version>13.0.8.Final</infinispan.version>
</properties>
```

## 4。嵌入式键盘锁配置

现在让我们为授权服务器定义 Spring 配置:

```java
@Configuration
public class EmbeddedKeycloakConfig {

    @Bean
    ServletRegistrationBean keycloakJaxRsApplication(
      KeycloakServerProperties keycloakServerProperties, DataSource dataSource) throws Exception {

        mockJndiEnvironment(dataSource);
        EmbeddedKeycloakApplication.keycloakServerProperties = keycloakServerProperties;
        ServletRegistrationBean servlet = new ServletRegistrationBean<>(
          new HttpServlet30Dispatcher());
        servlet.addInitParameter("javax.ws.rs.Application", 
          EmbeddedKeycloakApplication.class.getName());
        servlet.addInitParameter(ResteasyContextParameters.RESTEASY_SERVLET_MAPPING_PREFIX,
          keycloakServerProperties.getContextPath());
        servlet.addInitParameter(ResteasyContextParameters.RESTEASY_USE_CONTAINER_FORM_PARAMS, 
          "true");
        servlet.addUrlMappings(keycloakServerProperties.getContextPath() + "/*");
        servlet.setLoadOnStartup(1);
        servlet.setAsyncSupported(true);
        return servlet;
    }

    @Bean
    FilterRegistrationBean keycloakSessionManagement(
      KeycloakServerProperties keycloakServerProperties) {
        FilterRegistrationBean filter = new FilterRegistrationBean<>();
	filter.setName("Keycloak Session Management");
	filter.setFilter(new EmbeddedKeycloakRequestFilter());
	filter.addUrlPatterns(keycloakServerProperties.getContextPath() + "/*");

	return filter;
    }

    private void mockJndiEnvironment(DataSource dataSource) throws NamingException {		 
        NamingManager.setInitialContextFactoryBuilder(
          (env) -> (environment) -> new InitialContext() {
            @Override
            public Object lookup(Name name) {
                return lookup(name.toString());
            }

            @Override
            public Object lookup(String name) {
                if ("spring/datasource".equals(name)) {
                    return dataSource;
                } else if (name.startsWith("java:jboss/ee/concurrency/executor/")) {
                    return fixedThreadPool();
                }
                return null;
            }

            @Override
            public NameParser getNameParser(String name) {
                return CompositeName::new;
            }

            @Override
            public void close() {
            }
        });
    }

    @Bean("fixedThreadPool")
    public ExecutorService fixedThreadPool() {
        return Executors.newFixedThreadPool(5);
    }

    @Bean
    @ConditionalOnMissingBean(name = "springBootPlatform")
    protected SimplePlatformProvider springBootPlatform() {
        return (SimplePlatformProvider) Platform.getPlatform();
    }
} 
```

注意:不要担心编译错误，我们稍后将定义`EmbeddedKeycloakRequestFilter` 类。

正如我们在这里看到的，我们首先用`KeycloakServerProperties`将 Keycloak 配置为一个 JAX-RS 应用程序，用于持久存储我们的领域定义文件中指定的 Keycloak 属性。然后我们添加了一个会话管理过滤器，并模拟了一个 JNDI 环境来使用一个`spring/datasource`，这是我们的内存 H2 数据库。

## 5。 `KeycloakServerProperties`

现在让我们来看看我们刚刚提到的`KeycloakServerProperties`:

```java
@ConfigurationProperties(prefix = "keycloak.server")
public class KeycloakServerProperties {
    String contextPath = "/auth";
    String realmImportFile = "baeldung-realm.json";
    AdminUser adminUser = new AdminUser();

    // getters and setters

    public static class AdminUser {
        String username = "admin";
        String password = "admin";

        // getters and setters        
    }
} 
```

正如我们所看到的，**这是一个简单的 POJO 来设置`contextPath`、`adminUser`和领域定义文件**。

## 6。 `EmbeddedKeycloakApplication`

接下来，让我们看看这个类，它使用我们之前设置的配置来创建领域:

```java
public class EmbeddedKeycloakApplication extends KeycloakApplication {
    private static final Logger LOG = LoggerFactory.getLogger(EmbeddedKeycloakApplication.class);
    static KeycloakServerProperties keycloakServerProperties;

    protected void loadConfig() {
        JsonConfigProviderFactory factory = new RegularJsonConfigProviderFactory();
        Config.init(factory.create()
          .orElseThrow(() -> new NoSuchElementException("No value present")));
    }

    @Override
    protected ExportImportManager bootstrap() {
        final ExportImportManager exportImportManager = super.bootstrap();
        createMasterRealmAdminUser();
        createBaeldungRealm();
        return exportImportManager;
    }

    private void createMasterRealmAdminUser() {
        KeycloakSession session = getSessionFactory().create();
        ApplianceBootstrap applianceBootstrap = new ApplianceBootstrap(session);
        AdminUser admin = keycloakServerProperties.getAdminUser();
        try {
            session.getTransactionManager().begin();
            applianceBootstrap.createMasterRealmUser(admin.getUsername(), admin.getPassword());
            session.getTransactionManager().commit();
        } catch (Exception ex) {
            LOG.warn("Couldn't create keycloak master admin user: {}", ex.getMessage());
            session.getTransactionManager().rollback();
        }
        session.close();
    }

    private void createBaeldungRealm() {
        KeycloakSession session = getSessionFactory().create();
        try {
            session.getTransactionManager().begin();
            RealmManager manager = new RealmManager(session);
            Resource lessonRealmImportFile = new ClassPathResource(
              keycloakServerProperties.getRealmImportFile());
            manager.importRealm(JsonSerialization.readValue(lessonRealmImportFile.getInputStream(),
              RealmRepresentation.class));
            session.getTransactionManager().commit();
        } catch (Exception ex) {
            LOG.warn("Failed to import Realm json file: {}", ex.getMessage());
            session.getTransactionManager().rollback();
        }
        session.close();
    }
} 
```

## 7。定制平台实施

我们说过，Keycloak 是 RedHat/JBoss 开发的。因此，它提供了在 Wildfly 服务器上部署应用程序的功能和扩展库，或者作为 Quarkus 解决方案。

在这种情况下，我们正在远离那些替代品，因此，我们必须为一些特定于平台的接口和类提供定制实现。

例如，在我们刚刚配置的`EmbeddedKeycloakApplication` 中，我们首先加载了 Keycloak 的服务器配置`keycloak-server.json`，使用了抽象`JsonConfigProviderFactory`的一个空子类:

```java
public class RegularJsonConfigProviderFactory extends JsonConfigProviderFactory { }
```

然后，我们扩展了`KeycloakApplication`来创建两个领域:`master`和`baeldung`。这些是根据我们的领域定义文件`baeldung-realm.json`中指定的属性创建的。

正如您所看到的，我们使用一个`KeycloakSession` 来执行所有的事务，为了正常工作，我们必须创建一个自定义的`AbstractRequestFilter` ( [`EmbeddedKeycloakRequestFilter`](https://web.archive.org/web/20220809012715/https://github.com/Baeldung/spring-security-oauth/blob/master/oauth-rest/oauth-authorization-server/src/main/java/com/baeldung/auth/config/EmbeddedKeycloakRequestFilter.java) )并在`EmbeddedKeycloakConfig` 文件中使用`KeycloakSessionServletFilter` 为其设置一个 bean。

此外，我们需要几个**定制提供者，这样我们就有了自己的 [`org.keycloak.common.util.ResteasyProvider`](https://web.archive.org/web/20220809012715/https://github.com/Baeldung/spring-security-oauth/blob/master/oauth-rest/oauth-authorization-server/src/main/java/com/baeldung/auth/config/Resteasy3Provider.java) 和 [`org.keycloak.platform.PlatformProvider`](https://web.archive.org/web/20220809012715/https://github.com/Baeldung/spring-security-oauth/blob/master/oauth-rest/oauth-authorization-server/src/main/java/com/baeldung/auth/config/SimplePlatformProvider.java)** 实现，并且不依赖于外部依赖。

重要的是，关于这些定制提供者的信息应该包含在项目的`META-INF/services`文件夹中，以便它们在运行时被获取。

## 8.将这一切结合在一起

正如我们所见， **Keycloak 已经从应用程序端**大大简化了所需的配置。不需要以编程方式定义数据源或任何安全配置。

为了将这些整合在一起，我们需要定义 Spring 和 Spring Boot 应用程序的配置。

### 8.1.`application.yml`

对于弹簧配置，我们将使用一个简单的 YAML:

```java
server:
  port: 8083

spring:
  datasource:
    username: sa
    url: jdbc:h2:mem:testdb;DB_CLOSE_ON_EXIT=FALSE

keycloak:
  server:
    contextPath: /auth
    adminUser:
      username: bael-admin
      password: ********
    realmImportFile: baeldung-realm.json
```

### 8.2.Spring Boot 应用

最后，这是 Spring Boot 的应用程序:

```java
@SpringBootApplication(exclude = LiquibaseAutoConfiguration.class)
@EnableConfigurationProperties(KeycloakServerProperties.class)
public class AuthorizationServerApp {
    private static final Logger LOG = LoggerFactory.getLogger(AuthorizationServerApp.class);

    public static void main(String[] args) throws Exception {
        SpringApplication.run(AuthorizationServerApp.class, args);
    }

    @Bean
    ApplicationListener<ApplicationReadyEvent> onApplicationReadyEventListener(
      ServerProperties serverProperties, KeycloakServerProperties keycloakServerProperties) {
        return (evt) -> {
            Integer port = serverProperties.getPort();
            String keycloakContextPath = keycloakServerProperties.getContextPath();
            LOG.info("Embedded Keycloak started: http://localhost:{}{} to use keycloak", 
              port, keycloakContextPath);
        };
    }
}
```

值得注意的是，这里我们已经启用了`KeycloakServerProperties`配置来将其注入到`ApplicationListener` bean 中。

运行这个类后，**我们可以访问授权服务器的欢迎页面，网址为 http://localhost:8083/auth/** 。

### 8.3.可执行 JAR

我们还可以[创建一个可执行的 jar 文件](/web/20220809012715/https://www.baeldung.com/deployable-fat-jar-spring-boot)来打包并运行应用程序:

```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <mainClass>com.baeldung.auth.AuthorizationServerApp</mainClass>
        <requiresUnpack>
            <dependency>
                <groupId>org.keycloak</groupId>
                <artifactId>keycloak-connections-jpa</artifactId>
            </dependency>
            <dependency>
                <groupId>org.keycloak</groupId>
                <artifactId>keycloak-model-jpa</artifactId>
            </dependency>
        </requiresUnpack>
    </configuration>
</plugin>
```

在这里，我们已经指定了主类，还指示 Maven 解包一些 Keycloak 依赖项。这个**在运行时从 fat jars 中解包库，现在我们可以使用标准的`java -jar <artifact-name>`命令运行应用程序。**

如前所示，现在可以访问授权服务器的欢迎页面。

## 9.结论

在这个快速教程中，我们看到了如何设置一个嵌入在 Spring Boot 应用程序中的 Keycloak 服务器。GitHub 上的[提供了这个应用的源代码。](https://web.archive.org/web/20220809012715/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-rest/oauth-authorization-server)

这个实现的最初想法是由 Thomas Darimont 开发的，可以在项目[embedded-spring-boot-key cloak-server](https://web.archive.org/web/20220809012715/https://github.com/thomasdarimont/embedded-spring-boot-keycloak-server)中找到。