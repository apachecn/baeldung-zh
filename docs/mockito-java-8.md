# Mockito 的 Java 8 特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-java-8>

## 1。概述

Java 8 引入了一系列令人惊叹的新特性，比如 lambda 和 streams。自然地，Mockito 在其第二个主要版本中利用了这些最近的创新。

在本文中，我们将探索这一强大组合所提供的一切。

## 2。用默认方法模仿接口

从 Java 8 开始，我们现在可以在接口中编写方法实现。这可能是一个很棒的新功能，但是它对语言的引入违背了一个强有力的概念，这个概念从一开始就是 Java 的一部分。

Mockito 版本 1 没有为这种变化做好准备。基本上，因为它不允许我们要求它从接口调用真正的方法。

假设我们有一个包含两个方法声明的接口:第一个是我们都习惯的老式方法签名，另一个是全新的`default`方法:

```java
public interface JobService {

    Optional<JobPosition> findCurrentJobPosition(Person person);

    default boolean assignJobPosition(Person person, JobPosition jobPosition) {
        if(!findCurrentJobPosition(person).isPresent()) {
            person.setCurrentJobPosition(jobPosition);

            return true;
        } else {
            return false;
        }
    }
}
```

注意， *assignJobPosition()* `default`方法调用了未实现的`findCurrentJobPosition()` 方法。

现在，假设我们想要测试我们的 *assignJobPosition()* 的实现，而不编写`findCurrentJobPosition()`的实际实现。我们可以简单地创建一个`JobService,` 的模拟版本，然后告诉 Mockito 从对我们未实现的方法的调用中返回一个已知值，并在调用`assignJobPosition()`时调用真正的方法:

```java
public class JobServiceUnitTest {

    @Mock
    private JobService jobService;

    @Test
    public void givenDefaultMethod_whenCallRealMethod_thenNoExceptionIsRaised() {
        Person person = new Person();

        when(jobService.findCurrentJobPosition(person))
              .thenReturn(Optional.of(new JobPosition()));

        doCallRealMethod().when(jobService)
          .assignJobPosition(
            Mockito.any(Person.class), 
            Mockito.any(JobPosition.class)
        );

        assertFalse(jobService.assignJobPosition(person, new JobPosition()));
    }
}
```

这是完全合理的，如果我们使用抽象类而不是接口，它会工作得很好。

然而，Mockito version 1 的内部工作方式并不适合这种结构。如果我们在 Mockito pre 版本 2 上运行这段代码，我们会得到这个被很好描述的错误:

```java
org.mockito.exceptions.base.MockitoException:
Cannot call a real method on java interface. The interface does not have any implementation!
Calling real methods is only possible when mocking concrete classes.
```

Mockito 正在做它的工作，告诉我们它不能在接口上调用真正的方法，因为这个操作在 Java 8 之前是不可想象的。

好消息是，只要改变我们正在使用的 Mockito 版本，我们就可以消除这个错误。例如，使用 Maven，我们可以使用版本 2.7.5(最新的 Mockito 版本可以在这里找到):

```java
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.7.5</version>
    <scope>test</scope>
</dependency>
```

没有必要对代码进行任何修改。下次我们运行测试时，错误将不再发生。

## 3。返回`Optional` 和`Stream`T3 的默认值

`[Optional](https://web.archive.org/web/20230103154038/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html)`和`[Stream](https://web.archive.org/web/20230103154038/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html)`是其他 Java 8 新增加的。这两个类之间的一个相似之处是，它们都有一个表示空对象的特殊类型的值。这个空的物体使得它更容易避开迄今为止无所不在的`[NullPointerException](https://web.archive.org/web/20230103154038/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/NullPointerException.html).`

### 3.1。例子有`Optional`

考虑一个注入前一节中描述的`JobService`的服务，它有一个调用`JobService#findCurrentJobPosition()`的方法:

```java
public class UnemploymentServiceImpl implements UnemploymentService {

    private JobService jobService;

    public UnemploymentServiceImpl(JobService jobService) {
        this.jobService = jobService;
    }

    @Override
    public boolean personIsEntitledToUnemploymentSupport(Person person) {
        Optional<JobPosition> optional = jobService.findCurrentJobPosition(person);

        return !optional.isPresent();
    }
}
```

现在，假设我们想要创建一个测试来检查，当一个人没有当前的工作职位时，他们是否有权获得失业支持。

在这种情况下，我们将强制`findCurrentJobPosition()` 返回一个空的`Optional`。**在 Mockito 版本 2** 之前，我们需要模拟对该方法的调用:

```java
public class UnemploymentServiceImplUnitTest {

    @Mock
    private JobService jobService;

    @InjectMocks
    private UnemploymentServiceImpl unemploymentService;

    @Test
    public void givenReturnIsOfTypeOptional_whenMocked_thenValueIsEmpty() {
        Person person = new Person();

        when(jobService.findCurrentJobPosition(any(Person.class)))
          .thenReturn(Optional.empty());

        assertTrue(unemploymentService.personIsEntitledToUnemploymentSupport(person));
    }
}
```

第 13 行上的这个`when(…).thenReturn(…)`指令是必要的，因为 Mockito 对被模仿对象的任何方法调用的默认返回值是`null`。版本 2 改变了这种行为。

