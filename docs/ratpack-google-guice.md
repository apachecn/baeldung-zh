# ratpack 谷歌果汁集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ratpack-google-guice>

## 1。概述

在我们之前的[文章](/web/20220626203748/https://www.baeldung.com/ratpack)中，我们展示了如何使用 Ratpack 构建可伸缩的应用程序。

在本教程中，我们将进一步讨论如何使用`Google Guice`和 [Ratpack](https://web.archive.org/web/20220626203748/https://ratpack.io/) 作为依赖管理引擎。

## 2。为什么使用 Google Guice？

[`Google Guice`](https://web.archive.org/web/20220626203748/https://github.com/google/guice) 是`Apache License`旗下`Google`发布的`Java`平台开源软件框架。

这是一个非常轻量级的依赖管理模块，易于配置。此外，为了方便使用，它只允许构造函数级的依赖注入。

更多关于`Guice` 的细节可以在[这里](/web/20220626203748/https://www.baeldung.com/guice)找到。

## 3。将 Guice 与 Ratpack 一起使用

### 3.1。Maven 依赖关系

Ratpack 对`Guice`依赖有一流的支持。因此，我们不需要为`Guice;` 手动添加任何外部依赖，它已经与`Ratpack`一起预建了。关于`Ratpack`的`Guice`支持的更多细节可以在[这里](https://web.archive.org/web/20220626203748/https://ratpack.io/manual/current/guice.html)找到。

因此，我们只需要在`pom.xml`中添加以下核心`Ratpack`依赖关系:

```java
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-core</artifactId>
    <version>1.4.5</version>
</dependency>
```

你可以在 [Maven](https://web.archive.org/web/20220626203748/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.ratpack%22%20AND%20a%3A%22ratpack-core%22) [Central](https://web.archive.org/web/20220626203748/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.ratpack%22%20AND%20a%3A%22ratpack-core%22) 上查看最新版本。

### 3.2。建筑服务模块

一旦完成了`Maven`配置，我们将构建一个服务，并在这里的例子中充分利用一些简单的依赖注入。

让我们创建一个服务接口和一个服务类:

```java
public interface DataPumpService {
    String generate();
}
```

这是将充当注入器的服务接口。现在，我们必须构建服务类来实现它，并定义服务方法`generate():`

```java
public class DataPumpServiceImpl implements DataPumpService {

    @Override
    public String generate() {
        return UUID.randomUUID().toString();
    }

}
```

这里需要注意的重要一点是，由于我们使用的是`Ratpack's Guice`模块、**，我们不需要使用`Guice`的`@ImplementedBy`或`@Inject`注释来手工注入服务类。**

### 3.3。依赖性管理

使用`Google Guice`有两种方式来执行依赖管理。

第一种是使用`Guice`的 [`AbstractModule`](https://web.archive.org/web/20220626203748/https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/AbstractModule.html) ，其他的是使用 Guice 的[实例绑定](https://web.archive.org/web/20220626203748/https://github.com/google/guice/wiki/InstanceBindings)机制的方法:

```java
public class DependencyModule extends AbstractModule {

    @Override
    public void configure() {
        bind(DataPumpService.class).to(DataPumpServiceImpl.class)
          .in(Scopes.SINGLETON);
    }

}
```

这里需要注意几点:

*   通过扩展`AbstractModule`，我们覆盖了默认的`configure()`方法
*   我们将`DataPumpServiceImpl`类映射到`DataPumpService`接口，这是之前构建的服务层
*   我们还以`Singleton`的方式注入了依赖关系。

### 3.4。与现有应用集成

既然依赖关系管理配置已经准备好了，现在让我们集成它:

```java
public class Application {

    public static void main(String[] args) throws Exception {

      RatpackServer
          .start(server -> server.registry(Guice
            .registry(bindings -> bindings.module(DependencyModule.class)))
            .handlers(chain -> chain.get("randomString", ctx -> {
                DataPumpService dataPumpService = ctx.get(DataPumpService.class);
                ctx.render(dataPumpService.generate().length());
            })));
    }
}
```

在这里，我们用`registry()`绑定了扩展`AbstractModule`的`DependencyModule`类。`Ratpack's Guice`模块将在内部完成剩余的工作，并在应用程序中注入服务`Context`。

因为它在`application-context,`中可用，所以我们现在可以从应用程序的任何地方获取服务实例。这里，我们从当前上下文中获取了`DataPumpService`实例，并用服务的`generate()`方法映射了`/randomString` URL。

**因此，每当点击`/randomString` URL 时，都会返回随机的字符串片段。**

### 3.5。运行时实例绑定

如前所述，我们现在将使用 Guice 的实例绑定机制在运行时进行依赖管理。除了使用 Guice 的`bindInstance()`方法代替`AbstractModule`来注入依赖关系之外，它几乎与前面的技术一样:

```java
public class Application {

    public static void main(String[] args) throws Exception {

      RatpackServer.start(server -> server
        .registry(Guice.registry(bindings -> bindings
        .bindInstance(DataPumpService.class, new DataPumpServiceImpl())))
        .handlers(chain -> chain.get("randomString", ctx -> {
            DataPumpService dataPumpService = ctx.get(DataPumpService.class);
            ctx.render(dataPumpService.generate());
        })));
    }
}
```

这里，通过使用`bindInstance()`，我们正在执行实例绑定，即将`DataPumpService`接口注入到`DataPumpServiceImpl`类中。

通过这种方式，我们可以像前面的例子一样将服务实例注入到`application-context`中。

尽管我们可以使用这两种技术中的任何一种来进行依赖性管理，但是使用`AbstractModule`总是更好，因为它将依赖性管理模块与应用程序代码完全分离。这样，代码将会更加清晰，将来也更容易维护。

### 3.6。工厂绑定

还有一种叫做`factory binding`的依赖性管理方法。它与`Guice's`的实现没有直接关系，但也可以与`Guice`并行工作。

工厂类将客户端从实现中分离出来。一个简单的工厂使用静态方法来获取和设置接口的模拟实现。

我们可以使用已经创建的服务类来启用工厂绑定。我们只需要创建一个类似于`DependencyModule`的工厂类(它扩展了`Guice's AbstractModule`类)并通过静态方法绑定实例:

```java
public class ServiceFactory {

    private static DataPumpService instance;

    public static void setInstance(DataPumpService dataPumpService) {
        instance = dataPumpService;
    }

    public static DataPumpService getInstance() {
        if (instance == null) {
            return new DataPumpServiceImpl();
        }
        return instance;
    }
}
```

**这里，我们在工厂类**中静态注入服务接口。因此，一次只有该接口的一个实例可用于该工厂类。然后，我们创建了普通的`getter/setter`方法来设置和获取服务实例。

这里要注意的一点是，在`getter`方法中，我们进行了一次显式检查，以确保只有一个服务实例存在或不存在；如果为空，那么只有我们创建了实现服务类的实例，并返回相同的结果。

此后，我们可以在应用程序链中使用这个工厂实例:

```java
.get("factory", ctx -> ctx.render(ServiceFactory.getInstance().generate()))
```

## 4。测试

我们将使用`Ratpack`的[mainclassapplicationundest](https://web.archive.org/web/20220626203748/https://ratpack.io/manual/current/api/ratpack/test/MainClassApplicationUnderTest.html)在`Ratpack`的内部 JUnit 测试框架的帮助下测试我们的应用程序。我们必须为它添加必要的依赖项( [ratpack-test](https://web.archive.org/web/20220626203748/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.ratpack%22%20AND%20a%3A%22ratpack-test%22) )。

这里要注意的一点是，由于 URL 内容是动态的，我们在编写测试用例时无法预测它。因此，我们将匹配测试用例中的`/randomString` URL 端点的内容长度:

```java
@RunWith(JUnit4.class)
public class ApplicationTest {

    MainClassApplicationUnderTest appUnderTest
      = new MainClassApplicationUnderTest(Application.class);

    @Test
    public void givenStaticUrl_getDynamicText() {
        assertEquals(21, appUnderTest.getHttpClient()
          .getText("/randomString").length());
    }

    @After
    public void shutdown() {
        appUnderTest.close();
    }
}
```

## 5。结论

在这篇简短的文章中，我们展示了如何使用`Google Guice`和`Ratpack`。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626203748/https://github.com/eugenp/tutorials/tree/master/ratpack)