# Arquillian 测试简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/arquillian>

## 1。概述

Arquillian 是一个用于 Jakarta EE 的独立于容器的集成测试框架。使用 Arquillian 可以最小化管理容器、部署、框架初始化等的负担。

我们可以专注于编写实际的测试，而不是引导测试环境。

## 2。核心概念

### 2.1。部署档案

在容器中运行时，有一种简单的方法来测试我们的应用程序。

首先，`ShrinkWrap` 类提供了一个 API 来创建可部署的`*.jar,` `*.war,`和`*.ear`文件。

然后，Arquillian 允许我们在返回`ShrinkWrap`对象的方法上使用`@Deployment`注释来配置测试部署。

### 2.2。集装箱

Arquillian 区分三种不同类型的容器:

*   远程–使用远程协议(如 JMX)进行测试
*   托管——远程容器，但它们的生命周期由 Arquillian 管理
*   嵌入式–使用本地协议执行测试的本地容器

此外，我们可以根据容器的功能对其进行分类:

*   Jakarta EE 应用程序部署在 Glassfish 或 JBoss 等应用服务器上
*   部署在 Tomcat 或 Jetty 上的 Servlet 容器
*   独立容器
*   OSGI 容器

它检查运行时类路径，并自动选择可用的容器。

### 2.3。测试浓缩

Arquillian 通过提供依赖注入来丰富测试，这样我们就可以轻松地编写测试。

我们可以使用`@Inject`注入依赖关系，使用`@Resource`注入资源，使用`@EJB,` 注入 EJB 会话 beans 等等。

### 2.4。多名试车员

我们可以使用注释创建多个部署:

```java
@Deployment(name="myname" order = 1)
```

其中 name 是部署文件的名称，order 参数是部署的执行顺序，因此我们现在可以使用注释同时在多个部署上运行测试:

```java
@Test @OperateOnDeployment("myname")
```

使用在`@Deployment`注释中定义的顺序在`myname` 部署容器上执行 before 测试。

### 2.5。阿奎利亚语扩展

Arquillian 提供了多种扩展，以防核心运行时无法满足我们的测试需求。我们有持久性、事务、客户机/服务器、REST 扩展等等。

我们可以通过向 Maven 或 Gradle 配置文件添加适当的依赖项来启用这些扩展。

常用的扩展有无人机、石墨烯、硒。

## 3。Maven 依赖性和设置

让我们将下面的依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.jboss.arquillian</groupId>
    <artifactId>arquillian-bom</artifactId>
    <version>1.1.13.Final</version>
    <scope>import</scope>
    <type>pom</type>
</dependency>
<dependency>
    <groupId>org.glassfish.main.extras</groupId>
    <artifactId>glassfish-embedded-all</artifactId>
    <version>4.1.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.jboss.arquillian.container</groupId>
    <artifactId>arquillian-glassfish-embedded-3.1</artifactId>
    <version>1.0.0.Final</version>
    <scope>test</scope>
</dependency>
```

最新版本的依赖项可以在这里找到: [arquillian-bom](https://web.archive.org/web/20221128115507/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.jboss.arquillian%22%20AND%20a%3A%22arquillian-bom%22) 、[org . glassfish . main . extras](https://web.archive.org/web/20221128115507/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.glassfish.main.extras%22%20AND%20a%3A%22glassfish-embedded-all%22)、[org . JBoss . arquillian . container](https://web.archive.org/web/20221128115507/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.jboss.arquillian.container%22%20AND%20a%3A%22arquillian-glassfish-embedded-3.1%22)。

## 4。简单测试

### 4.1。创建一个组件

让我们从一个简单的组件开始。我们在这里不包括任何高级逻辑，以便能够专注于测试:

```java
public class Component {
    public void sendMessage(PrintStream to, String msg) {
        to.println(message(msg));
    }

    public String message(String msg) {
        return "Message, " + msg;
    }
}
```

使用 Arquillian，我们希望测试这个类在作为 CDI bean 调用时的行为是否正确。

### 4.2。编写我们的第一个阿奎利亚测试

首先，我们需要指定我们的测试类应该使用特定于框架的运行程序来运行:

```java
@RunWith(Arquillian.class) 
```

如果我们要在容器中运行我们的测试，我们需要使用`@Deployment`注释。

Arquillian 不使用整个类路径来隔离测试档案。相反，它使用了`ShrinkWrap` 类，这是一个用于创建归档的 Java API。当我们创建要测试的归档文件时，我们指定要在类路径中包含哪些文件来进行测试。在部署过程中， `ShrinkWrap` 只隔离测试所需的类。

使用`addclass()`方法，我们可以指定所有必要的类，还可以添加一个空的清单资源。

`JavaArchive.class` 创建一个名为`test.war,` 的 web 档案模型。这个文件被部署到容器中，然后被 Arquillian 用来执行测试:

```java
@Deployment
public static JavaArchive createDeployment() {
    return ShrinkWrap.create(JavaArchive.class)
      .addClass(Component.class)
      .addAsManifestResource(EmptyAsset.INSTANCE, "beans.xml");
}
```

然后我们在测试中注入我们的组件:

```java
@Inject
private Component component;
```

最后，我们执行测试:

```java
assertEquals("Message, MESSAGE",component.message(("MESSAGE")));

