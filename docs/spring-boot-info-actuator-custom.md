# Spring Boot 信息端点中的自定义信息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-info-actuator-custom>

## 1。概述

在这篇简短的文章中，我们将了解如何定制 Spring Boot 执行器的`/info`端点。

请[参考本文](/web/20220529021632/https://www.baeldung.com/spring-boot-actuators)了解更多关于 Boot 中的致动器以及如何配置它们。

## 2。/info 中的静态属性

如果我们有一些静态信息，比如应用程序的名称或它的版本很长一段时间不会改变，那么在我们的`application.properties`文件中添加这些细节是一个好主意:

```java
## Configuring info endpoint
info.app.name=Spring Sample Application
info.app.description=This is my first spring boot application
info.app.version=1.0.0
```

这就是我们要使这些数据在`/info`端点上可用所要做的一切。Spring 会自动将所有前缀为`info`的属性添加到`/info`端点:

```java
{
  "app": {
    "description": "This is my first spring boot application",
    "version": "1.0.0",
    "name": "Spring Sample Application"
  }
}
```

## 3。/info 中的环境变量

现在让我们在我们的`/info`端点中公开一个`Environment`变量:

```java
info.java-vendor = ${java.specification.vendor}
```

这将把 Java 供应商暴露给我们的`/info`端点:

```java
{
  "app": {
    "description": "This is my first spring boot application",
    "version": "1.0.0",
    "name": "Spring Sample Application"
  },
  "java-vendor": "Oracle Corporation",
}
```

请注意，所有的环境变量都已经在`/env`端点上可用。然而，同样可以在`/info`端点上快速公开。

## 4。来自持久层的自定义数据

现在让我们更进一步，从持久性存储中公开一些有用的数据。

为此，我们需要实现`InfoContributor`接口并覆盖`contribute()`方法:

```java
@Component
public class TotalUsersInfoContributor implements InfoContributor {

    @Autowired
    UserRepository userRepository;

    @Override
    public void contribute(Info.Builder builder) {
        Map<String, Integer> userDetails = new HashMap<>();
        userDetails.put("active", userRepository.countByStatus(1));
        userDetails.put("inactive", userRepository.countByStatus(0));

        builder.withDetail("users", userDetails);
    }
}
```

首先，我们需要将实现类标记为`@Component`。然后将所需的细节添加到提供给`contribute()`方法的`Info.Builder`实例中。

这种方法为我们向`/info`端点公开什么提供了很大的灵活性:

```java
{
  ...other /info data...,
  ...
  "users": {
    "inactive": 2,
    "active": 3
  }
}
```

## 5。结论

在本教程中，我们研究了向我们的`/info`端点添加定制数据的各种方法。

注意，我们还在讨论如何将 [git 信息](/web/20220529021632/https://www.baeldung.com/spring-git-information)添加到`/info`端点中。

和往常一样，本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220529021632/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-actuator)