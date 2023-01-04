# Ratpack

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ratpack-groovy>

## 1.概观

Ratpack 是一套轻量级 Java 库，用于构建具有反应、异步和非阻塞特性的可伸缩 HTTP 应用程序。

此外，Ratpack 还提供与技术和框架的集成，如 [Google Guice、](/web/20220701022822/https://www.baeldung.com/ratpack-google-guice)、 [RxJava](/web/20220701022822/https://www.baeldung.com/ratpack-rxjava) 和 [Hystrix](/web/20220701022822/https://www.baeldung.com/ratpack-hystrix) 。

在本教程中，我们将探索如何在 Groovy 中使用 Ratpack。

## 2.为什么这么棒？

Groovy 是一种运行在 JVM 中的强大的动态语言。

因此，Groovy 使得脚本和特定领域语言变得非常容易。有了 Ratpack，这就提供了快速的 web 应用程序开发。

**Ratpack 通过`ratpack-groovy`和`ratpack` `-groovy-test`库提供了与 Groovy 的简单集成。**

## 3.使用 Groovy 脚本的 Ratpack 应用程序

rat pack Groovy API 是用 Java 构建的，因此它们可以很容易地与 Java 和 Groovy 应用程序集成。它们在`ratpack.groovy `包装中提供。

实际上，结合 Groovy 的脚本能力和 Grape 依赖管理，我们可以用几行代码快速创建一个支持 Ratpack 的 web 应用程序:

```
@Grab('io.ratpack:ratpack-groovy:1.6.1')
import static ratpack.groovy.Groovy.ratpack

ratpack {
    handlers {
        get {
            render 'Hello World from Ratpack with Groovy!!'
        }    
    }
}
```

**这是我们的第一个处理程序，**处理一个 GET 请求。我们所要做的就是添加一些基本的 DSL 来启用 Ratpack 服务器。

现在让我们将它作为 Groovy 脚本来运行，以启动应用程序。默认情况下，应用程序将在`[http://localhost:5050](https://web.archive.org/web/20220701022822/http://localhost:5050/)`可用:

```
$ curl -s localhost:5050
Hello World from Ratpack with Groovy!!
```

我们还可以使用`ServerConfig`来配置端口:

```
ratpack {
    serverConfig {
        port(5056)
    }
}
```

**Ratpack 还提供了一个热重载特性**，这意味着我们可以更改`Ratpack.groovy`，然后在应用程序为我们的下一个 HTTP 请求服务时看到这些更改。

## 4.Ratpack-Groovy 依赖管理

有几种方法可以启用`ratpack-groovy`支持。

### 4.1.葡萄

我们可以使用 Groovy 的嵌入式依赖管理器 Grape。

这就像给我们的`Ratpack.groovy`脚本添加注释一样简单:

```
@Grab('io.ratpack:ratpack-groovy:1.6.1')
import static ratpack.groovy.Groovy.ratpack
```

### 4.2.Maven 依赖性

对于在 Maven 中的构建，我们所需要的就是添加对[和`ratpack-groovy`库](https://web.archive.org/web/20220701022822/https://mvnrepository.com/artifact/io.ratpack/ratpack-groovy)的依赖:

```
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-groovy</artifactId>
    <version>${ratpack.version}</version>
</dependency> 
```

### 4.3\. Gradle

我们可以通过在`build.gradle`中添加 Ratpack 的 Groovy Gradle 插件来启用`ratpack-groovy`集成:

```
plugins { 
  id 'io.ratpack.ratpack-groovy' version '1.6.1' 
}
```

## 5.Groovy 中的 Ratpack 处理程序

**处理程序提供了一种处理 web 请求和响应的方式**。可以在这个闭包中访问请求和响应对象。

我们可以使用 HTTP 方法处理 web 请求，比如 GET 和 POST `:`

```
handlers { 
    get("greet/:name") { ctx ->
        render "Hello " + ctx.getPathTokens().get("name") + " !!!"
    }
} 
```

我们可以通过`[http://localhost:5050/greet/<name>](https://web.archive.org/web/20220701022822/http://localhost:5050/greet/Norman)`测试这个 web 请求:

```
$ curl -s localhost:5050/greet/Norman
Hello Norman!!!
```

在处理程序的代码中，`ctx`是`Context`注册表对象，它允许访问路径变量、请求和响应对象。

处理程序也支持通过 Jackson 处理 JSON。

让我们返回从 Groovy 映射转换而来的 JSON:

```
get("data") {
    render Jackson.json([title: "Mr", name: "Norman", country: "USA"])
} 
```

```
$ curl -s localhost:5050/data
{"title":"Mr","name":"Norman","country":"USA"}
```

这里，`Jackson.json` 用于进行转换。

## 6.Ratpack 承诺 Groovy

众所周知，Ratpack 支持应用程序中的异步和非阻塞特性。这是通过 [Ratpack 承诺](/web/20220701022822/https://www.baeldung.com/ratpack-http-client)实现的。

承诺类似于 JavaScript 中使用的承诺，有点像 Java [`Future`](/web/20220701022822/https://www.baeldung.com/java-future) 。我们可以把一个`Promise`看作是一个将来可用的值的表示:

```
post("user") {
    Promise<User> user = parse(Jackson.fromJson(User)) 
    user.then { u -> render u.name } 
}
```

这里的最后一个动作是`then`动作，它决定如何处理最终值。在这种情况下，我们返回它作为对帖子的响应。

让我们更详细地理解这段代码。这里，`Jackson.fromJson`使用`ObjectMapper` `User`解析请求体的 JSON。然后，内置`Context`。`parse`方法将其绑定到`Promise`对象。

承诺异步运行。最终执行`then`操作时，返回响应:

```
curl -X POST -H 'Content-type: application/json' --data \
'{"id":3,"title":"Mrs","name":"Jiney Weiber","country":"UK"}' \
http://localhost:5050/employee

Jiney Weiber
```

我们应该注意到 Promise 库非常丰富，允许我们使用像`map`和`flatMap`这样的函数来链接动作。

## 7.与数据库集成

当我们的处理程序必须等待服务时，使用异步处理程序最有好处。让我们通过将 Ratpack 应用程序与 H2 数据库集成来演示这一点。

我们可以使用 Ratpack 的`HikariModule`类，它是 [HikariCP](/web/20220701022822/https://www.baeldung.com/hikaricp) JDBC 连接池的扩展，或者使用 Groovy [S `ql`](https://web.archive.org/web/20220701022822/http://docs.groovy-lang.org/latest/html/api/groovy/sql/Sql.html) 进行数据库集成。

### 7.1.`HikariModule`

要添加 HikariCP 支持，让我们首先在我们的`pom.xml`中添加以下[光](https://web.archive.org/web/20220701022822/https://search.maven.org/search?q=g:io.ratpack%20AND%20a:ratpack-hikari&core=gav)和 [H2](https://web.archive.org/web/20220701022822/https://search.maven.org/search?q=g:com.h2database%20AND%20a:h2&core=gav) maven 依赖项:

```
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-hikari</artifactId>
    <version>${ratpack.version}</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>${h2.version}</version>
</dependency>
```

或者，我们可以将以下依赖项添加到我们的`build.gradle`中:

```
dependencies {
  compile ratpack.dependency('hikari')
  compile "com.h2database:h2:$h2.version"
}
```

现在，我们将在连接池的`bindings`闭包下声明`HikariModule`:

```
import ratpack.hikari.HikariModule

ratpack {
    bindings {
        module(HikariModule) { config ->
            config.dataSourceClassName = 'org.h2.jdbcx.JdbcDataSource'
            config.addDataSourceProperty('URL', 
              "jdbc:h2:mem:devDB;INIT=RUNSCRIPT FROM 'classpath:/User.sql'")
        }
    }
} 
```

最后，我们准备使用 Java 的`Connection`和`PreparedStatement`来使用它进行简单的数据库操作:

```
get('fetchUserName/:id') { Context ctx ->
    Connection connection = ctx.get(DataSource.class).getConnection()
    PreparedStatement queryStatement = 
      connection.prepareStatement("SELECT NAME FROM USER WHERE ID=?")
    queryStatement.setInt(1, Integer.parseInt(ctx.getPathTokens().get("id")))
    ResultSet resultSet = queryStatement.executeQuery()
    resultSet.next()
    render resultSet.getString(1)  
} 
```

让我们检查处理程序是否按预期工作:

```
$ curl -s localhost:5050/fetchUserName/1
Norman Potter
```

### 7.2.Groovy `Sql` 类

我们可以使用 Groovy `Sql`进行快速数据库操作，通过像`rows`和`executeInsert`这样的方法:

```
get('fetchUsers') {
    def db = [url:'jdbc:h2:mem:devDB']
    def sql = Sql.newInstance(db.url, db.user, db.password)
    def users = sql.rows("SELECT * FROM USER");
    render(Jackson.json(users))
} 
```

```
$ curl -s localhost:5050/fetchUsers
[{"ID":1,"TITLE":"Mr","NAME":"Norman Potter","COUNTRY":"USA"},
{"ID":2,"TITLE":"Miss","NAME":"Ketty Smith","COUNTRY":"FRANCE"}]
```

让我们用`Sql`写一个 HTTP POST 例子:

```
post('addUser') {
    parse(Jackson.fromJson(User))
        .then { u ->
            def db = [url:'jdbc:h2:mem:devDB']
            Sql sql = Sql.newInstance(db.url, db.user, db.password)
            sql.executeInsert("INSERT INTO USER VALUES (?,?,?,?)", 
              [u.id, u.title, u.name, u.country])
            render "User $u.name inserted"
        }
}
```

```
$ curl -X POST -H 'Content-type: application/json' --data \
'{"id":3,"title":"Mrs","name":"Jiney Weiber","country":"UK"}' \
http://localhost:5050/addUser

User Jiney Weiber inserted
```

## 8.单元测试

### 8.1.设置测试

如前所述，Ratpack 还提供了[`ratpack``-groovy-test`库](https://web.archive.org/web/20220701022822/https://search.maven.org/search?q=g:io.ratpack%20AND%20a:ratpack-groovy-test&core=gav)，用于**测试`ratpack-groovy`应用。**

要使用它，我们可以将它作为 Maven 依赖项添加到我们的`pom.xml`:

```
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-groovy-test</artifactId>
    <version>1.6.1</version>
</dependency>
```

或者，我们可以在我们的`build.gradle`中添加 Gradle 依赖关系:

```
testCompile ratpack.dependency('groovy-test')
```

然后我们需要创建一个 Groovy 主类`RatpackGroovyApp.groovy`来测试`Ratpack.groovy`脚本。

```
public class RatpackGroovyApp {
    public static void main(String[] args) {
        File file = new File("src/main/groovy/com/baeldung/Ratpack.groovy");
        def shell = new GroovyShell()  
        shell.evaluate(file)
    }
}
```

当将 Groovy 测试作为 JUnit 测试运行时，**类将使用`GroovyShell`调用`Ratpack.groovy`脚本。**接下来，它将启动 Ratpack 服务器进行测试。

现在，让我们编写 Groovy 测试类`RatpackGroovySpec.groovy`以及通过`RatpackGroovyApp`启动 Ratpack 服务器的代码:

```
class RatpackGroovySpec {
    ServerBackedApplicationUnderTest ratpackGroovyApp = 
      new MainClassApplicationUnderTest(RatpackGroovyApp.class)
    @Delegate TestHttpClient client = 
      TestHttpClient.testHttpClient(ratpackGroovyApp)
}
```

Ratpack 提供了`MainClassApplicationUnderTest` 来模拟启动服务器的应用程序类。

### 8.2.编写我们的测试

让我们编写测试，从一个非常基本的测试开始，检查应用程序是否可以启动:

```
@Test
void "test if app is started"() {
    when:
    get("")

    then:
    assert response.statusCode == 200
    assert response.body.text == 
      "Hello World from Ratpack with Groovy!!"
}
```

现在让我们编写另一个测试来验证`fetchUsers` get 处理程序的响应:

```
@Test
void "test fetchUsers"() {
    when:
    get("fetchUsers")

    then:
    assert response.statusCode == 200
    assert response.body.text == 
      '[{"ID":1,"TITLE":"Mr","NAME":"Norman Potter","COUNTRY":"USA"},{"ID":2,"TITLE":"Miss","NAME":"Ketty Smith","COUNTRY":"FRANCE"}]'
}
```

Ratpack 测试框架负责为我们启动和停止服务器。

## 9.结论

在本文中，我们看到了几种使用 Groovy 为 Ratpack 编写 HTTP 处理程序的方法。我们还探讨了承诺和数据库集成。

我们已经看到了 Groovy 闭包、DSL 和 Groovy 的`Sql`如何使我们的代码简洁、高效和可读。同时，Groovy 的测试支持使得单元和集成测试变得简单明了。

有了这些技术，我们可以使用 Groovy 的动态语言特性和富于表现力的 API，通过 Ratpack 快速开发高性能、可伸缩的 HTTP 应用程序。

像往常一样，示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220701022822/https://github.com/eugenp/tutorials/tree/master/ratpack)