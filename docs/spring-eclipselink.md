# 春日食链指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-eclipselink>

## 1。概述

默认情况下，Spring Data 使用 Hibernate 作为默认的 JPA 实现提供者。

然而，Hibernate 肯定不是我们唯一可用的 JPA 实现。

在本文中，我们将通过必要的步骤来设置 [`EclipseLink`](https://web.archive.org/web/20220529012536/https://www.eclipse.org/eclipselink/) 作为 Spring 数据 JPA 的实现提供者。

## 2。Maven 依赖关系

要在我们的 Spring 应用程序中使用它，我们只需要在我们项目的`pom.xml`中添加 [`org.eclipse.persistence.jpa`](https://web.archive.org/web/20220529012536/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22org.eclipse.persistence.jpa%22) 依赖项:

```java
<dependency>
    <groupId>org.eclipse.persistence</groupId>
    <artifactId>org.eclipse.persistence.jpa</artifactId>
    <version>2.7.0</version>
</dependency>
```

默认情况下，Spring 数据来自 Hibernate 实现。

因为我们想使用`EclipseLink`作为 JPA 提供者，所以我们不再需要它了。

因此，我们可以通过排除它的依赖项来将其从项目中删除:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

下一步是告诉 Spring 框架我们想要使用`EclipseLink`作为 JPA 实现。

## 3。弹簧配置

**`JpaBaseConfiguration`是一个抽象类，为 Spring Boot 的 JPA** 定义 beans。为了定制它，我们必须实现一些方法，比如`createJpaVendorAdapter()`或者`getVendorProperties()`。

Spring 为 Hibernate 提供了一个名为`HibernateJpaAutoConfiguration`的开箱即用的配置实现。然而，对于`EclipseLink,`,我们必须创建一个定制配置。

首先，我们需要实现`createJpaVendorAdapter()` 方法，它指定了要使用的 JPA 实现。

Spring 为名为`EclipseLinkJpaVendorAdapter` 的`EclipseLink`提供了`AbstractJpaVendorAdapter`的**实现，我们将在我们的方法中使用它:**

```java
@Configuration 
public class EclipseLinkJpaConfiguration extends JpaBaseConfiguration { 

    @Override 
    protected AbstractJpaVendorAdapter createJpaVendorAdapter() { 
        return new EclipseLinkJpaVendorAdapter(); 
    }

    //...
}
```

此外，我们必须定义一些将被 EclipseLink 使用的特定于供应商的属性。

我们可以通过`getVendorProperties()`方法添加这些:

```java
@Override
protected Map<String, Object> getVendorProperties() {
    HashMap<String, Object> map = new HashMap<>();
    map.put(PersistenceUnitProperties.WEAVING, true);
    map.put(PersistenceUnitProperties.DDL_GENERATION, "drop-and-create-tables");
    return map;
}
```

类`org.eclipse.persistence.config.PersistenceUnitProperties` 包含我们可以为`EclipseLink.`定义的属性

在这个例子中，我们已经指定了我们希望在应用程序运行时使用编织并重新创建数据库模式。

**就这样！**这是从默认 Hibernate JPA 提供者更改为`EclipseLink.` 所需的全部实现

注意，Spring 数据使用 JPA API，而不是任何特定于供应商的方法。所以，理论上，从一家厂商换到另一家应该没有问题。

## 4。结论

在这个快速教程中，我们介绍了如何更改 Spring Data 使用的默认 JPA 实现提供者。

我们看到了从默认的 Hibernate 切换到`EclipseLink.`是多么的快速和简单

与往常一样，Github 上的[提供了示例的完整实现。](https://web.archive.org/web/20220529012536/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-eclipselink)