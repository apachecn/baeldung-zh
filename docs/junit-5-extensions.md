# JUnit 5 扩展指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5-extensions>

## 1。概述

在本文中，我们将看看 JUnit 5 测试库中的扩展模型。顾名思义，**Junit 5 扩展的目的是扩展测试类或方法**的行为，这些可以在多个测试中重用。

在 Junit 5 之前，JUnit 4 版本的库使用两种类型的组件来扩展测试:测试运行器和规则。相比之下，JUnit 5 通过引入一个概念简化了扩展机制:`Extension` API。

## 2。JUnit 5 扩展模型

JUnit 5 扩展与测试执行中的某个事件相关，称为扩展点。当到达某个生命周期阶段时，JUnit 引擎调用注册的扩展。

可以使用五种主要类型的扩展点:

*   测试实例后处理
*   条件测试执行
*   生命周期回调
*   参数分辨率
*   异常处理

我们将在接下来的小节中更详细地讨论每一个问题。

## 3。Maven 依赖关系

首先，让我们添加例子中需要的项目依赖项。我们需要的主要 JUnit 5 库是`junit-jupiter-engine`:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
```

同样，让我们也添加两个助手库用于我们的示例:

```java
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.8.2</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.196</version>
</dependency>
```

最新版本的 [junit-jupiter-engine](https://web.archive.org/web/20220628060626/https://search.maven.org/classic/#search%7Cga%7C1%7Cjunit%20jupiter%20engine) 、 [h2](https://web.archive.org/web/20220628060626/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22h2%22%20AND%20g%3A%22com.h2database%22) 和 [log4j-core](https://web.archive.org/web/20220628060626/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22log4j-core%22%20AND%20g%3A%22org.apache.logging.log4j%22) 可以从 Maven Central 下载。

## 4。创建 JUnit 5 扩展

为了创建 JUnit 5 扩展，我们需要定义一个类，该类实现一个或多个与 JUnit 5 扩展点相对应的接口。所有这些接口都扩展了主`Extension`接口，它只是一个标记接口。

### 4.1。`TestInstancePostProcessor`扩展

这种类型的扩展是在创建了一个测试实例之后执行的。要实现的接口是`TestInstancePostProcessor`，它有一个要覆盖的 `postProcessTestInstance()`方法。

该扩展的一个典型用例是将依赖项注入实例。例如，让我们创建一个扩展，它实例化一个`logger`对象，然后在测试实例上调用`setLogger()`方法:

```java
public class LoggingExtension implements TestInstancePostProcessor {

    @Override
    public void postProcessTestInstance(Object testInstance, 
      ExtensionContext context) throws Exception {
        Logger logger = LogManager.getLogger(testInstance.getClass());
        testInstance.getClass()
          .getMethod("setLogger", Logger.class)
          .invoke(testInstance, logger);
    }
}
```

从上面可以看出，`postProcessTestInstance()`方法提供了对测试实例的访问，并使用反射机制调用测试类的`setLogger()`方法。

### 4.2。条件测试执行

JUnit 5 提供了一种扩展，可以控制是否应该运行测试。这是通过实现`ExecutionCondition`接口来定义的。

让我们创建一个`EnvironmentExtension`类来实现这个接口并覆盖`evaluateExecutionCondition()`方法。

该方法验证表示当前环境名称的属性是否等于`“qa”`，并在这种情况下禁用测试:

```java
public class EnvironmentExtension implements ExecutionCondition {

