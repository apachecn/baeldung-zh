# 休眠错误“没有 EntityManager 的持久性提供程序”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-no-persistence-provider>

## 1.介绍

在本教程中，我们将看到如何解决一个常见的 [Hibernate](/web/20220627080255/https://www.baeldung.com/tag/hibernate/) 错误——“没有 EntityManager 的持久性提供者”。简单地说，持久性提供者指的是我们的应用程序中使用的特定 JPA 实现，用于将对象持久化到数据库中。

要了解更多关于 JPA 及其实现的信息，我们可以参考我们关于 JPA、Hibernate 和 EclipseLink 之间的[差异的文章。](/web/20220627080255/https://www.baeldung.com/jpa-hibernate-difference)

## 2.什么导致了错误

当**应用程序不知道应该使用哪个** **持久性提供者**时，我们将看到错误。

当持久化提供者既没有在`persistence.xml`文件中提到，也没有在`PersistenceUnitInfo` 实现类中配置时，就会出现这种情况。

## 3.修复错误

要修复这个错误，我们只需要在`persistence.xml` 文件中**定义持久性提供者:**

```java
<provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
```

或者，如果我们使用的是 **Hibernate 版本 4.2 或更早的版本**:

```java
<provider>org.hibernate.ejb.HibernatePersistence</provider>
```

如果我们已经在应用程序中实现了`PersistenceUnitInfo`接口，我们还必须覆盖
`getPersistenceProviderClassName()`方法:

```java
@Override
public String getPersistenceProviderClassName() {
    return HibernatePersistenceProvider.class.getName();
}
```

为了确保所有必需的 Hibernate jars 都可用，在`pom.xml`文件中添加 [`hibernate-core`](https://web.archive.org/web/20220627080255/https://search.maven.org/artifact/org.hibernate/hibernate-core) 依赖项很重要:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>${hibernate.version}</version>
</dependency>
```

## 4.结论

总而言之，我们已经看到了 [Hibernate](/web/20220627080255/https://www.baeldung.com/tag/hibernate/) 错误“没有 EntityManager 的持久性提供者”的可能原因以及解决它的各种方法。

像往常一样，示例 Hibernate 项目可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220627080255/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-exceptions)