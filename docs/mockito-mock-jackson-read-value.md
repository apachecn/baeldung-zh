# 模仿 ObjectMapper readValue()方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-mock-jackson-read-value>

## 1.概观

当使用 Jackson 对 JSON 进行反序列化的代码进行单元测试时，我们可能会发现模仿`ObjectMapper#readValue `方法更容易。通过这样做，我们不需要在测试中指定长的 JSON 输入。

在本教程中，我们将看到如何使用 [Mockito](/web/20220630014556/https://www.baeldung.com/mockito-series) 来实现这一点。

## 2.Maven 依赖性

首先，作为 Maven 依赖项，我们将使用`[mockito-core](https://web.archive.org/web/20220630014556/https://search.maven.org/artifact/org.mockito/mockito-core)`和`[jackson-databind](https://web.archive.org/web/20220630014556/https://search.maven.org/artifact/com.fasterxml.jackson.core/jackson-databind)`:

```
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>3.3.3</version>
    <scope>test</scope>
 </dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
    <type>bundle</type>
</dependency>
```

## 3.一个例子

让我们考虑一个简单的`Flower`类:

```
public class Flower {

    private String name;
    private Integer petals;

    public Flower(String name, Integer petals) {
        this.name = name;
        this.petals = petals;
    }

    // default constructor, getters and setters
}
```

假设我们有一个类来验证一个`Flower`对象的 JSON 字符串表示。它将`ObjectMapper `作为一个构造函数参数——这使得我们以后可以很容易地模仿它:

```
public class FlowerJsonStringValidator {
    private ObjectMapper objectMapper;

    public FlowerJsonStringValidator(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    public boolean flowerHasPetals(String jsonFlowerAsString) throws JsonProcessingException {
        Flower flower = objectMapper.readValue(jsonFlowerAsString, Flower.class);
        return flower.getPetals() > 0;
    }
}
```

接下来，我们将使用`Mockito `为验证器逻辑编写单元测试。

## 4.用 Mockito 进行单元测试

让我们从设置测试类开始。我们可以很容易地模仿一个`ObjectMapper `并将它作为构造函数参数传递给我们的`FlowerStringValidator`类:

```
@ExtendWith(MockitoExtension.class)
public class FlowerJsonStringValidatorUnitTest {

    @Mock
    private ObjectMapper objectMapper;

    private FlowerJsonStringValidator flowerJsonStringValidator;

    @BeforeEach
    public void setUp() {
        flowerJsonStringValidator = new FlowerJsonStringValidator(objectMapper);
    }

    ...
}
```

注意，我们在测试中使用了 [JUnit 5](/web/20220630014556/https://www.baeldung.com/junit-5) ，所以我们用`@ExtendWith(MockitoExtension.class)`注释了我们的测试类。

现在我们已经准备好了模拟测试，让我们编写一个简单的测试:

```
@Test
public void whenCallingHasPetalsWithPetals_thenReturnsTrue() throws JsonProcessingException {
    Flower rose = new Flower("testFlower", 100);

    when(objectMapper.readValue(anyString(), eq(Flower.class))).thenReturn(rose);

    assertTrue(flowerJsonStringValidator.flowerHasPetals("this can be a very long json flower"));

    verify(objectMapper, times(1)).readValue(anyString(), eq(Flower.class));
}
```

**因为我们在这里模仿`ObjectMapper `，所以我们可以忽略它的输入，关注它的输出**，然后它被传递给实际的验证器逻辑。正如我们所看到的，我们不需要指定有效的 JSON 输入，这在现实场景中可能会很长很难。

## 5.结论

在本文中，我们看到了如何模仿`ObjectMapper` 来提供有效的测试用例。最后，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220630014556/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito-2)