    @Override
    public ConditionEvaluationResult evaluateExecutionCondition(
      ExtensionContext context) {

        Properties props = new Properties();
        props.load(EnvironmentExtension.class
          .getResourceAsStream("application.properties"));
        String env = props.getProperty("env");
        if ("qa".equalsIgnoreCase(env)) {
            return ConditionEvaluationResult
              .disabled("Test disabled on QA environment");
        }

        return ConditionEvaluationResult.enabled(
          "Test enabled on QA environment");
    }
}
```

因此，注册这个扩展的测试将不会在`“qa”`环境中运行。

**如果我们不想验证某个条件，我们可以通过将 `junit.conditions.deactivate`配置键**设置为与该条件匹配的模式来禁用它。

这可以通过使用`-Djunit.conditions.deactivate=<pattern>`属性启动 JVM，或者通过向`LauncherDiscoveryRequest`添加一个配置参数来实现:

```java
public class TestLauncher {
    public static void main(String[] args) {
        LauncherDiscoveryRequest request
          = LauncherDiscoveryRequestBuilder.request()
          .selectors(selectClass("com.baeldung.EmployeesTest"))
          .configurationParameter(
            "junit.conditions.deactivate", 
            "com.baeldung.extensions.*")
          .build();

        TestPlan plan = LauncherFactory.create().discover(request);
        Launcher launcher = LauncherFactory.create();
        SummaryGeneratingListener summaryGeneratingListener
          = new SummaryGeneratingListener();
        launcher.execute(
          request, 
          new TestExecutionListener[] { summaryGeneratingListener });

        System.out.println(summaryGeneratingListener.getSummary());
    }
}
```

### 4.3。生命周期回调

这组扩展与测试生命周期中的事件相关，可以通过实现以下接口来定义:

*   `BeforeAllCallback`和`AfterAllCallback`——在所有测试方法执行之前和之后执行
*   `BeforeEachCallBack`和`AfterEachCallback`–在每种测试方法之前和之后执行
*   `BeforeTestExecutionCallback`和`AfterTestExecutionCallback`–在测试方法之前和之后立即执行

如果测试也定义了它的生命周期方法，执行的顺序是:

1.  `BeforeAllCallback`
2.  `BeforeAll`
3.  `BeforeEachCallback`
4.  `BeforeEach`
5.  `BeforeTestExecutionCallback`
6.  `Test`
7.  `AfterTestExecutionCallback`
8.  `AfterEach`
9.  `AfterEachCallback`
10.  毕竟
11.  `AfterAllCallback`

对于我们的例子，让我们定义一个类来实现这些接口，并控制使用 JDBC 访问数据库的测试行为。

首先，让我们创建一个简单的`Employee`实体:

```java
public class Employee {

    private long id;
    private String firstName;
    // constructors, getters, setters
}
```

我们还需要一个基于`.properties`文件创建`Connection`的实用程序类:

```java
public class JdbcConnectionUtil {

    private static Connection con;

    public static Connection getConnection() 
      throws IOException, ClassNotFoundException, SQLException{
        if (con == null) {
            // create connection
            return con;
        }
        return con;
    }
}
```

最后，让我们添加一个简单的基于 JDBC 的操作`Employee`记录的`DAO`:

```java
public class EmployeeJdbcDao {
    private Connection con;

    public EmployeeJdbcDao(Connection con) {
        this.con = con;
    }

    public void createTable() throws SQLException {
        // create employees table
    }

    public void add(Employee emp) throws SQLException {
       // add employee record
    }

    public List<Employee> findAll() throws SQLException {
       // query all employee records
    }
}
```

**让我们创建实现一些生命周期接口的扩展:**

```java
public class EmployeeDatabaseSetupExtension implements 
  BeforeAllCallback, AfterAllCallback, BeforeEachCallback, AfterEachCallback {
    //...
}
```

每个接口都包含一个我们需要覆盖的方法。

对于`BeforeAllCallback`接口，我们将覆盖`beforeAll()`方法，并在执行任何测试方法之前添加逻辑来创建我们的`employees`表:

```java
private EmployeeJdbcDao employeeDao = new EmployeeJdbcDao();

@Override
public void beforeAll(ExtensionContext context) throws SQLException {
    employeeDao.createTable();
}
```

接下来，我们将利用`BeforeEachCallback`和`AfterEachCallback`在一个事务中包装每个测试方法。这样做的目的是回滚对测试方法中执行的数据库的任何更改，以便下一个测试将在一个干净的数据库上运行。

在`beforeEach()`方法中，我们将创建一个保存点，用于将数据库的状态回滚到:

```java
private Connection con = JdbcConnectionUtil.getConnection();
private Savepoint savepoint;

