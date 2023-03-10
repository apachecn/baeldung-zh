# Jakarta EE 中的单独会话 Bean

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ee-singleton-session-bean>

## 1。概述

每当给定用例需要会话 Bean 的单个实例时，我们可以使用单例会话 Bean。

在本教程中，我们将通过一个 Jakarta EE 应用程序的例子来探究这一点。

## 2。肚子

**首先，我们需要在`pom.xml`中定义所需的 Maven 依赖项。**

让我们为 EJB 的部署定义 EJB API 和嵌入式 EJB 容器的依赖关系:

```java
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>8.0</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.apache.openejb</groupId>
    <artifactId>tomee-embedded</artifactId>
    <version>1.7.5</version>
</dependency>
```

最新版本可以在 Maven Central 上的 [JavaEE API](https://web.archive.org/web/20221126220300/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22javax%22%20AND%20a%3A%22javaee-api%22) 和 [tomEE](https://web.archive.org/web/20221126220300/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.tomee.maven%22%20AND%20a%3A%22tomee-embedded-maven-plugin%22) 找到。

## 3。会话 Beans 的类型

有三种类型的会话 Beans。在我们探索单体会话 Beans 之前，让我们看看这三种类型的生命周期之间有什么区别。

### 3.1。有状态会话 bean

有状态会话 Bean 维护与其通信的客户端的对话状态。

每个客户机创建一个新的有状态 Bean 实例，并且不与其他客户机共享。

当客户机和 bean 之间的通信结束时，会话 Bean 也会终止。

### 3.2。无状态会话 bean

无状态会话 Bean 不维护与客户端的任何对话状态。bean 仅在方法调用期间包含特定于客户端的状态。

与有状态会话 Bean 不同，连续的方法调用是独立的。

容器维护一个无状态 Beans 池，这些实例可以在多个客户机之间共享。

### 3.3。单一会话 bean

单一会话 bean 在应用程序的整个生命周期中维护 Bean 的状态。

单例会话 Bean 类似于无状态会话 Bean，但是在整个应用程序中只创建单例会话 Bean 的一个实例，并且在应用程序关闭之前不会终止。

bean 的单个实例在多个客户机之间共享，并且可以被并发访问。

## 4。创建单一会话 Bean

让我们从为它创建一个接口开始。

对于这个例子，让我们使用`javax.ejb.Local`注释来定义接口:

```java
@Local
public interface CountryState {
   List<String> getStates(String country);
   void setStates(String country, List<String> states);
}
```

使用`@Local`意味着在同一个应用程序中访问 bean。我们还可以选择使用`javax.ejb.Remote`注释，这允许我们远程调用 EJB。

现在，我们将定义实现 EJB bean 类。我们通过使用注释 javax `.ejb.Singleton`将该类标记为单例会话 Bean。

此外，让我们用 javax `.ejb.Startup`注释标记 bean，通知 EJB 容器在启动时初始化 bean:

```java
@Singleton
@Startup
public class CountryStateContainerManagedBean implements CountryState {
    ...
}
```

这被称为急切初始化。如果我们不使用`@Startup`，EJB 容器将决定何时初始化 bean。

我们还可以定义多个会话 bean 来初始化数据，并按照特定的顺序加载 bean。因此，我们将使用，`javax.ejb.DependsOn`注释来定义我们的 bean 对其他会话 bean 的依赖性。

`@DependsOn`注释的值是我们的 Bean 所依赖的 Bean 类名的数组:

```java
@Singleton 
@Startup 
@DependsOn({"DependentBean1", "DependentBean2"}) 
public class CountryStateCacheBean implements CountryState { 
    ...
}
```

我们将定义一个初始化 bean 的`initialize()`方法，并使用`javax.annotation.PostConstruct`注释使它成为一个生命周期回调方法。

有了这个注释，容器将在实例化 bean 时调用它:

```java
@PostConstruct
public void initialize() {

    List<String> states = new ArrayList<String>();
    states.add("Texas");
    states.add("Alabama");
    states.add("Alaska");
    states.add("Arizona");
    states.add("Arkansas");

    countryStatesMap.put("UnitedStates", states);
}
```

## 5。并发性

接下来，我们将设计单一会话 Bean 的并发管理。EJB 提供了两种方法来实现对单一会话 Bean 的并发访问:容器管理的并发和 Bean 管理的并发。

注解`javax.ejb.ConcurrencyManagement`为一个方法定义了并发策略。默认情况下，EJB 容器使用容器管理的并发。

`@ConcurrencyManagement`注释接受一个`javax.ejb.ConcurrencyManagementType`值。这些选项包括:

*   `ConcurrencyManagementType.CONTAINER`用于容器管理的并发。
*   `ConcurrencyManagementType.BEAN`用于 bean 管理的并发。

### 5.1。容器管理的并发性

简单地说，在容器管理的并发中，容器控制客户端如何访问方法。

让我们使用带有值`javax.ejb.ConcurrencyManagementType.CONTAINER`的`@ConcurrencyManagement` 注释:

```java
@Singleton
@Startup
@ConcurrencyManagement(ConcurrencyManagementType.CONTAINER)
public class CountryStateContainerManagedBean implements CountryState {
    ...
}
```

为了指定对单例的每个业务方法的访问级别，我们将使用`javax.ejb.Lock`注释。`javax.ejb.LockType`包含了`@Lock`注释的值。 `javax.ejb.LockType`定义了两个值:

*   **`LockType.WRITE`**–该值为调用客户端提供一个排他锁，防止所有其他客户端访问 bean 的所有方法。对于改变单例 bean 状态的方法，使用此方法。
*   这个值为多个客户端访问一个方法提供了并发锁。
    对于只从 bean 中读取数据的方法，使用此选项。

考虑到这一点，我们将使用`@Lock(LockType.WRITE)`注释定义`setStates()`方法，以防止客户端同时更新状态。

为了允许客户端并发读取数据，我们将用`@Lock(LockType.READ)`来注释`getStates()`:

```java
@Singleton 
@Startup 
@ConcurrencyManagement(ConcurrencyManagementType.CONTAINER) 
public class CountryStateContainerManagedBean implements CountryState { 

    private final Map<String, List<String> countryStatesMap = new HashMap<>();

    @Lock(LockType.READ) 
    public List<String> getStates(String country) { 
        return countryStatesMap.get(country);
    }

    @Lock(LockType.WRITE)
    public void setStates(String country, List<String> states) {
        countryStatesMap.put(country, states);
    }
}
```

为了长时间停止方法执行并无限期阻塞其他客户端，我们将使用`javax.ejb.AccessTimeout`注释来超时长时间等待的调用。

使用`@AccessTimeout`注释定义方法超时的毫秒数。超时后，容器抛出一个`javax.ejb.ConcurrentAccessTimeoutException`，方法执行终止。

### 5.2。Bean 管理的并发性

在 Bean 管理的并发中，容器不控制客户机对单一会话 Bean 的同时访问。开发人员需要自己实现并发性。

除非开发人员实现了并发，否则所有客户端都可以同时访问所有方法。Java 提供了 [`synchronization`](/web/20221126220300/https://www.baeldung.com/java-synchronized) 和 [`volatile`](/web/20221126220300/https://www.baeldung.com/java-volatile) 原语来实现并发。

要了解更多关于并发的信息，请阅读这里的`java.util.concurrent`和这里的。

对于 bean 管理的并发性，让我们为单一会话 Bean 类定义带有`javax.ejb.ConcurrencyManagementType.BEAN`值的`@ConcurrencyManagement`注释:

```java
@Singleton 
@Startup 
@ConcurrencyManagement(ConcurrencyManagementType.BEAN) 
public class CountryStateBeanManagedBean implements CountryState { 
   ... 
}
```

接下来，我们将编写使用`synchronized`关键字改变 bean 状态的`setStates()`方法:

```java
public synchronized void setStates(String country, List<String> states) {
    countryStatesMap.put(country, states);
}
```

关键字`synchronized`使得该方法一次只能被一个线程访问。

`getStates()`方法不会改变 Bean 的状态，因此它不需要使用`synchronized`关键字。

## 6。客户端

现在我们可以编写客户机来访问我们的单例会话 Bean。

我们可以在 JBoss、Glassfish 等应用程序容器服务器上部署会话 Bean。为了简单起见，我们将使用 javax `.ejb.embedded.EJBContainer`类。`EJBContainer`与客户端运行在同一个 JVM 中，提供企业 bean 容器的大部分服务。

首先，我们将创建一个`EJBContainer`的实例。该容器实例将搜索并初始化类路径中存在的所有 EJB 模块:

```java
public class CountryStateCacheBeanTest {

    private EJBContainer ejbContainer = null;

    private Context context = null;

    @Before
    public void init() {
        ejbContainer = EJBContainer.createEJBContainer();
        context = ejbContainer.getContext();
    }
}
```

接下来，我们将从初始化的容器对象中获取`javax.naming.Context`对象。使用`Context` 实例，我们可以获得对`CountryStateContainerManagedBean`的引用并调用方法:

```java
@Test
public void whenCallGetStatesFromContainerManagedBean_ReturnsStatesForCountry() throws Exception {

    String[] expectedStates = {"Texas", "Alabama", "Alaska", "Arizona", "Arkansas"};

    CountryState countryStateBean = (CountryState) context
      .lookup("java:global/singleton-ejb-bean/CountryStateContainerManagedBean");
    List<String> actualStates = countryStateBean.getStates("UnitedStates");

    assertNotNull(actualStates);
    assertArrayEquals(expectedStates, actualStates.toArray());
}

@Test
public void whenCallSetStatesFromContainerManagedBean_SetsStatesForCountry() throws Exception {

    String[] expectedStates = { "California", "Florida", "Hawaii", "Pennsylvania", "Michigan" };

    CountryState countryStateBean = (CountryState) context
      .lookup("java:global/singleton-ejb-bean/CountryStateContainerManagedBean");
    countryStateBean.setStates(
      "UnitedStates", Arrays.asList(expectedStates));

    List<String> actualStates = countryStateBean.getStates("UnitedStates");
    assertNotNull(actualStates);
    assertArrayEquals(expectedStates, actualStates.toArray());
}
```

类似地，我们可以使用`Context`实例来获取 Bean 管理的单例 Bean 的引用，并调用相应的方法:

```java
@Test
public void whenCallGetStatesFromBeanManagedBean_ReturnsStatesForCountry() throws Exception {

    String[] expectedStates = { "Texas", "Alabama", "Alaska", "Arizona", "Arkansas" };

    CountryState countryStateBean = (CountryState) context
      .lookup("java:global/singleton-ejb-bean/CountryStateBeanManagedBean");
    List<String> actualStates = countryStateBean.getStates("UnitedStates");

    assertNotNull(actualStates);
    assertArrayEquals(expectedStates, actualStates.toArray());
}

@Test
public void whenCallSetStatesFromBeanManagedBean_SetsStatesForCountry() throws Exception {

    String[] expectedStates = { "California", "Florida", "Hawaii", "Pennsylvania", "Michigan" };

    CountryState countryStateBean = (CountryState) context
      .lookup("java:global/singleton-ejb-bean/CountryStateBeanManagedBean");
    countryStateBean.setStates("UnitedStates", Arrays.asList(expectedStates));

    List<String> actualStates = countryStateBean.getStates("UnitedStates");
    assertNotNull(actualStates);
    assertArrayEquals(expectedStates, actualStates.toArray());
}
```

通过关闭`close()`方法中的`EJBContainer`来结束我们的测试:

```java
@After
public void close() {
    if (ejbContainer != null) {
        ejbContainer.close();
    }
}
```

## 7 .**。结论**

单例会话 Bean 与任何标准会话 Bean 一样灵活和强大，但允许我们应用单例模式在应用程序的客户端之间共享状态。

使用容器管理的并发可以很容易地实现单例 Bean 的并发管理，其中容器负责多个客户端的并发访问，或者您也可以使用 Bean 管理的并发实现您自己的自定义并发管理。

本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126220300/https://github.com/eugenp/tutorials/tree/master/spring-ejb-modules)