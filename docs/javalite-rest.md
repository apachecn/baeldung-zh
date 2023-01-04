# JavaLite 指南——构建 RESTful CRUD 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javalite-rest>

## 1。简介

**[JavaLite](https://web.archive.org/web/20220525134141/http://javalite.io/) 是一个框架集合，用于简化每个开发人员在构建应用程序时必须处理的常见任务**。

在本教程中，我们将着眼于构建一个简单的 API。

## 2。设置

在本教程中，我们将创建一个简单的 RESTful CRUD 应用程序。为了做到这一点，**我们将使用 ActiveWeb 和 active JDBC**——JavaLite 集成的两个框架。

那么，让我们开始添加我们需要的第一个依赖项:

```java
<dependency>
    <groupId>org.javalite</groupId>
    <artifactId>activeweb</artifactId>
    <version>1.15</version>
</dependency>
```

ActiveWeb 工件包含了 ActiveJDBC，所以不需要单独添加。请注意，最新的 [activeweb](https://web.archive.org/web/20220525134141/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.javalite%22%20AND%20a%3A%22activeweb%22) 版本可以在 Maven Central 中找到。

我们需要的第二个依赖项是数据库连接器。对于本例，我们将使用 MySQL，因此需要添加:

```java
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.45</version>
</dependency>
```

同样，最新的 [mysql-connector-java](https://web.archive.org/web/20220525134141/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22mysql%22%20AND%20a%3A%22mysql-connector-java%22) 依赖可以在 Maven Central 上找到。

我们必须添加的最后一个依赖项是 JavaLite 特有的:

```java
<plugin>
    <groupId>org.javalite</groupId>
    <artifactId>activejdbc-instrumentation</artifactId>
    <version>1.4.13</version>
    <executions>
        <execution>
            <phase>process-classes</phase>
            <goals>
                <goal>instrument</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

最新的[active JDBC-instrumentation](https://web.archive.org/web/20220525134141/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.javalite%22%20AND%20a%3A%22activejdbc-instrumentation%22)插件也可以在 Maven Central 中找到。

准备好所有这些之后，在开始实体、表和映射之前，我们将确保[支持的数据库](https://web.archive.org/web/20220525134141/http://javalite.io/activejdbc#supported-databases)之一**启动并运行**。如前所述，我们将使用 MySQL。

现在我们准备从对象关系映射开始。

## 3。对象关系映射

### 3.1。测绘和仪器仪表

让我们从**创建一个`Product`类开始，它将是我们的主要实体**:

```java
public class Product {}
```

并且，让我们也为它创建相应的表**:**

```java
CREATE TABLE PRODUCTS (
    id int(11) DEFAULT NULL auto_increment PRIMARY KEY,
    name VARCHAR(128)
);
```

最后，我们可以**修改我们的`Product` 类来进行映射**:

```java
public class Product extends Model {}
```

我们只需要扩展`org.javalite.activejdbc.Model`类。 **ActiveJDBC 从数据库中推断出 DB 模式参数**。由于这个功能，**不需要添加 getters 和 setters 或者任何注释**。

此外，ActiveJDBC 自动识别出需要将`Product` 类映射到`PRODUCTS` 表。它利用英语屈折将模型的单数形式转换为表格的复数形式。是的，它也有例外。

让我们的映射工作还需要最后一样东西:仪器。**插装是 ActiveJDBC** 需要的一个额外步骤，它将允许我们像使用 getters、setters 和 DAO 类方法一样使用我们的`Product`类。

运行检测之后，我们将能够做如下事情:

```java
Product p = new Product();
p.set("name","Bread");
p.saveIt();
```

或者:

```java
List<Product> products = Product.findAll();
```

这就是`activejdbc-instrumentation`插件的用武之地。由于我们的 pom 中已经有了依赖关系，我们应该看到在构建过程中被插装的类:

```java
...
[INFO] --- activejdbc-instrumentation:1.4.11:instrument (default) @ javalite ---
**************************** START INSTRUMENTATION ****************************
Directory: ...\tutorials\java-lite\target\classes
Instrumented class: .../tutorials/java-lite/target/classes/app/models/Product.class
**************************** END INSTRUMENTATION ****************************
...
```

接下来，我们将创建一个简单的测试来确保这是可行的。

### 3.2。测试

最后，为了测试我们的映射，我们将遵循三个简单的步骤:打开到数据库的连接，保存新产品并检索它:

```java
@Test
public void givenSavedProduct_WhenFindFirst_ThenSavedProductIsReturned() {

    Base.open(
      "com.mysql.jdbc.Driver",
      "jdbc:mysql://localhost/dbname",
      "user",
      "password");

    Product toSaveProduct = new Product();
    toSaveProduct.set("name", "Bread");
    toSaveProduct.saveIt();

    Product savedProduct = Product.findFirst("name = ?", "Bread");

    assertEquals(
      toSaveProduct.get("name"), 
      savedProduct.get("name"));
}
```

请注意，所有这些(以及更多)都可以通过只有一个空模型和工具来实现。

## 4。控制器

既然我们的映射已经准备好了，我们可以开始考虑我们的应用程序及其 CRUD 方法了。

为此，我们将利用处理 HTTP 请求的控制器。

让我们创建我们的`ProductsController`:

```java
@RESTful
public class ProductsController extends AppController {

    public void index() {
        // ...
    }

}
```

通过这种实现，ActiveWeb 将自动将`index()` 方法映射到以下 URI:

```java
http://<host>:<port>/products
```

用`@RESTful`、**标注的控制器提供了一组固定的方法，自动映射到不同的 URIs。**让我们看看对我们的 CRUD 示例有用的那些:

| **控制器方法** | **HTTP 方法** | **URI** |  |
| 创造 | `create()` | 邮政 | `http://host:port/products` |
| 红色的 | `show()` | 得到 | `http://host:port/products/{id}` |
| 亲爱的大家 | `index()` | 得到 | `http://host:port/products` |
| 更新 | `update()` | 放 | `http://host:port/products/{id}` |
| 删除 | `destroy()` | 删除 | `http://host:port/products/{id}` |

如果我们将这组方法添加到我们的`ProductsController`:

```java
@RESTful
public class ProductsController extends AppController {

    public void index() {
        // code to get all products
    }

    public void create() {
        // code to create a new product
    }

    public void update() {
        // code to update an existing product
    }

    public void show() {
        // code to find one product
    }

    public void destroy() {
        // code to remove an existing product 
    }
}
```

在继续我们的逻辑实现之前，我们将快速看一下我们需要配置的一些东西。

## 5。配置

ActiveWeb 主要基于约定，项目结构就是一个例子。 **ActiveWeb 项目需要遵循预定义的包布局**:

```java
src
 |----main
       |----java.app
       |     |----config
       |     |----controllers
       |     |----models
       |----resources
       |----webapp
             |----WEB-INF
             |----views
```

我们需要查看一个特定的包——a`pp.config`。

在这个包中，我们将创建三个类:

```java
public class DbConfig extends AbstractDBConfig {
    @Override
    public void init(AppContext appContext) {
        this.configFile("/database.properties");
    }
}
```

**该类使用项目根目录中的属性文件配置数据库连接**，该文件包含所需的参数:

```java
development.driver=com.mysql.jdbc.Driver
development.username=user
development.password=password
development.url=jdbc:mysql://localhost/dbname
```

这将自动创建连接，取代我们在映射测试的第一行中所做的。

我们需要包含在`app.config` 包中的第二个类是:

```java
public class AppControllerConfig extends AbstractControllerConfig {

    @Override
    public void init(AppContext appContext) {
        add(new DBConnectionFilter()).to(ProductsController.class);
    }
}
```

这段代码 **将把我们刚刚配置的连接绑定到我们的控制器。**

第三个类**将** **配置我们应用的上下文**:

```java
public class AppBootstrap extends Bootstrap {
    public void init(AppContext context) {}
}
```

创建完三个类之后，关于配置的最后一件事是**在`webapp/WEB-INF`目录下创建我们的`web.xml`文件**:

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns=...>

    <filter>
        <filter-name>dispatcher</filter-name>
        <filter-class>org.javalite.activeweb.RequestDispatcher</filter-class>
        <init-param>
            <param-name>exclusions</param-name>
            <param-value>css,images,js,ico</param-value>
        </init-param>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>dispatcher</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

</web-app>
```

现在配置已经完成，我们可以继续添加我们的逻辑。

## 6。实施 CRUD 逻辑

有了我们的`Product` 类提供的类似 DAO 的功能，**添加基本的 CRUD 功能**非常简单:

```java
@RESTful
public class ProductsController extends AppController {

    private ObjectMapper mapper = new ObjectMapper();    

    public void index() {
        List<Product> products = Product.findAll();
        // ...
    }

    public void create() {
        Map payload = mapper.readValue(getRequestString(), Map.class);
        Product p = new Product();
        p.fromMap(payload);
        p.saveIt();
        // ...
    }

    public void update() {
        Map payload = mapper.readValue(getRequestString(), Map.class);
        String id = getId();
        Product p = Product.findById(id);
        p.fromMap(payload);
        p.saveIt();
        // ...
    }

    public void show() {
        String id = getId();
        Product p = Product.findById(id);
        // ...
    }

    public void destroy() {
        String id = getId();
        Product p = Product.findById(id);
        p.delete();
        // ...
    }
}
```

很简单，对吧？然而，这还没有返回任何东西。为了做到这一点，我们必须创建一些视图。

## 7。视图

**ActiveWeb 使用 [FreeMarker](https://web.archive.org/web/20220525134141/http://freemarker.org/) 作为模板引擎，其所有模板都应该位于`src/main/webapp/WEB-INF/views`下。**

在该目录中，我们将视图放在一个名为`products`的文件夹中(与我们的控制器相同)。让我们创建第一个名为`_product.ftl`的模板:

```java
{
    "id" : ${product.id},
    "name" : "${product.name}"
}
```

很明显，这是一个 JSON 响应。当然，这只适用于一个产品，所以让我们继续创建另一个名为`index.ftl`的模板:

```java
[<@render partial="product" collection=products/>]
```

**这将基本上呈现一个名为`products`的集合，每个集合都由`_product.ftl`格式化。**

最后，**我们需要将来自控制器的结果绑定到相应的视图**:

```java
@RESTful
public class ProductsController extends AppController {

    public void index() {
        List<Product> products = Product.findAll();
        view("products", products);
        render();
    }

    public void show() {
        String id = getId();
        Product p = Product.findById(id);
        view("product", p);
        render("_product");
    }
}
```

在第一种情况下，我们将`products` 列表分配给我们的模板集合，也叫做`products`。

然后，因为我们没有指定任何视图，所以将使用`index.ftl` 。

在第二种方法中，我们将产品`p` 分配给视图中的元素`product` ,并明确地指出要呈现哪个视图。

我们还可以创建一个视图`message.ftl`:

```java
{
    "message" : "${message}",
    "code" : ${code}
}
```

然后调用它形成我们的任何一个`ProductsController`的方法:

```java
view("message", "There was an error.", "code", 200);
render("message");
```

现在让我们看看最后的`ProductsController`:

```java
@RESTful
public class ProductsController extends AppController {

    private ObjectMapper mapper = new ObjectMapper();

    public void index() {
        view("products", Product.findAll());
        render().contentType("application/json");
    }

    public void create() {
        Map payload = mapper.readValue(getRequestString(), Map.class);
        Product p = new Product();
        p.fromMap(payload);
        p.saveIt();
        view("message", "Successfully saved product id " + p.get("id"), "code", 200);
        render("message");
    }

    public void update() {
        Map payload = mapper.readValue(getRequestString(), Map.class);
        String id = getId();
        Product p = Product.findById(id);
        if (p == null) {
            view("message", "Product id " + id + " not found.", "code", 200);
            render("message");
            return;
        }
        p.fromMap(payload);
        p.saveIt();
        view("message", "Successfully updated product id " + id, "code", 200);
        render("message");
    }

    public void show() {
        String id = getId();
        Product p = Product.findById(id);
        if (p == null) {
            view("message", "Product id " + id + " not found.", "code", 200);
            render("message");
            return;
        }
        view("product", p);
        render("_product");
    }

    public void destroy() {
        String id = getId();
        Product p = Product.findById(id);
        if (p == null) {
            view("message", "Product id " + id + " not found.", "code", 200);
            render("message");
            return;
        }
        p.delete();
        view("message", "Successfully deleted product id " + id, "code", 200);
        render("message");
    }

    @Override
    protected String getContentType() {
        return "application/json";
    }

    @Override
    protected String getLayout() {
        return null;
    }
}
```

至此，我们的应用程序完成了，我们准备运行它。

## 8。运行应用程序

我们将使用 Jetty 插件:

```java
<plugin>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-maven-plugin</artifactId>
    <version>9.4.8.v20171121</version>
</plugin>
```

在 Maven Central 中找到最新的 [jetty-maven-plugin](https://web.archive.org/web/20220525134141/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.eclipse.jetty%22%20AND%20a%3A%22jetty-maven-plugin%22) 。

我们准备好了，**我们可以运行我们的应用程序了**:

```java
mvn jetty:run
```

让我们创建几个产品:

```java
$ curl -X POST http://localhost:8080/products 
  -H 'content-type: application/json' 
  -d '{"name":"Water"}'
{
    "message" : "Successfully saved product id 1",
    "code" : 200
}
```

```java
$ curl -X POST http://localhost:8080/products 
  -H 'content-type: application/json' 
  -d '{"name":"Bread"}'
{
    "message" : "Successfully saved product id 2",
    "code" : 200
}
```

..阅读它们:

```java
$ curl -X GET http://localhost:8080/products
[
    {
        "id" : 1,
        "name" : "Water"
    },
    {
        "id" : 2,
        "name" : "Bread"
    }
]
```

..更新其中一个:

```java
$ curl -X PUT http://localhost:8080/products/1 
  -H 'content-type: application/json' 
  -d '{"name":"Juice"}'
{
    "message" : "Successfully updated product id 1",
    "code" : 200
}
```

…阅读我们刚刚更新的内容:

```java
$ curl -X GET http://localhost:8080/products/1
{
    "id" : 1,
    "name" : "Juice"
}
```

最后，我们可以删除一个:

```java
$ curl -X DELETE http://localhost:8080/products/2
{
    "message" : "Successfully deleted product id 2",
    "code" : 200
}
```

## 9。结论

JavaLite 有很多工具可以帮助开发人员在几分钟内启动并运行应用程序。然而，虽然基于约定可以产生更干净、更简单的代码，但是理解类、包和文件的命名和位置需要一段时间。

这只是对 ActiveWeb 和 ActiveJDBC 的介绍，可以在他们的网站上找到更多的文档，在 T2 的 Github 项目中寻找我们的产品应用。