@Override
public void beforeEach(ExtensionContext context) throws SQLException {
    con.setAutoCommit(false);
    savepoint = con.setSavepoint("before");
}
```

然后，在`afterEach()`方法中，我们将回滚在测试方法执行期间所做的数据库更改:

```java
@Override
public void afterEach(ExtensionContext context) throws SQLException {
    con.rollback(savepoint);
}
```

为了关闭连接，我们将使用在所有测试完成后执行的`afterAll()`方法:

```java
@Override
public void afterAll(ExtensionContext context) throws SQLException {
    if (con != null) {
        con.close();
    }
}
```

### 4.4。参数分辨率

如果一个测试构造函数或方法接收到一个参数，这必须在运行时由一个`ParameterResolver`来解决。

让我们定义我们自己的自定义`ParameterResolver`来解析类型`EmployeeJdbcDao`的参数:

```java
public class EmployeeDaoParameterResolver implements ParameterResolver {

    @Override
    public boolean supportsParameter(ParameterContext parameterContext, 
      ExtensionContext extensionContext) throws ParameterResolutionException {
        return parameterContext.getParameter().getType()
          .equals(EmployeeJdbcDao.class);
    }

    @Override
    public Object resolveParameter(ParameterContext parameterContext, 
      ExtensionContext extensionContext) throws ParameterResolutionException {
        return new EmployeeJdbcDao();
    }
}
```

我们的解析器实现了`ParameterResolver`接口并覆盖了`supportsParameter()`和 `resolveParameter()`方法。第一个验证参数的类型，而第二个定义获取参数实例的逻辑。

### 4.5。异常处理

最后但同样重要的是，`TestExecutionExceptionHandler`接口可以用来定义测试在遇到特定类型的异常时的行为。

例如，我们可以创建一个扩展，它将记录并忽略类型`FileNotFoundException`的所有异常，同时重新抛出任何其他类型:

```java
public class IgnoreFileNotFoundExceptionExtension 
  implements TestExecutionExceptionHandler {

    Logger logger = LogManager
      .getLogger(IgnoreFileNotFoundExceptionExtension.class);

    @Override
    public void handleTestExecutionException(ExtensionContext context,
      Throwable throwable) throws Throwable {

        if (throwable instanceof FileNotFoundException) {
            logger.error("File not found:" + throwable.getMessage());
            return;
        }
        throw throwable;
    }
}
```

## 5。注册扩展

既然我们已经定义了测试扩展，我们需要用 JUnit 5 测试注册它们。为了实现这一点，我们可以使用`@ExtendWith`注释。

注释可以被多次添加到一个测试中，或者接收一个扩展列表作为参数:

```java
@ExtendWith({ EnvironmentExtension.class, 
  EmployeeDatabaseSetupExtension.class, EmployeeDaoParameterResolver.class })
@ExtendWith(LoggingExtension.class)
@ExtendWith(IgnoreFileNotFoundExceptionExtension.class)
public class EmployeesTest {
    private EmployeeJdbcDao employeeDao;
    private Logger logger;

    public EmployeesTest(EmployeeJdbcDao employeeDao) {
        this.employeeDao = employeeDao;
    }

    @Test
    public void whenAddEmployee_thenGetEmployee() throws SQLException {
        Employee emp = new Employee(1, "john");
        employeeDao.add(emp);
        assertEquals(1, employeeDao.findAll().size());   
    }

    @Test
    public void whenGetEmployees_thenEmptyList() throws SQLException {
        assertEquals(0, employeeDao.findAll().size());   
    }

