# Java EE 会话 Beans

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ejb-session-beans>

## 1。简介

企业会话 Beans 可以大致分为:

1.  无状态会话 Beans
2.  有状态会话 Beans

在这篇简短的文章中，我们将讨论这两种主要类型的会话 beans。

## 2。设置

要使用企业 bean 3.2**，**请确保将最新版本添加到`pom.xml` 文件的`dependencies`部分:

```java
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>7.0</version>
    <scope>provided</scope>
</dependency>
```

The latest dependency can be found in the [Maven Repository](https://web.archive.org/web/20220626204520/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22javax%22%20AND%20a%3A%22javaee-api%22). This dependency ensures that all Java EE 7 APIs are available during compile time. The `provided` scope ensures that once deployed; the dependency will be provided by the container where it has been deployed.

## 3。无状态 bean

无状态会话 bean 是一种企业 bean，通常用于独立操作。它没有任何关联的客户端状态，但可以保留其实例状态。

让我们看一个例子来演示无状态 bean 是如何工作的。

### 3.1。创建无状态 Bean

首先，让我们创建`StatelessEJB` bean。我们使用`@Stateless`注释将 bean 标记为无状态:

```java
@Stateless
public class StatelessEJB {

    public String name;

}
```

然后我们创建上述无状态 bean 的第一个客户机，名为`EJBClient1`:

```java
public class EJBClient1 {

    @EJB
    public StatelessEJB statelessEJB;

}
```

然后，我们声明另一个名为`EJBClient2,`的客户机，它访问同一个无状态 bean:

```java
public class EJBClient2 {

    @EJB
    public StatelessEJB statelessEJB;

}
```

### 3.2。测试无状态 Bean

为了测试 EJB 的无状态性，我们可以通过以下方式使用我们在上面声明的两个客户端:

```java
@RunWith(Arquillian.class)
public class StatelessEJBTest {

    @Inject
    private EJBClient1 ejbClient1;

    @Inject
    private EJBClient2 ejbClient2;

    @Test
    public void givenOneStatelessBean_whenStateIsSetInOneBean
      _secondBeanShouldHaveSameState() {

        ejbClient1.statelessEJB.name = "Client 1";
        assertEquals("Client 1", ejbClient1.statelessEJB.name);
        assertEquals("Client 1", ejbClient2.statelessEJB.name);
    }

    @Test
    public void givenOneStatelessBean_whenStateIsSetInBothBeans
      _secondBeanShouldHaveSecondBeanState() {

        ejbClient1.statelessEJB.name = "Client 1";
        ejbClient2.statelessEJB.name = "Client 2";
        assertEquals("Client 2", ejbClient2.statelessEJB.name);
    }

    // Arquillian setup code removed for brevity

}
```

我们首先将两个 EBJ 客户注入到单元测试中。

然后，在第一个测试方法中，我们将注入到`EJBClient1`中的 EJB 中的`name`变量设置为值`Client 1\.` 。现在，当我们比较两个客户端中的`name`变量的值时，我们应该看到该值相等。**这表明状态没有保存在无状态 beans 中**。

让我们用不同的方式来证明这一点。在第二个测试方法中，我们看到，一旦我们在第二个客户端中设置了`name`变量，它就会“覆盖”通过`ejbClient1`赋予它的任何值。

## 4。有状态 bean

有状态会话 beans 维护事务内部和事务之间的状态。这就是为什么每个有状态会话 bean 都与特定的客户端相关联。在管理有状态会话 bean 的实例池时，容器可以自动保存和检索 bean 的状态。

### 4.1。创建有状态 Bean

有状态会话 bean 标有`@Stateful`注释。有状态 bean 的代码如下:

```java
@Stateful
public class StatefulEJB {

    public String name;

}
```

我们的有状态 bean 的第一个本地客户机编写如下:

```java
public class EJBClient1 {

    @EJB
    public StatefulEJB statefulEJB;

}
```

名为`EJBClient2`的第二个客户端也像`EJBClient1`一样被创建:

```java
public class EJBClient2 {

    @EJB
    public StatefulEJB statefulEJB;

}
```

### 4.2。测试有状态 Bean

有状态 bean 的功能在`EJBStatefulBeanTest`单元测试中以如下方式进行测试:

```java
@RunWith(Arquillian.class)
public class StatefulEJBTest {

    @Inject
    private EJBClient1 ejbClient1;

    @Inject
    private EJBClient2 ejbClient2;

    @Test
    public void givenOneStatefulBean_whenTwoClientsSetValueOnBean
      _thenClientStateIsMaintained() {

        ejbClient1.statefulEJB.name = "Client 1";
        ejbClient2.statefulEJB.name = "Client 2";
        assertNotEquals(ejbClient1.statefulEJB.name, ejbClient2.statefulEJB.name);
        assertEquals("Client 1", ejbClient1.statefulEJB.name);
        assertEquals("Client 2", ejbClient2.statefulEJB.name);
    }

    // Arquillian setup code removed for brevity

}
```

像以前一样，两个 EJB 客户被注入到单元测试中。在测试方法中，我们可以看到`name`变量的值是通过 *ejbClient1* 客户端设置的，即使通过`ejbClient2`设置的`name`的值不同，该值也会得到维护。**这表明 EJB 的状态被保持**。

## 5。无状态与有状态会话 Bean

现在让我们来看看这两种会话 beans 之间的主要区别。

### 5.1。无状态 bean

*   无状态会话 beans 不维护客户端的任何状态。因此，它们可以用于创建与多个客户端交互的对象池
*   因为无状态 beans 没有每个客户端的任何状态，所以它们具有更好的性能
*   它们可以并行处理来自多个客户端的多个请求
*   可用于从数据库中检索对象

### 5.2。有状态 bean

*   有状态会话 beans 可以维护多个客户端的状态，并且任务不在客户端之间共享
*   该状态在会话期间持续。会话销毁后，状态不会保留
*   容器可以将状态序列化并存储为陈旧状态以供将来使用。这样做是为了节省应用服务器资源和支持 bean 故障，并且是钝化的
*   可以用来解决生产者-消费者类型的问题

## 6。结论

因此，我们创建了两种类型的会话 bean 和相应的客户端来调用 bean 中的方法。该项目演示了两种主要类型的会话 beans 的行为。

和往常一样，本文的[源代码](https://web.archive.org/web/20220626204520/https://github.com/eugenp/tutorials/tree/master/spring-ejb-modules)可以在 GitHub 上找到。