因为我们在处理`Optional,` **时很少处理空值，所以默认情况下，Mockito 现在返回一个空的`Optional`**。这与调用 [`Optional.empty()`](https://web.archive.org/web/20230103154038/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#empty()) 的返回值完全相同。

因此，**当使用 Mockito version 2** 时，我们可以去掉第 13 行，我们的测试仍然会成功:

```java
public class UnemploymentServiceImplUnitTest {

    @Test
    public void givenReturnIsOptional_whenDefaultValueIsReturned_thenValueIsEmpty() {
        Person person = new Person();

        assertTrue(unemploymentService.personIsEntitledToUnemploymentSupport(person));
    }
}
```

### 3.2。例子有`Stream`

当我们模仿一个返回`Stream`的方法时，也会发生同样的行为。

让我们向我们的`JobService`接口添加一个新方法，该方法返回一个代表一个人曾经工作过的所有职位的流:

```java
public interface JobService {
    Stream<JobPosition> listJobs(Person person);
}
```

此方法用于另一个新方法，该方法将查询一个人是否曾经从事过与给定搜索字符串相匹配的工作:

```java
public class UnemploymentServiceImpl implements UnemploymentService {

    @Override
    public Optional<JobPosition> searchJob(Person person, String searchString) {
        return jobService.listJobs(person)
          .filter((j) -> j.getTitle().contains(searchString))
          .findFirst();
    }
}
```

因此，假设我们想要正确地测试`searchJob(),` 的实现，而不必担心编写`listJobs()`并假设我们想要测试这个人还没有从事任何工作的场景。在这种情况下，我们希望`listJobs()`返回一个空的`Stream`。

**在 Mockito 版本 2 之前，我们需要模拟调用`listJobs()`** 来编写这样的测试:

```java
public class UnemploymentServiceImplUnitTest {

    @Test
    public void givenReturnIsOfTypeStream_whenMocked_thenValueIsEmpty() {
        Person person = new Person();
        when(jobService.listJobs(any(Person.class))).thenReturn(Stream.empty());

        assertFalse(unemploymentService.searchJob(person, "").isPresent());
    }
}
```

**如果我们升级到版本 2** ，我们可以丢弃`when(…).thenReturn(…)`调用，因为现在 **Mockito 将在默认情况下返回一个空的`Stream`给被模仿的方法**:

```java
public class UnemploymentServiceImplUnitTest {

    @Test
    public void givenReturnIsStream_whenDefaultValueIsReturned_thenValueIsEmpty() {
        Person person = new Person();

        assertFalse(unemploymentService.searchJob(person, "").isPresent());
    }
}
```

## 4。利用λ表达式

有了 Java 8 的 lambda 表达式，我们可以让语句更加简洁易读。当使用 Mockito 时，lambda 表达式带来的简单性的两个很好的例子是`[ArgumentMatchers](https://web.archive.org/web/20230103154038/https://static.javadoc.io/org.mockito/mockito-core/2.7.10/org/mockito/ArgumentMatcher.html)`和 custom `[Answers](https://web.archive.org/web/20230103154038/https://static.javadoc.io/org.mockito/mockito-core/2.7.10/org/mockito/stubbing/Answer.html)`。

### 4.1。λ和`ArgumentMatcher`的组合

在 Java 8 之前，我们需要创建一个实现`ArgumentMatcher`的类，并在`matches()`方法中编写我们的自定义规则。

在 Java 8 中，我们可以用一个简单的 lambda 表达式替换内部类:

```java
public class ArgumentMatcherWithLambdaUnitTest {

    @Test
    public void whenPersonWithJob_thenIsNotEntitled() {
        Person peter = new Person("Peter");
        Person linda = new Person("Linda");

        JobPosition teacher = new JobPosition("Teacher");

        when(jobService.findCurrentJobPosition(
          ArgumentMatchers.argThat(p -> p.getName().equals("Peter"))))
          .thenReturn(Optional.of(teacher));

        assertTrue(unemploymentService.personIsEntitledToUnemploymentSupport(linda));
        assertFalse(unemploymentService.personIsEntitledToUnemploymentSupport(peter));
    }
}
```

### 4.2。`Answer`λ和自定义的组合

将 lambda 表达式与 Mockito 的`Answer`结合使用也能达到同样的效果。

例如，如果我们想要模拟对`listJobs()`方法的调用，以便使它返回一个包含单个`JobPosition`的`Stream`(如果`Person`的名称是“彼得”),否则返回一个空的`Stream`,我们就必须创建一个实现`Answer`接口的类(匿名的或内部的)。

同样，lambda 表达式的使用，允许我们以内联方式编写所有模拟行为:

```java
public class CustomAnswerWithLambdaUnitTest {

    @Before
    public void init() {
        when(jobService.listJobs(any(Person.class))).then((i) ->
          Stream.of(new JobPosition("Teacher"))
          .filter(p -> ((Person) i.getArgument(0)).getName().equals("Peter")));
    }
}
```

注意，在上面的实现中，不需要`PersonAnswer`内部类。

## 5。结论

在本文中，我们介绍了如何利用 Java 8 和 Mockito version 2 的新特性来编写更干净、更简单、更短的代码。如果您不熟悉我们在这里看到的一些 Java 8 特性，请查看我们的一些文章:

*   [Lambda 表达式和函数接口:技巧和最佳实践](/web/20230103154038/https://www.baeldung.com/java-8-lambda-expressions-tips)
*   [Java 8 中的新特性](/web/20230103154038/https://www.baeldung.com/java-8-new-features)
*   [Java 8 可选指南](/web/20230103154038/https://www.baeldung.com/java-optional)
*   [Java 8 流简介](/web/20230103154038/https://www.baeldung.com/java-8-streams-introduction)

此外，检查我们的 [GitHub 库](https://web.archive.org/web/20230103154038/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito)上附带的代码。