component.sendMessage(System.out, "MESSAGE");
```

## 5。测试企业 Java bean

### 5.1。企业 Java Bean

使用 Arquillian，我们可以测试企业 Java Bean 的依赖注入，为此，我们创建了一个类，该类具有将任何单词转换为小写的方法:

```java
public class ConvertToLowerCase {
    public String convert(String word){
        return word.toLowerCase();
    }
}
```

使用这个类，我们创建一个无状态类来调用之前创建的方法:

```java
@Stateless
public class CapsConvertor {
    public ConvertToLowerCase getLowerCase(){
        return new ConvertToLowerCase();
    }
}
```

`CapsConvertor`类被注入到服务 bean 中:

```java
@Stateless
public class CapsService {

    @Inject
    private CapsConvertor capsConvertor;

    public String getConvertedCaps(final String word){
        return capsConvertor.getLowerCase().convert(word);
    }
}
```

### 5.2。测试企业 Java Bean

现在我们可以使用 Arquillian 来测试我们的企业 Java Bean，注入`CapsService`:

```java
@Inject
private CapsService capsService;

@Test
public void givenWord_WhenUppercase_ThenLowercase(){
    assertTrue("capitalize".equals(capsService.getConvertedCaps("CAPITALIZE")));
    assertEquals("capitalize", capsService.getConvertedCaps("CAPITALIZE"));
}
```

使用`ShrinkWrap,` ,我们确保所有类都正确连接:

```java
@Deployment
public static JavaArchive createDeployment() {
    return ShrinkWrap.create(JavaArchive.class)
      .addClasses(CapsService.class, CapsConvertor.class, ConvertToLowerCase.class)
      .addAsManifestResource(EmptyAsset.INSTANCE, "beans.xml");
}
```

## 6。测试 JPA

### 6.1。持久性

我们还可以使用 Arquillian 来测试持久性。首先，我们要创建我们的实体:

```java
@Entity
public class Car {

    @Id
    @GeneratedValue
    private Long id;

    @NotNull
    private String name;

    // getters and setters
}
```

我们有一个保存汽车名称的表。

然后，我们将创建 EJB 来对我们的数据执行基本操作:

```java
@Stateless
public class CarEJB {

    @PersistenceContext(unitName = "defaultPersistenceUnit")
    private EntityManager em;

    public Car saveCar(Car car) {
        em.persist(car);
        return car;
    }

    public List<Car> findAllCars() {
    Query query 
      = em.createQuery("SELECT b FROM Car b ORDER BY b.name ASC");
    List<Car> entries = query.getResultList();

    return entries == null ? new ArrayList<>() : entries;    

    public void deleteCar(Car car) {
        car = em.merge(car);
        em.remove(car);
    }
}
```

使用`saveCar`我们可以将汽车名称保存到数据库中，使用`findAllCars,` 我们可以获得所有存储的汽车，还可以使用`deleteCar`从数据库中删除一辆汽车。

### 6.2。用 Arquillian 测试持久性

现在我们可以使用 Arquillian 执行一些基本的测试。

首先，我们将类添加到我们的`ShrinkWrap:`

```java
.addClasses(Car.class, CarEJB.class)
.addAsResource("META-INF/persistence.xml")
```

然后我们创建我们的测试:

```java
@Test
public void testCars() {
    assertTrue(carEJB.findAllCars().isEmpty());
    Car c1 = new Car();
    c1.setName("Impala");
    Car c2 = new Car();
    c2.setName("Lincoln");
    carEJB.saveCar(c1);
    carEJB.saveCar(c2);

    assertEquals(2, carEJB.findAllCars().size());

    carEJB.deleteCar(c1);

    assertEquals(1, carEJB.findAllCars().size());
}
```

在这个测试中，我们首先创建四个 car 实例，并检查数据库中的行数是否与我们创建的相同。

## 8。结论

在本教程中，我们将:

*   介绍阿奎利亚人的核心概念
*   在阿奎利亚测试中注入了一种成分
*   测试一辆 EJB
*   经过测试的持久性
*   使用 Maven 执行了 Arquillian 测试

你可以在 Github 的文章[中找到代码。](https://web.archive.org/web/20221128115507/https://github.com/eugenp/tutorials/tree/master/web-modules/jee-7)