# 避免服务层的脆弱测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/testing-the-java-service-layer>

## 目录

*   [**1。**概述](#overview)
*   [**2。**层层](#templates)
*   [**3。**动机和模糊单元测试的界限](#javaconfig)
*   [**4。**结论](#conclusion)

## 1。概述

有很多方法可以测试一个应用的服务层。本文的目标是通过完全模拟与数据库的交互，展示一种孤立地对该层进行单元测试的方法。

这个例子将使用 [Spring](https://web.archive.org/web/20220120011609/https://spring.io/ "Spring") 进行依赖注入，使用 JUnit、 [Hamcrest](https://web.archive.org/web/20220120011609/https://code.google.com/archive/p/hamcrest/ "Hamcrest") 和 [Mockito](https://web.archive.org/web/20220120011609/https://code.google.com/p/mockito/ "Mockito") 进行测试，但是技术可能会有所不同。

## 2。层层

典型的 java web 应用程序在 DAL/DAO 层之上有一个服务层，这个服务层又会调用原始的持久层。

 `### 1.1。服务层

```java
@Service
public class FooService implements IFooService{

   @Autowired
   IFooDAO dao;

   @Override
   public Long create( Foo entity ){
      return this.dao.create( entity );
   }

}
```

### 1.2。DAL/DAO 层

```java
@Repository
public class FooDAO extends HibernateDaoSupport implements IFooDAO{

   public Long create( Foo entity ){
      Preconditions.checkNotNull( entity );

      return (Long) this.getHibernateTemplate().save( entity );
   }

}
```

## 3。动机和模糊单元测试的界限

当单元测试一个服务时，标准的**单元**通常是服务**类**，就这么简单。测试将模拟底层——在本例中是 DAO/DAL 层，并验证其上的交互。DAO 层也是如此——模拟与数据库的交互(在本例中为`HibernateTemplate`),并验证与数据库的交互。

这是一种有效的方法，但是它会导致脆弱的测试——添加或删除一个层几乎总是意味着完全重写测试。发生这种情况是因为测试依赖于层的确切结构，而这种结构的改变意味着测试的改变。

为了避免这种不灵活性，我们可以通过改变单元的定义来扩大单元测试的范围——我们可以将持久化操作视为一个单元，从服务层到 DAO，一直到原始的持久化——无论它是什么。现在，单元测试将使用服务层的 API，并将模拟出原始的持久性——在本例中，是`HibernateTemplate`:

```java
public class FooServiceUnitTest{

   FooService instance;

   private HibernateTemplate hibernateTemplateMock;

   @Before
   public void before(){
      this.instance = new FooService();
      this.instance.dao = new FooDAO();
      this.hibernateTemplateMock = mock( HibernateTemplate.class );
      this.instance.dao.setHibernateTemplate( this.hibernateTemplateMock );
   }

   @Test
   public void whenCreateIsTriggered_thenNoException(){
      // When
      this.instance.create( new Foo( "testName" ) );
   }

   @Test( expected = NullPointerException.class )
   public void whenCreateIsTriggeredForNullEntity_thenException(){
      // When
      this.instance.create( null );
   }

   @Test
   public void whenCreateIsTriggered_thenEntityIsCreated(){
      // When
      Foo entity = new Foo( "testName" );
      this.instance.create( entity );

      // Then
      ArgumentCaptor< Foo > argument = ArgumentCaptor.forClass( Foo.class );
      verify( this.hibernateTemplateMock ).save( argument.capture() );
      assertThat( entity, is( argument.getValue() ) );
   }

}
```

现在，测试只关注一个单一的责任——当创建被触发时，创建是否到达数据库？

最后一个测试使用 Mockito 验证语法来检查 hibernate 模板上的`save`方法是否被调用，并捕获流程中的参数，以便也可以对其进行检查。创建实体的**责任**通过这个交互测试来验证，不需要检查任何状态——测试相信 hibernate 保存逻辑正在按预期工作。当然，这也需要测试，但那是另一种责任和另一种类型的测试。

## 4。结论

这种技术总是导致更集中的测试，这使得它们对变化更有弹性和灵活性。测试失败的唯一原因是测试中的**职责**被破坏。`