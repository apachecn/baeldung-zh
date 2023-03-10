# 用 Spring 和 Java 泛型简化 DAO

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/simplifying-the-data-access-layer-with-spring-and-java-generics>

## 1。概述

本文将关注于通过对系统中的所有实体使用单一的、通用化的数据访问对象来简化 DAO 层，这将导致优雅的数据访问，没有不必要的混乱或冗长。

我们将基于我们在上一篇关于 Spring 和 Hibernate 的文章中看到的抽象 DAO 类，并添加泛型支持。

## 延伸阅读:

## [Spring Data JPA 简介](/web/20221129002243/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)

Introduction to Spring Data JPA with Spring 4 - the Spring config, the DAO, manual and generated queries and transaction management.[Read more](/web/20221129002243/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) →

## [带 Spring 的 JPA 指南](/web/20221129002243/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)

Setup JPA with Spring - how to set up the EntityManager factory and use the raw JPA APIs.[Read more](/web/20221129002243/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) →

## [使用 JPA、Hibernate 和 Spring 数据 JPA 进行审计](/web/20221129002243/https://www.baeldung.com/database-auditing-jpa)

This article demonstrates three approaches to introducing auditing into an application: JPA, Hibernate Envers, and Spring Data JPA.[Read more](/web/20221129002243/https://www.baeldung.com/database-auditing-jpa) →

****

## 2。Hibernate 和 JPA DAOs

大多数产品代码库都有某种类型的 DAO 层。通常，实现的范围从没有抽象基类的多个类到某种泛型类。然而，有一点是一致的——总有不止一个。最有可能的是，Dao 和系统中的实体之间存在一对一的关系。

此外，根据所涉及的泛型的级别，实际的实现可能从大量重复的代码到几乎为空的代码都有所不同，大部分逻辑都分组在一个基本抽象类中。

这些多重实现通常可以被一个参数化的 DAO 所取代。我们可以通过充分利用 Java 泛型提供的类型安全来实现这一点，这样就不会丢失任何功能。

接下来我们将展示这个概念的两个实现，一个用于 Hibernate 中心持久层，另一个侧重于 JPA。这些实现并不完整，但是我们可以很容易地添加更多的数据访问方法。

### 2.1.抽象 Hibernate DAO

让我们快速看一下`AbstractHibernateDao`类:

```java
public abstract class AbstractHibernateDao<T extends Serializable> {
    private Class<T> clazz;

    @Autowired
    protected SessionFactory sessionFactory;

    public void setClazz(final Class<T> clazzToSet) {
        clazz = Preconditions.checkNotNull(clazzToSet);
    }

    public T findOne(final long id) {
        return (T) getCurrentSession().get(clazz, id);
    }

    public List<T> findAll() {
        return getCurrentSession().createQuery("from " + clazz.getName()).list();
    }

    public T create(final T entity) {
        Preconditions.checkNotNull(entity);
        getCurrentSession().saveOrUpdate(entity);
        return entity;
    }

    public T update(final T entity) {
        Preconditions.checkNotNull(entity);
        return (T) getCurrentSession().merge(entity);
    }

    public void delete(final T entity) {
        Preconditions.checkNotNull(entity);
        getCurrentSession().delete(entity);
    }

    public void deleteById(final long entityId) {
        final T entity = findOne(entityId);
        Preconditions.checkState(entity != null);
        delete(entity);
    }

    protected Session getCurrentSession() {
        return sessionFactory.getCurrentSession();
    }
}
```

这是一个具有几个数据访问方法的抽象类，使用`SessionFactory`来操作实体。

我们在这里使用 Google Guava 的前提条件来确保方法或构造函数是用有效的参数值调用的。如果前提条件失败，就会抛出定制的异常。

### 2.2。通用 Hibernate DAO

现在我们有了抽象的 DAO 类，我们可以只扩展它一次。**通用 DAO 实现将成为我们需要的唯一实现**:

```java
@Repository
@Scope(BeanDefinition.SCOPE_PROTOTYPE)
public class GenericHibernateDao<T extends Serializable>
  extends AbstractHibernateDao<T> implements IGenericDao<T>{
   //
}
```

首先，**注意，通用实现本身是参数化的**，允许客户端根据具体情况选择正确的参数。这意味着客户端可以获得类型安全的所有好处，而不需要为每个实体创建多个工件。

其次，注意**这个泛型 DAO 实现**的原型范围。使用这个作用域意味着 Spring 容器将在每次被请求时创建一个新的 DAO 实例(包括自动连接)。这将允许服务根据需要为不同的实体使用多个具有不同参数的 Dao。

这个范围如此重要的原因在于 Spring 在容器中初始化 beans 的方式。让泛型 DAO 没有作用域意味着使用默认的单例作用域**，这将导致 DAO 的单个实例存在于容器**中。对于任何更复杂的场景来说，这显然是一个很大的限制。

`IGenericDao`只是所有 DAO 方法的一个接口，因此我们可以注入我们需要的实现:

```java
public interface IGenericDao<T extends Serializable> {
    void setClazz(Class< T > clazzToSet);

    T findOne(final long id);

    List<T> findAll();

    T create(final T entity);

    T update(final T entity);

    void delete(final T entity);

    void deleteById(final long entityId);
} 
```

### 2.3。抽象 JPA 刀

`AbstractJpaDao`与`AbstractHibernateDao:`非常相似

```java
public abstract class AbstractJpaDAO<T extends Serializable> {
    private Class<T> clazz;

    @PersistenceContext(unitName = "entityManagerFactory")
    private EntityManager entityManager;

    public final void setClazz(final Class<T> clazzToSet) {
        this.clazz = clazzToSet;
    }

    public T findOne(final long id) {
        return entityManager.find(clazz, id);
    }

    @SuppressWarnings("unchecked")
    public List<T> findAll() {
        return entityManager.createQuery("from " + clazz.getName()).getResultList();
    }

    public T create(final T entity) {
        entityManager.persist(entity);
        return entity;
    }

    public T update(final T entity) {
        return entityManager.merge(entity);
    }

    public void delete(final T entity) {
        entityManager.remove(entity);
    }

    public void deleteById(final long entityId) {
        final T entity = findOne(entityId);
        delete(entity);
    }
}
```

类似于 Hibernate DAO 实现，我们直接使用 Java 持久性 API，而不依赖于现在已经过时的 Spring `JpaTemplate`。

### 2.4。通用 JPA 刀

与 Hibernate 实现类似，JPA 数据访问对象也很简单:

```java
@Repository
@Scope( BeanDefinition.SCOPE_PROTOTYPE )
public class GenericJpaDao< T extends Serializable >
 extends AbstractJpaDao< T > implements IGenericDao< T >{
   //
}
```

## 3。注入此刀

**我们现在有了一个可以注入的单一 DAO 接口。**我们还需要指定`Class:`

```java
@Service
class FooService implements IFooService{

   IGenericDao<Foo> dao;

   @Autowired
   public void setDao(IGenericDao<Foo> daoToSet) {
      dao = daoToSet;
      dao.setClazz(Foo.class);
   }

   // ...
}
```

Spring **使用 setter 注入**自动连接新的 DAO 实例，以便可以使用`Class`对象定制实现。在这之后，DAO 被完全参数化，并准备好供服务使用。

当然，还有其他方法可以为 DAO 指定类——通过反射，或者甚至用 XML。我倾向于这种更简单的解决方案，因为与使用反射相比，可读性和透明性都有所提高。

## 4。结论

本文讨论了数据访问层的**简化，它提供了一个通用 DAO 的单一的、可重用的实现。我们展示了在基于 Hibernate 和 JPA 的环境中的实现。结果是一个精简的持久层，没有不必要的混乱。**

关于使用基于 Java 的配置和项目的基本 Maven pom 建立 Spring 上下文的逐步介绍，请参见本文。

最后，本文的代码可以在 GitHub 项目中找到。