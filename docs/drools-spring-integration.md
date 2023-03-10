# Drools Spring 集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/drools-spring-integration>

## 1。简介

在这个快速教程中，我们将把 Drools 和 Spring 集成在一起。如果你刚刚开始使用 Drools，[看看这篇介绍文章。](/web/20220805084651/https://www.baeldung.com/drools)

## 2。Maven 依赖关系

让我们从将以下依赖项添加到我们的`pom.xml`文件开始:

```java
<dependency>
    <groupId>org.drools</groupId>
    <artifactId>drools-core</artifactId>
    <version>7.0.0.Final</version>
</dependency>
<dependency>
    <groupId>org.kie</groupId>
    <artifactId>kie-spring</artifactId>
    <version>7.0.0.Final</version>
</dependency>
```

最新版本可以在这里找到 [drools-core](https://web.archive.org/web/20220805084651/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.drools%22%20AND%20a%3A%22drools-core%22) 和 [kie-spring](https://web.archive.org/web/20220805084651/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.kie%22%20AND%20a%3A%22kie-spring%22) 的版本。

## 3。初始数据

现在让我们定义将在我们的例子中使用的数据。我们将根据行驶距离和夜间附加费标志来计算乘车费用。

这里有一个简单的对象，它将被用作一个`Fact:`

```java
public class TaxiRide {
    private Boolean isNightSurcharge;
    private Long distanceInMile;

    // standard constructors, getters/setters
}
```

让我们还定义另一个业务对象，它将用于表示票价:

```java
public class Fare {
    private Long nightSurcharge;
    private Long rideFare;

    // standard constructors, getters/setters
}
```

现在，让我们定义一个计算出租车费用的业务规则:

```java
global com.baeldung.spring.drools.model.Fare rideFare;
dialect  "mvel"

rule "Calculate Taxi Fare - Scenario 1"
    when
        taxiRideInstance:TaxiRide(isNightSurcharge == false && distanceInMile < 10);
    then
      	rideFare.setNightSurcharge(0);
       	rideFare.setRideFare(70);
end 
```

我们可以看到，定义了一个规则来计算给定的`TaxiRide`的总费用。

该规则接受一个`TaxiRide`对象，并检查`isNightSurcharge`属性是否为 *false* 并且`distanceInMile`属性值是否小于 10，然后将票价计算为 70 并将`nightSurcharge` 属性设置为 0。

计算的输出被设置到`Fare`对象以备将来使用。

## 4。弹簧集成

### 4.1。弹簧豆配置

现在，让我们继续讨论 Spring 集成。

我们将定义一个 Spring bean 配置类，它将负责实例化`TaxiFareCalculatorService` bean 及其依赖项:

```java
@Configuration
@ComponentScan("com.baeldung.spring.drools.service")
public class TaxiFareConfiguration {
    private static final String drlFile = "TAXI_FARE_RULE.drl";

    @Bean
    public KieContainer kieContainer() {
        KieServices kieServices = KieServices.Factory.get();

        KieFileSystem kieFileSystem = kieServices.newKieFileSystem();
        kieFileSystem.write(ResourceFactory.newClassPathResource(drlFile));
        KieBuilder kieBuilder = kieServices.newKieBuilder(kieFileSystem);
        kieBuilder.buildAll();
        KieModule kieModule = kieBuilder.getKieModule();

        return kieServices.newKieContainer(kieModule.getReleaseId());
    }
} 
```

`KieServices`是一个 singleton，作为获取 Kie 提供的所有服务的单点入口。使用`KieServices.Factory.get().`检索`KieServices`

接下来，我们需要得到`KieContainer` ，它是运行规则引擎所需的所有对象的占位符。

`KieContainer` 是在其他豆子的帮助下建造的，包括`KieFileSystem, KieBuilder,` 和 `KieModule.`

让我们继续创建一个`KieModule`,它是定义称为`KieBase.` 的规则知识所需的所有资源的容器

```java
KieModule kieModule = kieBuilder.getKieModule();
```

> `KieBase`是一个知识库，包含所有与应用程序相关的知识，如规则、流程、功能、类型模型，它隐藏在`KieModule`中。`KieBase`可以从`KieContainer.`中获得

一旦`KieModule`被创建，我们可以继续创建`KieContainer`-**-**，其中包含定义了`KieBase`的`KieModule`。`KieContainer`是使用一个模块创建的:

```java
KieContainer kContainer = kieServices.newKieContainer(kieModule.getReleaseId());
```

### 4.2。春季服务

让我们定义一个服务类，它通过将`Fact`对象传递给引擎来处理结果，从而执行实际的业务逻辑:

```java
@Service
public class TaxiFareCalculatorService {

    @Autowired
    private KieContainer kieContainer;

    public Long calculateFare(TaxiRide taxiRide, Fare rideFare) {
        KieSession kieSession = kieContainer.newKieSession();
        kieSession.setGlobal("rideFare", rideFare);
        kieSession.insert(taxiRide);
        kieSession.fireAllRules();
        kieSession.dispose();
        return rideFare.getTotalFare();
    }
} 
```

最后，使用`KieContainer`实例创建一个`KieSession`。一个`KieSession`实例是一个可以插入输入数据的地方。`KieSession`与引擎交互，根据插入的事实处理规则中定义的实际业务逻辑。

Global(就像一个全局变量)用于将信息传递给引擎。我们可以使用`setGlobal(“key”, value);`来设置全局。在本例中，我们将`Fare`对象设置为全局，以存储计算出的出租车费用。

正如我们在第 4 节中讨论的， **a `Rule`需要数据在**上操作。我们使用`kieSession`T3 将`Fact`插入到会话中

一旦我们完成了输入`Fact,`的设置，我们就可以通过调用`fireAllRules().`来请求引擎执行业务逻辑

最后，我们需要通过调用`dispose()`方法来清理会话以避免内存泄漏。

## 5。行动范例

现在，我们可以连接一个 Spring 上下文，并看到 Drools 按预期工作:

```java
@Test
public void whenNightSurchargeFalseAndDistLessThan10_thenFixWithoutNightSurcharge() {
    TaxiRide taxiRide = new TaxiRide();
    taxiRide.setIsNightSurcharge(false);
    taxiRide.setDistanceInMile(9L);
    Fare rideFare = new Fare();
    Long totalCharge = taxiFareCalculatorService.calculateFare(taxiRide, rideFare);

    assertNotNull(totalCharge);
    assertEquals(Long.valueOf(70), totalCharge);
}
```

## 6。结论

在本文中，我们通过一个简单的用例了解了 Drools Spring 集成。

与往常一样，GitHub 上的[提供了示例和代码片段的实现。](https://web.archive.org/web/20220805084651/https://github.com/eugenp/tutorials/tree/master/spring-drools)