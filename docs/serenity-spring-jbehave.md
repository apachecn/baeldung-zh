# Serenity BDD 与 Spring 和 JBehave

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/serenity-spring-jbehave>

## 1。简介

之前，我们已经[介绍了 Serenity BDD 框架](/web/20220626204235/https://www.baeldung.com/serenity-bdd)。

在本文中，我们将介绍如何将 Serenity BDD 与 Spring 集成。

## 2。Maven 依赖关系

为了在我们的 Spring 项目中启用 Serenity，我们需要将`[serenity-core](https://web.archive.org/web/20220626204235/https://search.maven.org/classic/#artifactdetails%7Cnet.serenity-bdd%7Cserenity-core%7C1.4.0%7Cjar)`和`[serenity-spring](https://web.archive.org/web/20220626204235/https://search.maven.org/classic/#artifactdetails%7Cnet.serenity-bdd%7Cserenity-spring%7C1.4.0%7Cjar)`添加到`pom.xml`中:

```java
<dependency>
    <groupId>net.serenity-bdd</groupId>
    <artifactId>serenity-core</artifactId>
    <version>1.4.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>net.serenity-bdd</groupId>
    <artifactId>serenity-spring</artifactId>
    <version>1.4.0</version>
    <scope>test</scope>
</dependency>
```

我们还需要配置 [`serenity-maven-plugin`](https://web.archive.org/web/20220626204235/https://search.maven.org/classic/#artifactdetails%7Cnet.serenity-bdd.maven.plugins%7Cserenity-maven-plugin%7C1.4.0%7Cjar) ，这对生成 Serenity 测试报告很重要:

```java
<plugin>
    <groupId>net.serenity-bdd.maven.plugins</groupId>
    <artifactId>serenity-maven-plugin</artifactId>
    <version>1.4.0</version>
    <executions>
        <execution>
            <id>serenity-reports</id>
            <phase>post-integration-test</phase>
            <goals>
                <goal>aggregate</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## 3。弹簧集成

春季综合测试需要`@RunWith` [`SpringJUnit4ClassRunner`](https://web.archive.org/web/20220626204235/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/context/junit4/SpringJUnit4ClassRunner.html) 。但是我们不能直接用 Serenity 运行测试程序，因为 Serenity 测试需要由`SerenityRunner`运行。

对于 Serenity 测试，我们可以使用`SpringIntegrationMethodRule`和`SpringIntegrationClassRule`来启用注入。

我们将基于一个简单的场景进行测试:给定一个数字，当添加另一个数字时，返回总和。

### 3.1。`SpringIntegrationMethodRule`

`SpringIntegrationMethodRule`是一种 [`MethodRule`](https://web.archive.org/web/20220626204235/http://junit.org/junit4/javadoc/4.12/org/junit/rules/MethodRule.html) 应用于测试的方法。Spring 上下文将在`@Before`之前和`@BeforeClass`之后构建。

假设我们有一个属性要注入到 beans 中:

```java
<util:properties id="props">
    <prop key="adder">4</prop>
</util:properties>
```

现在让我们添加`SpringIntegrationMethodRule`来在我们的测试中启用值注入:

```java
@RunWith(SerenityRunner.class)
@ContextConfiguration(locations = "classpath:adder-beans.xml")
public class AdderMethodRuleIntegrationTest {

    @Rule 
    public SpringIntegrationMethodRule springMethodIntegration 
      = new SpringIntegrationMethodRule();

    @Steps 
    private AdderSteps adderSteps;

    @Value("#{props['adder']}") 
    private int adder;

    @Test
    public void givenNumber_whenAdd_thenSummedUp() {
        adderSteps.givenNumber();
        adderSteps.whenAdd(adder);
        adderSteps.thenSummedUp(); 
    }
}
```

它还支持`spring test`的方法级注释。如果某些测试方法弄脏了测试上下文，我们可以在上面标记 [`@DirtiesContext`](https://web.archive.org/web/20220626204235/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/annotation/DirtiesContext.html) :

```java
@RunWith(SerenityRunner.class)
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
@ContextConfiguration(classes = AdderService.class)
public class AdderMethodDirtiesContextIntegrationTest {

    @Steps private AdderServiceSteps adderServiceSteps;

    @Rule public SpringIntegrationMethodRule springIntegration = new SpringIntegrationMethodRule();

    @DirtiesContext
    @Test
    public void _0_givenNumber_whenAddAndAccumulate_thenSummedUp() {
        adderServiceSteps.givenBaseAndAdder(randomInt(), randomInt());
        adderServiceSteps.whenAccumulate();
        adderServiceSteps.summedUp();

        adderServiceSteps.whenAdd();
        adderServiceSteps.sumWrong();
    }

    @Test
    public void _1_givenNumber_whenAdd_thenSumWrong() {
        adderServiceSteps.whenAdd();
        adderServiceSteps.sumWrong();
    }

}
```

在上面的例子中，当我们调用`adderServiceSteps.whenAccumulate()`时，`adderServiceSteps`中注入的`@Service`的基数字段将会改变:

```java
@ContextConfiguration(classes = AdderService.class)
public class AdderServiceSteps {

    @Autowired
    private AdderService adderService;

    private int givenNumber;
    private int base;
    private int sum;

    public void givenBaseAndAdder(int base, int adder) {
        this.base = base;
        adderService.baseNum(base);
        this.givenNumber = adder;
    }

    public void whenAdd() {
        sum = adderService.add(givenNumber);
    }

    public void summedUp() {
        assertEquals(base + givenNumber, sum);
    }

    public void sumWrong() {
        assertNotEquals(base + givenNumber, sum);
    }

    public void whenAccumulate() {
        sum = adderService.accumulate(givenNumber);
    }

}
```

具体来说，我们将总和分配给基数:

```java
@Service
public class AdderService {

    private int num;

    public void baseNum(int base) {
        this.num = base;
    }

    public int currentBase() {
        return num;
    }

    public int add(int adder) {
        return this.num + adder;
    }

    public int accumulate(int adder) {
        return this.num += adder;
    }
}
```

在第一个测试`_0_givenNumber_whenAddAndAccumulate_thenSummedUp`中，基数被改变，使得上下文变脏。当我们试图加另一个数时，我们得不到预期的和。

注意，即使我们用`@DirtiesContext`标记了第一个测试，第二个测试仍然受到影响:相加后，总和仍然是错误的。为什么？

现在，在处理方法级别`@DirtiesContext`时，Serenity 的 Spring 集成只为当前测试实例重建测试上下文。将不会重建`@Steps`中的底层依赖上下文。

为了解决这个问题，我们可以在当前的测试实例中注入`@Service`，并使服务成为`@Steps`的显式依赖:

```java
@RunWith(SerenityRunner.class)
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
@ContextConfiguration(classes = AdderService.class)
public class AdderMethodDirtiesContextDependencyWorkaroundIntegrationTest {

    private AdderConstructorDependencySteps adderSteps;

    @Autowired private AdderService adderService;

    @Before
    public void init() {
        adderSteps = new AdderConstructorDependencySteps(adderService);
    }

    //...
}
```

```java
public class AdderConstructorDependencySteps {

    private AdderService adderService;

    public AdderConstructorDependencySteps(AdderService adderService) {
        this.adderService = adderService;
    }

    // ...
}
```

或者，我们可以将条件初始化步骤放在`@Before`部分，以避免脏上下文。但是这种解决方案在一些复杂的情况下可能不可用。

```java
@RunWith(SerenityRunner.class)
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
@ContextConfiguration(classes = AdderService.class)
public class AdderMethodDirtiesContextInitWorkaroundIntegrationTest {

    @Steps private AdderServiceSteps adderServiceSteps;

    @Before
    public void init() {
        adderServiceSteps.givenBaseAndAdder(randomInt(), randomInt());
    }

    //...
}
```

### 3.2。`SpringIntegrationClassRule`

要启用类级注释，我们应该使用`SpringIntegrationClassRule`。假设我们有以下测试类:每一个都污染了环境:

```java
@RunWith(SerenityRunner.class)
@ContextConfiguration(classes = AdderService.class)
public static abstract class Base {

    @Steps AdderServiceSteps adderServiceSteps;

    @ClassRule public static SpringIntegrationClassRule springIntegrationClassRule = new SpringIntegrationClassRule();

    void whenAccumulate_thenSummedUp() {
        adderServiceSteps.whenAccumulate();
        adderServiceSteps.summedUp();
    }

    void whenAdd_thenSumWrong() {
        adderServiceSteps.whenAdd();
        adderServiceSteps.sumWrong();
    }

    void whenAdd_thenSummedUp() {
        adderServiceSteps.whenAdd();
        adderServiceSteps.summedUp();
    }
}
```

```java
@DirtiesContext(classMode = AFTER_CLASS)
public static class DirtiesContextIntegrationTest extends Base {

    @Test
    public void givenNumber_whenAdd_thenSumWrong() {
        super.whenAdd_thenSummedUp();
        adderServiceSteps.givenBaseAndAdder(randomInt(), randomInt());
        super.whenAccumulate_thenSummedUp();
        super.whenAdd_thenSumWrong();
    }
}
```

```java
@DirtiesContext(classMode = AFTER_CLASS)
public static class AnotherDirtiesContextIntegrationTest extends Base {

    @Test
    public void givenNumber_whenAdd_thenSumWrong() {
        super.whenAdd_thenSummedUp();
        adderServiceSteps.givenBaseAndAdder(randomInt(), randomInt());
        super.whenAccumulate_thenSummedUp();
        super.whenAdd_thenSumWrong();
    }
}
```

在这个例子中，将为类级别`@DirtiesContext`重建所有隐式注入。

### 3.3。`SpringIntegrationSerenityRunner`

有一个方便的类`SpringIntegrationSerenityRunner`可以自动添加上面的两个集成规则。我们可以用这个运行器运行上面的测试，以避免在我们的测试中指定方法或类测试规则:

```java
@RunWith(SpringIntegrationSerenityRunner.class)
@ContextConfiguration(locations = "classpath:adder-beans.xml")
public class AdderSpringSerenityRunnerIntegrationTest {

    @Steps private AdderSteps adderSteps;

    @Value("#{props['adder']}") private int adder;

    @Test
    public void givenNumber_whenAdd_thenSummedUp() {
        adderSteps.givenNumber();
        adderSteps.whenAdd(adder);
        adderSteps.thenSummedUp();
    }
}
```

## 4。SpringMVC 集成

如果我们只需要用 Serenity 测试 SpringMVC 组件，我们可以简单地使用[放心](/web/20220626204235/https://www.baeldung.com/rest-assured-tutorial)中的`RestAssuredMockMvc`来代替`serenity-spring`集成。

### 4.1。Maven 依赖关系

我们需要将[放心的 spring-mock-mvc](https://web.archive.org/web/20220626204235/https://search.maven.org/classic/#artifactdetails%7Cio.rest-assured%7Cspring-mock-mvc%7C3.0.3%7Cjar) 依赖添加到`pom.xml`中:

```java
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>spring-mock-mvc</artifactId>
    <version>3.0.3</version>
    <scope>test</scope>
</dependency>
```

### 4.2。`RestAssuredMockMvc`在行动中

现在让我们测试以下控制器:

```java
@RequestMapping(value = "/adder", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@RestController
public class PlainAdderController {

    private final int currentNumber = RandomUtils.nextInt();

    @GetMapping("/current")
    public int currentNum() {
        return currentNumber;
    }

    @PostMapping
    public int add(@RequestParam int num) {
        return currentNumber + num;
    }
}
```

我们可以像这样利用`RestAssuredMockMvc`的模拟 MVC 实用程序:

```java
@RunWith(SerenityRunner.class)
public class AdderMockMvcIntegrationTest {

    @Before
    public void init() {
        RestAssuredMockMvc.standaloneSetup(new PlainAdderController());
    }

    @Steps AdderRestSteps steps;

    @Test
    public void givenNumber_whenAdd_thenSummedUp() throws Exception {
        steps.givenCurrentNumber();
        steps.whenAddNumber(randomInt());
        steps.thenSummedUp();
    }
}
```

那么剩下的部分就和我们怎么用`rest-assured`没什么区别了:

```java
public class AdderRestSteps {

    private MockMvcResponse mockMvcResponse;
    private int currentNum;

    @Step("get the current number")
    public void givenCurrentNumber() throws UnsupportedEncodingException {
        currentNum = Integer.valueOf(given()
          .when()
          .get("/adder/current")
          .mvcResult()
          .getResponse()
          .getContentAsString());
    }

    @Step("adding {0}")
    public void whenAddNumber(int num) {
        mockMvcResponse = given()
          .queryParam("num", num)
          .when()
          .post("/adder");
        currentNum += num;
    }

    @Step("got the sum")
    public void thenSummedUp() {
        mockMvcResponse
          .then()
          .statusCode(200)
          .body(equalTo(currentNum + ""));
    }
}
```

## 5。Serenity、JBehave 和 Spring

Serenity 的 Spring 集成支持与 [JBehave](/web/20220626204235/https://www.baeldung.com/jbehave-rest-testing) 无缝协作。让我们将测试场景写成一个 JBehave 故事:

```java
Scenario: A user can submit a number to adder and get the sum
Given a number
When I submit another number 5 to adder
Then I get a sum of the numbers
```

我们可以在`@Service` 中实现逻辑，并通过 API 公开动作:

```java
@RequestMapping(value = "/adder", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@RestController
public class AdderController {

    private AdderService adderService;

    public AdderController(AdderService adderService) {
        this.adderService = adderService;
    }

    @GetMapping("/current")
    public int currentNum() {
        return adderService.currentBase();
    }

    @PostMapping
    public int add(@RequestParam int num) {
        return adderService.add(num);
    }
}
```

现在我们可以在`RestAssuredMockMvc`的帮助下构建 Serenity-JBehave 测试，如下所示:

```java
@ContextConfiguration(classes = { 
  AdderController.class, AdderService.class })
public class AdderIntegrationTest extends SerenityStory {

    @Autowired private AdderService adderService;

    @BeforeStory
    public void init() {
        RestAssuredMockMvc.standaloneSetup(new AdderController(adderService));
    }
}
```

```java
public class AdderStory {

    @Steps AdderRestSteps restSteps;

    @Given("a number")
    public void givenANumber() throws Exception{
        restSteps.givenCurrentNumber();
    }

    @When("I submit another number $num to adder")
    public void whenISubmitToAdderWithNumber(int num){
        restSteps.whenAddNumber(num);
    }

    @Then("I get a sum of the numbers")
    public void thenIGetTheSum(){
        restSteps.thenSummedUp();
    }
}
```

我们只能用`@ContextConfiguration`标记`SerenityStory`，然后自动启用弹簧注射。这与`@Steps`上的`@ContextConfiguration`工作原理完全相同。

## 6。总结

在本文中，我们介绍了如何将 Serenity BDD 与 Spring 集成。集成并不完美，但肯定会实现的。

和往常一样，完整的实现可以在 GitHub 项目中找到。