# Spring Security:探索 JDBC 认证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-jdbc-authentication>

## 1.概观

在这个简短的教程中，我们将探索 Spring 提供的使用现有的 [`DataSource`](/web/20221108144117/https://www.baeldung.com/spring-boot-configure-data-source-programmatic) 配置执行 JDBC 认证的能力。

在我们用数据库支持的 userdailsservice 帖子进行的[认证中，我们分析了一种实现这一点的方法，即通过我们自己实现`UserDetailService `接口。](/web/20221108144117/https://www.baeldung.com/spring-security-authentication-with-a-database)

这一次，我们将利用`AuthenticationManagerBuilder#jdbcAuthentication` 指令来分析这种更简单方法的利弊。

## 2.使用嵌入式 H2 连接

首先，我们将分析如何使用嵌入式 H2 数据库实现身份验证。

这很容易实现，因为大多数 Spring Boot 的自动配置都是为这种情况现成准备的。

### 2.1.依赖性和数据库配置

让我们首先按照我们之前的 [Spring Boot 与 H2 数据库](/web/20221108144117/https://www.baeldung.com/spring-boot-h2-database)帖子的说明:

1.  包括相应的`spring-boot-starter-data-jpa `和`h2`依赖关系
2.  使用应用程序属性配置数据库连接
3.  启用 H2 控制台

### 2.2.配置 JDBC 身份验证

**我们将使用 [Spring Security 的`AuthenticationManagerBuilder` 配置助手](https://web.archive.org/web/20221108144117/https://spring.io/guides/topicals/spring-security-architecture#_customizing_authentication_managers)来配置 JDBC 认证:**

```
@Autowired
private DataSource dataSource;

@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth)
  throws Exception {
    auth.jdbcAuthentication()
      .dataSource(dataSource)
      .withDefaultSchema()
      .withUser(User.withUsername("user")
        .password(passwordEncoder().encode("pass"))
        .roles("USER"));
}

@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

正如我们所看到的，我们正在使用自动配置的`DataSource.` 。`withDefaultSchema`指令添加了一个将填充默认模式的数据库脚本，允许存储用户和权限。

这个基本的用户模式记录在 Spring 安全附录中。

最后，我们以编程方式用默认用户在数据库中创建一个条目。

### 2.3.验证配置

让我们创建一个非常简单的端点来检索经过身份验证的*主体*信息:

```
@RestController
@RequestMapping("/principal")
public class UserController {

    @GetMapping
    public Principal retrievePrincipal(Principal principal) {
        return principal;
    }
}
```

此外，我们将保护该端点，同时允许访问 H2 控制台:

```
@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity httpSecurity)
      throws Exception {
        httpSecurity.authorizeRequests()
          .antMatchers("/h2-console/**")
          .permitAll()
          .anyRequest()
          .authenticated()
          .and()
          .formLogin();

        httpSecurity.csrf()
          .ignoringAntMatchers("/h2-console/**");
        httpSecurity.headers()
          .frameOptions()
          .sameOrigin();
      return http.build();
    }
}
```

注意:这里我们复制了由 Spring Boot 实现的安全配置，但是在真实场景中，我们可能根本不会启用 H2 控制台。

现在，我们将运行应用程序并浏览 H2 控制台。我们可以验证 **Spring 正在我们的嵌入式数据库中创建两个表:`users` 和`authorities.`**

它们的结构对应于我们之前提到的 Spring 安全附录中定义的结构。

最后，让我们验证并请求`/principal` 端点查看相关信息，包括用户详细信息。

### 2.4.在后台

在这篇文章的开始，我们提供了一个教程的链接，解释了如何使用`UserDetailsService `接口定制数据库支持的认证；如果我们想了解事情是如何进行的，我们强烈建议看一下这篇文章。

**在这种情况下，我们依赖于 Spring Security 提供的相同接口的实现；`JdbcDaoImpl`。**

如果我们研究这个类，我们将看到它使用的`UserDetails `实现，以及从数据库中检索用户信息的机制。

对于这个简单的场景来说，这样做很好，但是如果我们想要定制数据库模式，或者即使我们想要使用不同的数据库供应商，这样做也有一些缺点。

让我们看看如果我们更改配置以使用不同的 JDBC 服务会发生什么。

## 3.为不同的数据库调整模式

在这一节中，我们将使用 MySQL 数据库在我们的项目上配置身份验证。

正如我们接下来将看到的，为了实现这一点，我们需要避免使用默认模式，并提供我们自己的模式。

### 3.1.依赖性和数据库配置

首先，让我们移除`h2 `依赖项，并将其替换为相应的 MySQL 库:

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.17</version>
</dependency>
```

和往常一样，我们可以在 [Maven Central](https://web.archive.org/web/20221108144117/https://search.maven.org/search?q=a:mysql-connector-java%20g:mysql) 中查找最新版本的库。

现在，让我们相应地重新设置应用程序属性:

```
spring.datasource.url=
  jdbc:mysql://localhost:3306/jdbc_authentication
spring.datasource.username=root
spring.datasource.password=pass
```

### 3.2.运行默认配置

当然，这些应该定制为连接到您正在运行的 MySQL 服务器。出于测试目的，这里我们将使用 Docker 启动一个新实例:

```
docker run -p 3306:3306
  --name bael-mysql
  -e MYSQL_ROOT_PASSWORD=pass
  -e MYSQL_DATABASE=jdbc_authentication
  mysql:latest
```

现在让我们运行这个项目，看看默认配置是否适合 MySQL 数据库。

实际上，由于一个`SQLSyntaxErrorException`，应用程序将无法启动。这其实是有道理的；正如我们所说的，大多数默认的自动配置适用于 HSQLDB。

在这种情况下，**指令提供的 DDL 脚本使用了一种不适合 MySQL 的方言。**

因此，我们需要避免使用这种模式，并提供我们自己的模式。

### 3.3.调整身份验证配置

由于我们不想使用默认模式，我们必须从`AuthenticationManagerBuilder`配置中删除适当的语句。

此外，由于我们将提供自己的 SQL 脚本，我们可以避免尝试以编程方式创建用户:

```
@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth)
  throws Exception {
    auth.jdbcAuthentication()
      .dataSource(dataSource);
}
```

现在让我们看一下数据库初始化脚本。

首先，我们的`schema.sql`:

```
CREATE TABLE users (
  username VARCHAR(50) NOT NULL,
  password VARCHAR(100) NOT NULL,
  enabled TINYINT NOT NULL DEFAULT 1,
  PRIMARY KEY (username)
);

CREATE TABLE authorities (
  username VARCHAR(50) NOT NULL,
  authority VARCHAR(50) NOT NULL,
  FOREIGN KEY (username) REFERENCES users(username)
);

CREATE UNIQUE INDEX ix_auth_username
  on authorities (username,authority);
```

然后，我们的`data.sql`:

```
-- User user/pass
INSERT INTO users (username, password, enabled)
  values ('user',
    '$2a$10$8.UnVuG9HHgffUDAlk8qfOuVGkqRzgVymGe07xd00DMxs.AQubh4a',
    1);

INSERT INTO authorities (username, authority)
  values ('user', 'ROLE_USER');
```

最后，我们应该修改一些其他的应用程序属性:

*   因为我们不希望 Hibernate 现在创建模式，所以我们应该禁用`ddl-auto`属性
*   默认情况下，Spring Boot 只为嵌入式数据库初始化数据源，但这里的情况并非如此:

```
spring.sql.init.mode=always
spring.jpa.hibernate.ddl-auto=none
```

因此，我们现在应该能够正确地启动我们的应用程序，从端点认证和检索`Principal `数据。

另外，请注意,`spring.sql.init.mode`属性是在 Spring Boot 2.5.0 中引入的；对于早期版本，我们需要使用`spring.datasource.initialization-mode.`

## 4.为不同的模式调整查询

让我们更进一步。想象一下默认模式不适合我们的需求。

### 4.1.更改默认模式

例如，假设我们已经有了一个结构与默认结构略有不同的数据库:

```
CREATE TABLE bael_users (
  name VARCHAR(50) NOT NULL,
  email VARCHAR(50) NOT NULL,
  password VARCHAR(100) NOT NULL,
  enabled TINYINT NOT NULL DEFAULT 1,
  PRIMARY KEY (email)
);

CREATE TABLE authorities (
  email VARCHAR(50) NOT NULL,
  authority VARCHAR(50) NOT NULL,
  FOREIGN KEY (email) REFERENCES bael_users(email)
);

CREATE UNIQUE INDEX ix_auth_email on authorities (email,authority);
```

最后，我们的`data.sql`脚本也将适应这一变化:

```
-- User [[email protected]](/web/20221108144117/https://www.baeldung.com/cdn-cgi/l/email-protection)/pass
INSERT INTO bael_users (name, email, password, enabled)
  values ('user',
    '[[email protected]](/web/20221108144117/https://www.baeldung.com/cdn-cgi/l/email-protection)',
    '$2a$10$8.UnVuG9HHgffUDAlk8qfOuVGkqRzgVymGe07xd00DMxs.AQubh4a',
    1);

INSERT INTO authorities (email, authority)
  values ('[[email protected]](/web/20221108144117/https://www.baeldung.com/cdn-cgi/l/email-protection)', 'ROLE_USER');
```

### 4.2.使用新模式运行应用程序

让我们启动我们的应用程序。它初始化正确，这是有意义的，因为我们的模式是正确的。

现在，如果我们尝试登录，我们会发现在出示凭据时会出现错误提示。

Spring Security 仍然在数据库中寻找一个`username `字段。幸运的是，JDBC 认证配置提供了定制用于在认证过程中检索用户详细信息的查询的可能性。

### 4.3.自定义搜索查询

修改查询非常容易。我们只需在配置`AuthenticationManagerBuilder`时提供我们自己的 SQL 语句:

```
@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) 
  throws Exception {
    auth.jdbcAuthentication()
      .dataSource(dataSource)
      .usersByUsernameQuery("select email,password,enabled "
        + "from bael_users "
        + "where email = ?")
      .authoritiesByUsernameQuery("select email,authority "
        + "from authorities "
        + "where email = ?");
}
```

我们可以再次启动应用程序，并使用新的凭证访问`/principal`端点。

## 5.结论

正如我们所看到的，这种方法比创建我们自己的`UserDetailService` 实现简单得多，这意味着一个艰苦的过程；创建实现`UserDetail `接口的实体和类，并将存储库添加到我们的项目中。

当然，缺点是当我们的数据库或逻辑与 Spring 安全解决方案提供的默认策略不同时，它提供的灵活性很小。

最后，我们可以看看 GitHub 库中的完整示例。为了简单起见，我们甚至包括了一个使用 PostgreSQL 的例子，这个例子在本教程中没有展示。