    public void setLogger(Logger logger) {
        this.logger = logger;
    }
}
```

我们可以看到我们的测试类有一个带`EmployeeJdbcDao`参数的构造函数，它将通过扩展`EmployeeDaoParameterResolver`扩展来解析。

通过添加`EnvironmentExtension`，我们的测试将只在不同于`“qa”`的环境中执行。

我们的测试还将创建`employees`表，并通过添加`EmployeeDatabaseSetupExtension`将每个方法包装在一个事务中。即使首先执行了`whenAddEmployee_thenGetEmploee()`测试，向表中添加了一条记录，第二次测试也会在表中找到 0 条记录。

通过使用`LoggingExtension`，一个 logger 实例将被添加到我们的类中。

最后，我们的测试类将忽略所有的`FileNotFoundException`实例，因为它正在添加相应的扩展。

### 5.1。自动延期登记

如果我们想为我们应用程序中的所有测试注册一个扩展，我们可以通过将完全限定名添加到`/META-INF/services/org.junit.jupiter.api.extension.Extension`文件中来实现:

```java
com.baeldung.extensions.LoggingExtension
```

为了启用这种机制，我们还需要将`junit.jupiter.extensions.autodetection.enabled`配置键设置为 true。这可以通过使用–`Djunit.jupiter.extensions.autodetection.enabled=true` 属性启动 JVM，或者通过向`LauncherDiscoveryRequest`添加一个配置参数来实现:

```java
LauncherDiscoveryRequest request
  = LauncherDiscoveryRequestBuilder.request()
  .selectors(selectClass("com.baeldung.EmployeesTest"))
  .configurationParameter("junit.jupiter.extensions.autodetection.enabled", "true")
.build();
```

### 5.2。程序化扩展注册

虽然使用注释注册扩展是一种更具声明性和不唐突的方法，但它有一个显著的缺点:**我们不能容易地定制扩展行为**。例如，对于当前的扩展注册模型，我们不能接受来自客户端的数据库连接属性。

除了基于声明性注释的方法，JUnit 还提供了一个 API 来注册扩展`p` `rogrammatically. `例如，我们可以改进`JdbcConnectionUtil `类来接受连接属性:

```java
public class JdbcConnectionUtil {

    private static Connection con;

    // no-arg getConnection

    public static Connection getConnection(String url, String driver, String username, String password) {
        if (con == null) {
            // create connection 
            return con;
        }

        return con;
    }
}
```

此外，我们应该为`EmployeeDatabaseSetupExtension `扩展添加一个新的构造函数，以支持定制的数据库属性:

```java
public EmployeeDatabaseSetupExtension(String url, String driver, String username, String password) {
    con = JdbcConnectionUtil.getConnection(url, driver, username, password);
    employeeDao = new EmployeeJdbcDao(con);
}
```

**现在，为了用定制的数据库属性注册雇员扩展，我们应该用@ `RegisterExtension `注释:**注释一个静态字段

```java
@ExtendWith({EnvironmentExtension.class, EmployeeDaoParameterResolver.class})
public class ProgrammaticEmployeesUnitTest {

    private EmployeeJdbcDao employeeDao;

    @RegisterExtension 
    static EmployeeDatabaseSetupExtension DB =
      new EmployeeDatabaseSetupExtension("jdbc:h2:mem:AnotherDb;DB_CLOSE_DELAY=-1", "org.h2.Driver", "sa", "");

    // same constrcutor and tests as before
}
```

在这里，我们连接到内存中的 H2 数据库来运行测试。

### 5.3。注册顺序

**JUnit 在注册使用`@ExtendsWith`注释声明性定义的扩展之后，注册`@RegisterExtension `静态字段。**我们也可以使用非静态字段进行编程注册，但是它们将在测试方法实例化和后处理器之后注册。

如果我们通过`@RegisterExtension`以编程方式注册多个扩展，JUnit 将按照确定的顺序注册这些扩展。尽管排序是确定的，但是用于排序的算法是不明显的和内部的。为了**强制执行特定的注册顺序，我们可以使用`[@Order](https://web.archive.org/web/20220628060626/https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/Order.html) `注释:**

```java
public class MultipleExtensionsUnitTest {

    @Order(1) 
    @RegisterExtension 
    static EmployeeDatabaseSetupExtension SECOND_DB = // omitted

    @Order(0)
    @RegisterExtension     
    static EmployeeDatabaseSetupExtension FIRST_DB = // omitted

    @RegisterExtension     
    static EmployeeDatabaseSetupExtension LAST_DB = // omitted

    // omitted
}
```

这里，扩展是基于优先级排序的**，其中较低的值比较高的值**具有更高的优先级。同样，没有`@Order `注释的扩展可能具有最低的优先级。

## 6。结论

在本教程中，我们展示了如何利用 JUnit 5 扩展模型来创建定制的测试扩展。

这些例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628060626/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5)