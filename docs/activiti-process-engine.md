# 活动中的流程引擎配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/activiti-process-engine>

## 1。概述

在我们之前的 [Activiti with Java](/web/20221208143839/https://www.baeldung.com/java-activiti) 介绍文章中，我们看到了`ProcessEngine` 的重要性，并通过框架提供的默认静态 API 创建了一个。

除了默认方式，还有其他创建`ProcessEngine`的方式——我们将在这里探讨。

## 2。获得一个`ProcessEngine` 实例

有两种方法可以获得`ProcessEngine` 的实例:

1.  使用`ProcessEngines`类
2.  以编程方式，通过`ProcessEngineConfiguration`

让我们仔细看看这两种方法的例子。

## 3。使用`ProcessEngines`类获取`ProcessEngine`

通常，`ProcessEngine`是使用一个名为`activiti.cfg.xml,` 的 XML 文件配置的，默认创建过程也将使用这个文件。

下面是这种配置的一个简单示例:

```java
<beans >
    <bean id="processEngineConfiguration" class=
      "org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <property name="jdbcUrl"
          vasentence you have mentioned and also changed thelue="jdbc:h2:mem:activiti;DB_CLOSE_DELAY=1000" />
        <property name="jdbcDriver" value="org.h2.Driver" />
        <property name="jdbcUsername" value="root" />
        <property name="jdbcPassword" value="" />
        <property name="databaseSchemaUpdate" value="true" />
    </bean>
</beans>
```

注意这里引擎的持久性方面是如何配置的。

而现在，我们可以获得`ProcessEngine`:

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
```

## 4。使用`ProcessEngineConfiguration` 获取`ProcessEngine`

越过获取引擎的默认途径——创建`ProcessEngineConfiguration`有两种方法:

1.  使用 XML 配置
2.  使用 Java 配置

让我们从 XML 配置开始。

如第 2.1 节所述。–我们可以通过编程定义`ProcessEngineConfiguration` ，并使用该实例构建`ProcessEngine`:

```java
@Test 
public void givenXMLConfig_whenCreateDefaultConfiguration_thenGotProcessEngine() {
    ProcessEngineConfiguration processEngineConfiguration 
      = ProcessEngineConfiguration
        .createProcessEngineConfigurationFromResourceDefault();
    ProcessEngine processEngine 
      = processEngineConfiguration.buildProcessEngine();

    assertNotNull(processEngine);
    assertEquals("root", processEngine.getProcessEngineConfiguration()
      .getJdbcUsername());
}
```

方法`createProcessEngineConfigurationFromResourceDefault()` 也会寻找`activiti.cfg.xml`文件，现在我们只需要调用 `buildProcessEngine()` API。

在这种情况下，它寻找的默认 bean 名称是`processEngineConfiguration`。如果我们想改变配置文件名或 bean 名，我们可以使用其他可用的方法来创建`ProcessEngineConfiguration.`

让我们来看几个例子。

首先，我们将更改配置文件名，并要求 API 使用我们的自定义文件:

```java
@Test 
public void givenDifferentNameXMLConfig_whenGetProcessEngineConfig_thenGotResult() {
    ProcessEngineConfiguration processEngineConfiguration 
      = ProcessEngineConfiguration
        .createProcessEngineConfigurationFromResource(
          "my.activiti.cfg.xml");
    ProcessEngine processEngine = processEngineConfiguration
      .buildProcessEngine();

    assertNotNull(processEngine);
    assertEquals("baeldung", processEngine.getProcessEngineConfiguration()
      .getJdbcUsername());
}
```

现在，让我们也更改 bean 的名称:

```java
@Test 
public void givenDifferentBeanNameInXMLConfig_whenGetProcessEngineConfig_thenGotResult() {
    ProcessEngineConfiguration processEngineConfiguration 
      = ProcessEngineConfiguration
        .createProcessEngineConfigurationFromResource(
          "my.activiti.cfg.xml", 
          "myProcessEngineConfiguration");
    ProcessEngine processEngine = processEngineConfiguration
      .buildProcessEngine();

    assertNotNull(processEngine);
    assertEquals("baeldung", processEngine.getProcessEngineConfiguration()
      .getJdbcUsername());
}
```

当然，现在配置期望不同的名称，我们需要在运行测试之前更改文件名(和 bean 名称)以匹配。

创建引擎的其他可用选项有`createProcessEngineConfigurationFromInputStream(InputStream inputStream),`
`createProcessEngineConfigurationFromInputStream(InputStream inputStream, String beanName)`。

如果我们不想使用 XML 配置，**我们也可以只使用 Java 配置进行设置**。

我们将和四个不同的班级一起工作。其中每个都代表不同的环境:

1.  `org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration –` `ProcessEngine`以独立的方式使用，由数据库支持
2.  `org.activiti.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration –`默认情况下，使用 H2 内存数据库。当引擎启动和关闭时，DB 被创建和删除——因此，这种配置风格可用于测试
3.  `org.activiti.spring.SpringProcessEngineConfiguration –`用于春季环境
4.  `org.activiti.engine.impl.cfg.JtaProcessEngineConfiguration –`引擎以独立模式运行，具有 JTA 事务

让我们来看几个例子。

下面是一个用于创建独立流程引擎配置的 JUnit 测试:

```java
@Test 
public void givenNoXMLConfig_whenCreateProcessEngineConfig_thenCreated() {
    ProcessEngineConfiguration processEngineConfiguration 
      = ProcessEngineConfiguration
        .createStandaloneProcessEngineConfiguration();
    ProcessEngine processEngine = processEngineConfiguration
      .setDatabaseSchemaUpdate(ProcessEngineConfiguration
        .DB_SCHEMA_UPDATE_TRUE)
      .setJdbcUrl("jdbc:h2:mem:my-own-db;DB_CLOSE_DELAY=1000")
      .buildProcessEngine();

    assertNotNull(processEngine);
    assertEquals("sa", processEngine.getProcessEngineConfiguration()
      .getJdbcUsername());
}
```

类似地，我们将编写一个 JUnit 测试用例，使用内存数据库创建独立的流程引擎配置:

```java
@Test 
public void givenNoXMLConfig_whenCreateInMemProcessEngineConfig_thenCreated() {
    ProcessEngineConfiguration processEngineConfiguration 
      = ProcessEngineConfiguration
      .createStandaloneInMemProcessEngineConfiguration();
    ProcessEngine processEngine = processEngineConfiguration
      .buildProcessEngine();

    assertNotNull(processEngine);
    assertEquals("sa", processEngine.getProcessEngineConfiguration()
      .getJdbcUsername());
}
```

## 5。数据库设置

默认情况下，Activiti API 将使用 H2 内存数据库，数据库名为“Activiti”，用户名为“sa”。

如果我们需要使用任何其他数据库，我们将必须使用两个主要属性显式地进行设置。

`databaseType`–有效值为`h2, mysql, oracle, postgres, mssql, db2`。这也可以从数据库配置中找出，但如果自动检测失败，这将非常有用。

这个属性允许我们定义当引擎启动或关闭时数据库会发生什么。它可以有以下三个值:

1.  `false`(默认)–该选项根据库验证数据库模式的版本。如果不匹配，引擎将抛出异常
2.  `true`–建立流程引擎配置时，对数据库进行检查。将相应地创建/更新数据库创建-删除
3.  "`–`"–这将在创建流程引擎时创建 DB 模式，并在流程引擎关闭时删除它。

我们可以将数据库配置定义为 JDBC 属性:

```java
<property name="jdbcUrl" value="jdbc:h2:mem:activiti;DB_CLOSE_DELAY=1000" />
<property name="jdbcDriver" value="org.h2.Driver" />
<property name="jdbcUsername" value="sa" />
<property name="jdbcPassword" value="" />
<property name="databaseType" value="mysql" /> 
```

或者，如果我们使用`DataSource`:

```java
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" >
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://localhost:3306/activiti" />
    <property name="username" value="activiti" />
    <property name="password" value="activiti" />
    <property name="defaultAutoCommit" value="false" />
    <property name="databaseType" value="mysql" />
</bean> 
```

## 6。结论

在这个快速教程中，我们关注了在 Activiti 中创建`ProcessEngine` 的几种不同方式。

我们还看到了处理数据库配置的不同属性和方法。

和往常一样，我们看到的例子的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143839/https://github.com/eugenp/tutorials/tree/master/spring-activiti)