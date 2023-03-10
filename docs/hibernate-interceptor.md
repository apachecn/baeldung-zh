# 冬眠拦截器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-interceptor>

## 1。概述

在这个讨论中，我们将看看在 [Hibernate 的](https://web.archive.org/web/20221107223704/http://hibernate.org/)抽象关系映射实现中截取操作的各种方式。

## 2。定义 Hibernate 拦截器

Hibernate 拦截器是一个接口，它允许我们对 Hibernate 中的某些事件做出反应。

这些拦截器被注册为回调，并在 Hibernate 的会话和应用程序之间提供通信链接。通过这样的回调，应用程序可以拦截核心 Hibernate 的操作，比如`save, update, delete, etc.`

有两种定义拦截器的方法:

1.  **实现`org.hibernate.Interceptor`** 接口
2.  **扩展`org.hibernate.EmptyInterceptor`** 类

### 2.1。实现一个`Interceptor`接口

**实现`org.hibernate.Interceptor`需要实现大约 14 个伴随方法**。这些方法包括`onLoad, onSave, onDelete, findDirty,`等等。

确保任何实现拦截器接口的类都是可序列化的也很重要(`implements java.io.Serializable`)。

一个典型的例子是这样的:

```java
public class CustomInterceptorImpl implements Interceptor, Serializable {

    @Override
    public boolean onLoad(Object entity, Serializable id, 
      Object[] state, String[] propertyNames, Type[] types) 
      throws CallbackException {
        // ...
        return false;
    }

    // ...

    @Override
    public String onPrepareStatement(String sql) {
        // ...   
        return sql;
    }

}
```

如果没有特殊的要求，强烈推荐使用**扩展`EmptyInterceptor`** 类，并且只重写所需的方法。

### 2.2。`EmptyInterceptor`延伸

扩展`org.hibernate.EmptyInterceptor`类提供了一种更简单的定义拦截器的方法。**我们现在只需要覆盖与我们想要拦截的操作相关的方法。**

例如，我们可以将`CustomInterceptor`定义为:

```java
public class CustomInterceptor extends EmptyInterceptor {
}
```

如果我们需要在数据保存操作执行之前拦截它们，我们需要覆盖`onSave`方法:

```java
@Override
public boolean onSave(Object entity, Serializable id, 
  Object[] state, String[] propertyNames, Type[] types) {

    if (entity instanceof User) {
        logger.info(((User) entity).toString());
    }
    return super.onSave(entity, id, state, propertyNames, types);
}
```

注意这个实现是如何简单地打印出实体的——如果它是一个`User`。

虽然有可能返回值`true`或`false`，但是允许通过调用`super.onSave()`来传播`onSave`事件是一个很好的做法。

**另一个用例是为数据库交互提供审计跟踪。**我们可以使用`onFlushDirty()`方法来知道一个实体何时发生变化。

对于`User`对象，我们可以决定在`User`类型的实体发生变化时更新它的`lastModified` date 属性。

这可以通过以下方式实现:

```java
@Override
public boolean onFlushDirty(Object entity, Serializable id, 
  Object[] currentState, Object [] previousState, 
  String[] propertyNames, Type[] types) {

    if (entity instanceof User) {
        ((User) entity).setLastModified(new Date());
        logger.info(((User) entity).toString());
    }
    return super.onFlushDirty(entity, id, currentState, 
      previousState, propertyNames, types);
}
```

其他事件如`delete`和`load`(对象初始化)可以通过分别实现相应的`onDelete`和 `onLoad`方法来拦截。

## 3。正在登记`Interceptors`

Hibernate 拦截器既可以注册为`Session`作用域，也可以注册为`SessionFactory-scoped`。

### 3.1。`Session-scoped Interceptor`

一个`Session`范围的拦截器链接到一个特定的会话。它是在会话定义或打开时创建的，如下所示:

```java
public static Session getSessionWithInterceptor(Interceptor interceptor) 
  throws IOException {
    return getSessionFactory().withOptions()
      .interceptor(interceptor).openSession();
}
```

在上文中，我们用特定的 hibernate 会话显式注册了一个拦截器。

### 3.2。`SessionFactory`-作用域`Interceptor`

在构建`SessionFactory.`之前注册一个`SessionFactory-`范围的拦截器，这通常是通过`SessionFactoryBuilder`实例上的`applyInterceptor`方法完成的:

```java
ServiceRegistry serviceRegistry = configureServiceRegistry();
SessionFactory sessionFactory = getSessionFactoryBuilder(serviceRegistry)
  .applyInterceptor(new CustomInterceptor())
  .build();
```

值得注意的是，`SessionFactory-`范围的拦截器将应用于所有会话。因此，我们需要小心不要存储特定于会话的状态——因为这个拦截器将被不同的会话同时使用。

对于特定于会话的行为，建议使用不同的拦截器显式打开一个会话，如前面所示。

对于`SessionFactory`范围的拦截器，我们自然需要确保它是线程安全的。这可以通过在属性文件中指定会话上下文来实现:

```java
hibernate.current_session_context_class=org.hibernate.context.internal.ThreadLocalSessionContext
```

或者将它添加到我们的 XML 配置文件中:

```java
<property name="hibernate.current_session_context_class">
    org.hibernate.context.internal.ThreadLocalSessionContext
</property>
```

此外，为了确保可串行化，`SessionFactory`范围的拦截器必须实现`Serializable`接口的`readResolve`方法。

## 4。结论

我们已经看到了如何将 Hibernate 拦截器定义和注册为`Session`作用域或`SessionFactory`作用域。在这两种情况下，我们必须确保拦截器是可序列化的，尤其是如果我们想要一个可序列化的会话。

拦截器的其他替代方案包括 Hibernate 事件和 JPA 回调。

和往常一样，你可以在 Github 上查看完整的[源代码。](https://web.archive.org/web/20221107223704/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate5)