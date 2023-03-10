# 春与冬眠的道

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/persistence-layer-with-spring-and-hibernate>

## 1。概述

本文将展示如何用 Spring 和 Hibernate 实现 DAO。关于核心 Hibernate 配置，请查看之前的 [Hibernate 5 with Spring](/web/20220628154420/https://www.baeldung.com/hibernate-5-spring) 文章。

## 2。不再有 Spring 模板

从 Spring 3.0 和 Hibernate 3.0.1 开始，**Spring`HibernateTemplate`不再需要**来管理休眠会话。现在可以利用由 Hibernate 直接管理的[上下文会话](https://web.archive.org/web/20220628154420/http://docs.jboss.org/hibernate/core/3.6/reference/en-US/html/architecture.html#architecture-current-session "Hibernate - contextual sessions reference")–**会话，这些会话在整个事务范围内都是活动的。**

因此，现在的最佳实践是直接使用 Hibernate API 而不是`HibernateTemplate.`,这将有效地将 DAO 层实现与 Spring 完全解耦。

### 2.1。`HibernateTemplate`异常翻译无

异常翻译是`HibernateTemplate`的职责之一——将低级别的 Hibernate 异常翻译成高级别的通用 Spring 异常。

在没有模板的情况下，**这种机制对于所有用`@Repository`** 注释的 Dao 仍然是启用的并且是活动的 **。在幕后，它使用了一个 Spring bean 后处理器，该处理器将使用 Spring 上下文中找到的所有`PersistenceExceptionTranslator`来通知所有的`@Repository`bean。**

需要记住的一点是，异常转换使用代理。为了让 Spring 能够围绕 DAO 类创建代理，这些不能被声明为`final`。

### 2.2。无模板的 Hibernate 会话管理

当 Hibernate 对上下文会话的支持出现时，`HibernateTemplate`基本上就过时了。事实上，该类的 Javadoc 现在突出了这一方面(从原来的粗体):

> 注意:从 Hibernate 3.0.1 开始，事务性 Hibernate 访问代码也可以用普通 Hibernate 风格编码。因此，对于新开始的项目，考虑采用标准的 Hibernate3 风格编码数据访问对象，基于{ @ link org . hibernate . session factory # getCurrentSession()}。

## 3。刀

我们将从基础 DAO**开始——一个抽象的、参数化的 DAO** ,它支持常见的通用操作，并且我们可以为每个实体进行扩展:

```java
public abstract class AbstractHibernateDao<T extends Serializable> {
    private Class<T> clazz;

    @Autowired
    protected SessionFactory sessionFactory;

    public final void setClazz(final Class<T> clazzToSet) {
        clazz = Preconditions.checkNotNull(clazzToSet);
    }

    // API
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

这里有几个方面很有趣——如前所述，抽象 DAO 没有扩展任何 Spring 模板(比如`HibernateTemplate`)。相反，Hibernate `SessionFactory` 被直接注入到 DAO 中，并且将具有主 Hibernate API 的角色，通过上下文`Session`它公开:

`this.sessionFactory.getCurrentSession();`

还要注意，构造函数接收实体的`Class`作为在泛型操作中使用的参数。

现在，让我们看一下**这个 DAO** 对于`Foo`实体的示例实现:

```java
@Repository
public class FooDAO extends AbstractHibernateDAO< Foo > implements IFooDAO{

   public FooDAO(){
      setClazz(Foo.class );
   }
}
```

## 4。结论

本文介绍了用 Hibernate 和 Spring 配置和实现持久层。

讨论了停止依赖 DAO 层模板的原因，以及配置 Spring 来管理事务和 Hibernate 会话可能存在的缺陷。最终的结果是一个轻量级的、干净的 DAO 实现，几乎没有对 Spring 的编译时依赖。

这个简单项目的实现可以在[github 项目](https://web.archive.org/web/20220628154420/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jpa-2 "DAO with Hibernate - example project")中找到。