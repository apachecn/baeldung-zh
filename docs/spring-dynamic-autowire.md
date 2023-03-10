# 如何在 Spring 中动态自动连接 Bean

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-dynamic-autowire>

## 1.介绍

在这个简短的教程中，我们将展示如何在 Spring 中动态自动连接 bean。

我们将从展示一个真实的用例开始，在这个用例中，动态自动布线可能会有所帮助。除此之外，我们将通过两种不同的方式展示如何在 Spring 中解决它。

## 2.动态自动布线用例

在我们需要动态改变 Spring 的 bean 执行逻辑的地方，动态自动连接很有帮助。这很实用，尤其是在根据一些运行时变量选择要执行的代码的地方。

为了演示一个真实的用例，让我们创建一个控制世界不同地区服务器的应用程序。出于这个原因，我们用两个简单的方法创建了一个接口:

```java
public interface RegionService {
    boolean isServerActive(int serverId);

    String getISOCountryCode();
}
```

和两种实现方式:

```java
@Service("GBregionService")
public class GBRegionService implements RegionService {
    @Override
    public boolean isServerActive(int serverId) {
        return false;
    }

    @Override
    public String getISOCountryCode() {
        return "GB";
    }
}
```

```java
@Service("USregionService")
public class USRegionService implements RegionService {
    @Override
    public boolean isServerActive(int serverId) {
        return true;
    }

    @Override
    public String getISOCountryCode() {
        return "US";
    }
}
```

假设我们有一个网站，用户可以选择检查服务器在所选区域是否处于活动状态。因此，**我们希望有一个服务类，它可以根据用户**的输入动态地改变`RegionService`接口的实现。毫无疑问，这是动态 bean 自动连接发挥作用的用例。

## 3.使用`BeanFactory`

`BeanFactory` 是用于访问 Spring bean 容器的根接口。特别是，它包含了获取特定 beans 的有用方法。由于`BeanFactory`也是一个 Spring bean，我们可以自动连接并直接在我们的类中使用它:

```java
@Service
public class BeanFactoryDynamicAutowireService {
    private static final String SERVICE_NAME_SUFFIX = "regionService";
    private final BeanFactory beanFactory;

    @Autowired
    public BeanFactoryDynamicAutowireService(BeanFactory beanFactory) {
        this.beanFactory = beanFactory;
    }

    public boolean isServerActive(String isoCountryCode, int serverId) {
        RegionService service = beanFactory.getBean(getRegionServiceBeanName(isoCountryCode), 
          RegionService.class);

        return service.isServerActive(serverId);
    }

    private String getRegionServiceBeanName(String isoCountryCode) {
        return isoCountryCode + SERVICE_NAME_SUFFIX;
    }
}
```

我们使用了一个重载版本的`getBean() `方法来获取具有给定名称和期望类型的 bean。

虽然这很有效，但我们真的更愿意依赖一些更习惯的东西；也就是使用依赖注入的东西。

## 4.使用接口

为了用依赖注入解决这个问题，我们将依赖 Spring 的一个鲜为人知的特性。

除了标准的单字段自动连接， **Spring **还让我们能够将所有实现特定接口的 beans 收集到一个`Map`**** 中:

```java
@Service
public class CustomMapFromListDynamicAutowireService {
    private final Map<String, RegionService> servicesByCountryCode;

    @Autowired
    public CustomMapFromListDynamicAutowireService(List<RegionService> regionServices) {
        servicesByCountryCode = regionServices.stream()
                .collect(Collectors.toMap(RegionService::getISOCountryCode, Function.identity()));
    }

    public boolean isServerActive(String isoCountryCode, int serverId) {
        RegionService service = servicesByCountryCode.get(isoCountryCode);

        return service.isServerActive(serverId);
    }
}
```

我们已经在一个构造函数中创建了一个映射，它按照国家代码保存实现。此外，我们可以稍后在方法中使用它来获得特定的实现，以检查给定的服务器在特定的区域中是否是活动的。

## 5.结论

在这个快速教程中，我们看到了如何使用两种不同的方法在 Spring 中动态自动连接 bean。

和往常一样，本文中显示的代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221005074900/https://github.com/eugenp/tutorials/tree/master/spring-core-4)