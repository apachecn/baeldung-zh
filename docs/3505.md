# 用 Spring 测试一个模拟 JNDI 数据源

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mock-jndi-datasource>

## 1.概观

通常，当测试使用 JNDI 的应用程序时，我们可能希望使用模拟的数据源，而不是真实的数据源。为了使我们的单元测试简单，并且完全独立于任何外部环境，这是测试时的常见做法。

在本教程中，我们将展示如何使用 Spring 框架和简单的 JNDI 库测试一个模拟 JNDI 数据源。

在本教程中，我们将只关注单元测试。但是一定要看看我们关于如何使用 JPA 和 JNDI 数据源创建一个 [Spring 应用程序的文章。](/web/20220630005731/https://www.baeldung.com/spring-persistence-jpa-jndi-datasource)

## 2.JNDI 快速回顾

简而言之， [JNDI](/web/20220630005731/https://www.baeldung.com/jndi) **将逻辑名称绑定到外部资源，比如数据库连接**。主要思想是应用程序不需要知道任何关于已定义数据源的信息，除了它的 JNDI 名称。

简单地说，所有的命名操作都与上下文相关，所以要使用 JNDI 访问命名服务，我们需要首先创建一个`InitialContext`对象。顾名思义，`InitialContext`类**封装了为命名操作提供起点的初始(根)上下文。**

简单地说，根上下文充当入口点。没有它，JNDI 不能绑定或查找我们的资源。

## 3.如何用 Spring 测试 JNDI 数据源

Spring 通过`SimpleNamingContextBuilder`提供了与 JNDI 的开箱即用集成。这个助手类提供了一个很好的方法来模拟 JNDI 环境进行测试。

因此，让我们看看如何使用`SimpleNamingContextBuilder`类对 JNDI 数据源进行单元测试。

首先，我们需要为绑定和检索数据源对象构建一个**初始命名上下文:**

```
@BeforeEach
public void init() throws Exception {
    SimpleNamingContextBuilder.emptyActivatedContextBuilder();
    this.initContext = new InitialContext();
}
```

我们已经使用`emptyActivatedContextBuilder()`方法创建了根上下文，因为它提供了比构造函数更大的灵活性，因为它创建一个新的构建器或返回现有的构建器。

现在我们有了一个上下文，让我们实现一个单元测试，看看如何使用 JNDI 存储和检索一个 JDBC `DataSource` 对象:

```
@Test
public void whenMockJndiDataSource_thenReturnJndiDataSource() throws Exception {
    this.initContext.bind("java:comp/env/jdbc/datasource", 
      new DriverManagerDataSource("jdbc:h2:mem:testdb"));
    DataSource ds = (DataSource) this.initContext.lookup("java:comp/env/jdbc/datasource");

    assertNotNull(ds.getConnection());
}
```

正如我们看到的`,` ，我们使用` bind()`方法将我们的 JDBC `DataSource` 对象映射到名字`java:comp/env/jdbc/datasource`。

然后，我们使用`lookup()`方法从我们的 JNDI 上下文中检索一个`DataSource`引用，该引用使用了我们之前绑定 JDBC `DataSource` 对象时使用的确切逻辑名称。

注意，如果在上下文中找不到指定的对象，JNDI 将简单地抛出一个异常。

值得一提的是，从 Spring 5.2 开始，` SimpleNamingContextBuilder`类被**弃用，取而代之的是其他解决方案，如 [Simple-JNDI](https://web.archive.org/web/20220630005731/https://github.com/h-thurow/Simple-JNDI)** 。

## 4.使用简单 JNDI 模拟和测试 JNDI 数据源

简单——JNDI 允许我们**将属性文件中定义的对象绑定到一个模拟的 JNDI 环境**。它对从 Java EE 容器外的 JNDI 获取类型为 *javax.sql.DataSource* 的对象提供了强大的支持。

因此，让我们看看如何使用它`.`。首先，我们需要将`Simple-JNDI`依赖项添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>com.github.h-thurow</groupId>
    <artifactId>simple-jndi</artifactId>
    <version>0.23.0</version>
</dependency>
```

最新版本的简单 JNDI 库可以在 Maven Central 上找到。

接下来，我们将为简单 JNDI 配置设置 JNDI 上下文所需的所有细节。为此，**我们需要创建一个`jndi.properties`文件，该文件需要放在类路径**中:

```
java.naming.factory.initial=org.osjava.sj.SimpleContextFactory
org.osjava.sj.jndi.shared=true
org.osjava.sj.delimiter=.
jndi.syntax.separator=/
org.osjava.sj.space=java:/comp/env
org.osjava.sj.root=src/main/resources/jndi
```

`java.naming.factory.initial` 指定将用于创建初始上下文的上下文工厂类。

`org.osjava.sj.jndi.shared=true` 意味着所有的`InitialContext`对象将共享相同的内存。

正如我们所看到的，我们使用了`org.osjava.sj.space` 属性将`java:/comp/env`定义为所有 JNDI 查找的起点。

使用`org.osjava.sj.delimiter`和`jndi.syntax.separator` 属性背后的基本思想是避免 [ENC](https://web.archive.org/web/20220630005731/https://github.com/h-thurow/Simple-JNDI/issues/1) 问题。

`org.osjava.sj.root` 属性让我们定义存储属性文件的**路径**。在我们的例子中，所有文件都位于`src/main/resources/jndi` 文件夹下。

因此，让我们在我们的`datasource.properties`文件中定义一个`javax.sql.DataSource`对象:

```
ds.type=javax.sql.DataSource
ds.driver=org.h2.Driver
ds.url=jdbc:jdbc:h2:mem:testdb
ds.user=sa
ds.password=password
```

现在，让我们为单元测试创建一个`InitialContext`对象:

```
@BeforeEach
public void setup() throws Exception {
    this.initContext = new InitialContext();
}
```

最后，我们将实现一个单元测试用例**来检索已经在`datasource.properties`文件**中定义的`DataSource`对象:

```
@Test
public void whenMockJndiDataSource_thenReturnJndiDataSource() throws Exception {
    String dsString = "org.h2.Driver::::jdbc:jdbc:h2:mem:testdb::::sa";
    Context envContext = (Context) this.initContext.lookup("java:/comp/env");
    DataSource ds = (DataSource) envContext.lookup("datasource/ds");

    assertEquals(dsString, ds.toString());
}
```

## 5.结论

在本教程中，我们解释了如何应对在 J2EE 容器之外测试 JNDI 的挑战。我们研究了如何使用 Spring 框架和 Simple-JNDI 库测试模拟 JNDI 数据源。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220630005731/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-persistence-simple)