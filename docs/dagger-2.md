# 匕首 2 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/dagger-2>

## 1。简介

在本教程中，我们将看看 Dagger 2——一个快速、轻量级的依赖注入框架。

该框架可用于 Java 和 Android，但编译时注入带来的高性能使其成为后者的领先解决方案。

## 2。依赖注入

稍微提醒一下，[依赖注入](/web/20221023105416/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)是更一般的控制原则反转的具体应用，其中程序的流程由程序本身控制。

它是通过一个外部组件实现的，该组件提供了其他对象所需的对象实例(或依赖关系)。

不同的框架以不同的方式实现依赖注入。特别是，这些差异中最显著的一个是注入是发生在运行时还是编译时。

运行时 DI 通常基于反射，使用起来更简单，但在运行时速度较慢。运行时 DI 框架的一个例子是 Spring。

另一方面，编译时 DI 是基于代码生成的。这意味着所有重量级操作都在编译期间执行。编译时 DI 增加了复杂性，但通常执行速度更快。

匕首 2 就属于这一类。

## 3。Maven/Gradle 配置

为了在项目中使用 Dagger，我们需要将[的`dagger`依赖](https://web.archive.org/web/20221023105416/https://search.maven.org/classic/#artifactdetails%7Ccom.google.dagger%7Cdagger%7C2.16%7Cjar)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>com.google.dagger</groupId>
    <artifactId>dagger</artifactId>
    <version>2.16</version>
</dependency>
```

此外，我们还需要[包含 Dagger 编译器](https://web.archive.org/web/20221023105416/https://search.maven.org/classic/#artifactdetails%7Ccom.google.dagger%7Cdagger-compiler%7C2.16%7Cjar)，用于将我们的注释类转换成用于注入的代码:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.6.1</version>
    <configuration>
         <annotationProcessorPaths>
              <path>
                  <groupId>com.google.dagger</groupId>
                  <artifactId>dagger-compiler</artifactId>
                  <version>2.16</version>
              </path>
         </annotationProcessorPaths>
    </configuration>
</plugin>
```

通过这种配置，Maven 会将生成的代码输出到`target/generated-sources/annotations`中。

出于这个原因，**如果我们想使用它的代码完成特性，我们可能需要进一步配置我们的 IDE** 。一些 ide 直接支持注释处理器，而另一些 ide 可能需要我们将这个目录添加到构建路径中。

或者，如果我们在 Gradle 上使用 Android，我们可以包含两个依赖项:

```java
compile 'com.google.dagger:dagger:2.16'
annotationProcessor 'com.google.dagger:dagger-compiler:2.16'
```

现在我们的项目中有了 Dagger，让我们创建一个示例应用程序来看看它是如何工作的。

## 4。实施

在我们的例子中，我们将尝试通过注射零部件来制造汽车。

现在，**匕首在很多地方使用标准的 JSR-330 注解**，一个是`@Inject.`

我们可以给字段或构造函数添加注释。但是，由于 **Dagger 不支持私有字段**上的注入，我们将使用构造函数注入来保持封装:

```java
public class Car {

    private Engine engine;
    private Brand brand;

    @Inject
    public Car(Engine engine, Brand brand) {
        this.engine = engine;
        this.brand = brand;
    }

    // getters and setters

}
```

接下来，我们将实现代码来执行注入。更具体地说，我们将创建:

*   一个**模块**，它是一个提供或构建对象依赖关系的类，以及
*   一个**组件**，它是一个用来生成注射器的接口

复杂的项目可能包含多个模块和组件，但是因为我们正在处理一个非常基本的程序，所以每一个都足够了。

让我们看看如何实现它们。

### 4.1。模块

为了创建一个模块，**我们需要用`@Module` 注释**来注释这个类。此注释表明该类可以使依赖项对容器可用:

```java
@Module
public class VehiclesModule {
}
```

然后，**我们需要在构造依赖关系的方法上添加`@Provides`注释**:

```java
@Module
public class VehiclesModule {
    @Provides
    public Engine provideEngine() {
        return new Engine();
    }

    @Provides
    @Singleton
    public Brand provideBrand() { 
        return new Brand("Baeldung"); 
    }
}
```

另外，请注意，我们可以配置给定依赖项的范围。在这种情况下，我们将 singleton 范围赋予我们的`Brand`实例，这样所有的 car 实例共享同一个 brand 对象。

### 4.2。组件

接下来，我们将创建我们的组件接口`. `，这个类将生成 Car 实例，注入由`VehiclesModule`提供的依赖关系。

简单地说，我们需要一个返回`Car` 和**的方法签名，我们需要用`@Component`注释**来标记这个类:

```java
@Singleton
@Component(modules = VehiclesModule.class)
public interface VehiclesComponent {
    Car buildCar();
}
```

注意我们是如何将模块类作为参数传递给`@Component`注释的。**如果我们不这样做，Dagger 就不会知道如何构建汽车的依赖项。**

此外，由于我们的模块提供了一个 singleton 对象，我们必须给我们的组件相同的作用域，因为 **Dagger 不允许未作用域的组件引用作用域绑定**。

### 4.3。客户代码

最后，我们可以运行`mvn compile`来触发注释处理器并生成注入器代码。

之后，我们将找到与接口同名的组件实现，只是前缀为“`Dagger`”:

```java
@Test
public void givenGeneratedComponent_whenBuildingCar_thenDependenciesInjected() {
    VehiclesComponent component = DaggerVehiclesComponent.create();

    Car carOne = component.buildCar();
    Car carTwo = component.buildCar();

    Assert.assertNotNull(carOne);
    Assert.assertNotNull(carTwo);
    Assert.assertNotNull(carOne.getEngine());
    Assert.assertNotNull(carTwo.getEngine());
    Assert.assertNotNull(carOne.getBrand());
    Assert.assertNotNull(carTwo.getBrand());
    Assert.assertNotEquals(carOne.getEngine(), carTwo.getEngine());
    Assert.assertEquals(carOne.getBrand(), carTwo.getBrand());
}
```

## 5。弹簧类比

熟悉 Spring 的人可能已经注意到了这两个框架之间的一些相似之处。

Dagger 的`@Module`注释让容器以一种与 Spring 的任何原型注释非常相似的方式来识别一个类(例如，`@Service`、`@Controller`……)。同样，`@Provides`和`@Component`分别几乎相当于春天的`@Bean`和`@Lookup`。

Spring 也有自己的`@Scope`注释，与`@Singleton`相关联，尽管这里已经注意到另一个不同之处，Spring 默认采用单例作用域，而 Dagger 默认采用 Spring 开发人员可能称之为原型作用域的作用域，每次需要依赖项时调用 provider 方法。

## 6。结论

在本文中，我们通过一个基本的例子介绍了如何设置和使用 Dagger 2。我们还考虑了运行时注入和编译时注入之间的差异。

和往常一样，文章中的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221023105416/https://github.com/eugenp/tutorials/tree/master/dagger)