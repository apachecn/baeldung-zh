# 带 JPA 和弹簧的刀

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-dao-jpa>

## 1。概述

本文将展示如何用 Spring 和 JPA 实现 DAO。关于核心 JPA 配置，参见[关于 JPA](/web/20220625231030/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa "Spring 3 and JPA with Hibernate") 与 Spring 的文章。

## 2。不再有 Spring 模板

从 Spring 3.1 开始，`JpaTemplate`和相应的`JpaDaoSupport` **已经被**弃用，取而代之的是使用原生 Java 持久性 API。

另外，这两个类都只与 JPA 1 相关(来自`JpaTemplate` javadoc):

> 注意，这个类没有升级到 JPA 2.0，而且永远不会升级。

因此，现在的最佳实践是**直接使用 Java 持久性 API**而不是`JpaTemplate`。

### 2.1。没有模板的异常翻译

`JpaTemplate`的职责之一是**异常翻译**——将低级别的异常翻译成高级别的通用 Spring 异常。

没有模板，**异常翻译仍然启用，并且对于所有用`@Repository`标注的 Dao，**功能齐全。Spring 用一个 bean 后处理器实现了这一点，它会用容器中找到的所有`PersistenceExceptionTranslator`通知所有的`@Repository`bean。

同样重要的是要注意到**异常转换机制使用代理**——为了让 Spring 能够围绕 DAO 类创建代理，这些不能被声明`**final**`。

## 3。 **刀**

首先，我们将实现所有 Dao 的基础层——一个使用泛型的抽象类，旨在进行扩展:

```java
public abstract class AbstractJpaDAO< T extends Serializable > {

   private Class< T > clazz;

   @PersistenceContext
   EntityManager entityManager;

   public final void setClazz( Class< T > clazzToSet ){
      this.clazz = clazzToSet;
   }

   public T findOne( long id ){
      return entityManager.find( clazz, id );
   }
   public List< T > findAll(){
      return entityManager.createQuery( "from " + clazz.getName() )
       .getResultList();
   }

   public void create( T entity ){
      entityManager.persist( entity );
   }

   public T update( T entity ){
      return entityManager.merge( entity );
   }

   public void delete( T entity ){
      entityManager.remove( entity );
   }
   public void deleteById( long entityId ){
      T entity = findOne( entityId );
      delete( entity );
   }
}
```

这里最有趣的一点是`EntityManager`被注入的方式——使用标准的`@PersistenceContext`注释。在幕后，这是由`PersistenceAnnotationBeanPostProcessor`处理的——它处理注释，从容器中检索 JPA 实体管理器并注入它。

持久性后处理器要么通过在配置中定义它来显式创建，要么通过在名称空间配置中定义`context:annotation-config`或`context:component-scan`来自动创建。

另外，请注意实体`**Class**`是在构造函数中传递的，将在泛型操作中使用:

```java
@Repository
public class FooDAO extends AbstractJPADAO< Foo > implements IFooDAO{

   public FooDAO(){
      setClazz(Foo.class );
   }
}
```

## 4。结论

本教程展示了**如何使用基于 XML 和 Java 的配置，用 Spring 和 JPA** 建立一个 DAO 层。我们还讨论了为什么不使用`JpaTemplate`以及如何用`EntityManager`替换它。最终的结果是一个轻量级的、干净的 DAO 实现，几乎没有对 Spring 的编译时依赖。

这个简单项目的实现可以在[GitHub 项目](https://web.archive.org/web/20220625231030/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jpa "DAO with JPA - example project")中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。