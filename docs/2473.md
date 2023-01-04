# Junit 5 中的动态测试指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit5-dynamic-tests>

## 1。概述

动态测试是 JUnit 5 中引入的一种新的编程模型。在本文中，我们将看看动态测试到底是什么以及如何创建它们。

如果你完全不熟悉 JUnit 5，你可能想查看 JUnit 5 的预览版和我们的主要指南[。](/web/20220525141341/https://www.baeldung.com/junit-5)

## 2。什么是`DynamicTest`？

用`@Test`注释标注的标准测试是静态测试，它们在编译时被完全指定。 **A `DynamicTest`是运行时**产生的测试。这些测试是由用`@TestFactory`注释标注的工厂方法生成的。

一个`@TestFactory`方法必须返回一个`Stream`、`Collection`、`Iterable`或`DynamicTest`实例的`Iterator`。返回任何其他东西都会导致一个`JUnitException`，因为无效的返回类型在编译时无法被检测到。除此之外，`@TestFactory` 方法不能是 stati `c`或`private`。

`DynamicTest`的执行方式与标准`@Test`不同，并且不支持生命周期回调。也就是说， **`@BeforeEach`和`@AfterEach`方法不会被`DynamicTest`的**调用。

## 3。创造`DynamicTests`

首先，让我们看看创建`DynamicTest`的不同方法。

这里的例子本质上不是动态的，但是它们将为创建真正动态的例子提供一个良好的起点。

我们将创建一个`DynamicTest`的`Collection`:

```
@TestFactory
Collection<DynamicTest> dynamicTestsWithCollection() {
    return Arrays.asList(
      DynamicTest.dynamicTest("Add test",
        () -> assertEquals(2, Math.addExact(1, 1))),
      DynamicTest.dynamicTest("Multiply Test",
        () -> assertEquals(4, Math.multiplyExact(2, 2))));
}
```

`@TestFactory`方法告诉 JUnit 这是一个创建动态测试的工厂。如我们所见，我们只返回了`DynamicTest`的一个`Collection`。**每个`DynamicTest`由两部分组成，测试名称或显示名称，以及一个`Executable`** 。

输出将包含我们传递给动态测试的显示名称:

```
Add test(dynamicTestsWithCollection())
Multiply Test(dynamicTestsWithCollection())
```

相同的测试可以修改为返回一个`Iterable`、`Iterator`或`Stream`:

```
@TestFactory
Iterable<DynamicTest> dynamicTestsWithIterable() {
    return Arrays.asList(
      DynamicTest.dynamicTest("Add test",
        () -> assertEquals(2, Math.addExact(1, 1))),
      DynamicTest.dynamicTest("Multiply Test",
        () -> assertEquals(4, Math.multiplyExact(2, 2))));
}

@TestFactory
Iterator<DynamicTest> dynamicTestsWithIterator() {
    return Arrays.asList(
      DynamicTest.dynamicTest("Add test",
        () -> assertEquals(2, Math.addExact(1, 1))),
      DynamicTest.dynamicTest("Multiply Test",
        () -> assertEquals(4, Math.multiplyExact(2, 2))))
        .iterator();
}

@TestFactory
Stream<DynamicTest> dynamicTestsFromIntStream() {
    return IntStream.iterate(0, n -> n + 2).limit(10)
      .mapToObj(n -> DynamicTest.dynamicTest("test" + n,
        () -> assertTrue(n % 2 == 0)));
}
```

请注意，如果`@TestFactory`返回一个`Stream`，那么一旦所有测试执行完毕，它将自动关闭。

输出将与第一个例子非常相似。它将包含我们传递给动态测试的显示名称。

## 4。创建一个`DynamicTests`的`Stream`或

出于演示的目的，考虑一个`DomainNameResolver`，当我们将域名作为输入传递时，它返回一个 IP 地址。

为了简单起见，让我们看一下我们工厂方法的高层框架:

```
@TestFactory
Stream<DynamicTest> dynamicTestsFromStream() {

    // sample input and output
    List<String> inputList = Arrays.asList(
      "www.somedomain.com", "www.anotherdomain.com", "www.yetanotherdomain.com");
    List<String> outputList = Arrays.asList(
      "154.174.10.56", "211.152.104.132", "178.144.120.156");

    // input generator that generates inputs using inputList
    /*...code here...*/

    // a display name generator that creates a 
    // different name based on the input
    /*...code here...*/

    // the test executor, which actually has the 
    // logic to execute the test case
    /*...code here...*/

    // combine everything and return a Stream of DynamicTest
    /*...code here...*/
}
```

除了我们已经熟悉的`@TestFactory`注释，这里没有太多与`DynamicTest`相关的代码。

两个`ArrayList`将分别用作`DomainNameResolver`的输入和预期输出。

现在让我们来看看输入生成器:

```
Iterator<String> inputGenerator = inputList.iterator();
```

输入生成器只不过是`String`的一个`Iterator`。它使用我们的`inputList`，并逐个返回域名。

显示名称生成器相当简单:

```
Function<String, String> displayNameGenerator 
  = (input) -> "Resolving: " + input;
```

显示名称生成器的任务只是为测试用例提供一个显示名称，这个测试用例将在 JUnit 报告或者我们 ide 的 JUnit 选项卡中使用。

这里我们只是利用域名为每个测试生成唯一的名称。不要求创建唯一的名称，但这将有助于防止任何失败。有了这个，我们就能够说出测试用例失败的域名。

**现在让我们看看测试的核心部分——测试执行代码:**

```
DomainNameResolver resolver = new DomainNameResolver();
ThrowingConsumer<String> testExecutor = (input) -> {
    int id = inputList.indexOf(input);

    assertEquals(outputList.get(id), resolver.resolveDomain(input));
};
```

我们已经使用了`ThrowingConsumer`，它是一个用于编写测试用例的`@FunctionalInterface`。对于数据生成器生成的每个输入，我们从`outputList`获取预期输出，从`DomainNameResolver`的实例获取实际输出。

现在，最后一部分只是简单地组装所有的部件，并作为`DynamicTest`的`Stream`返回:

```
return DynamicTest.stream(
  inputGenerator, displayNameGenerator, testExecutor);
```

就是这样。运行测试将显示包含由我们的显示名称生成器定义的名称的报告:

```
Resolving: www.somedomain.com(dynamicTestsFromStream())
Resolving: www.anotherdomain.com(dynamicTestsFromStream())
Resolving: www.yetanotherdomain.com(dynamicTestsFromStream())
```

## 5。使用 Java 8 改进`DynamicTest`功能

通过使用 Java 8 的特性，可以极大地改进前一节中编写的测试工厂。生成的代码将更加简洁，并且可以用更少的行来编写:

```
@TestFactory
Stream<DynamicTest> dynamicTestsFromStreamInJava8() {

    DomainNameResolver resolver = new DomainNameResolver();

    List<String> domainNames = Arrays.asList(
      "www.somedomain.com", "www.anotherdomain.com", "www.yetanotherdomain.com");
    List<String> outputList = Arrays.asList(
      "154.174.10.56", "211.152.104.132", "178.144.120.156");

    return inputList.stream()
      .map(dom -> DynamicTest.dynamicTest("Resolving: " + dom, 
        () -> {int id = inputList.indexOf(dom);

      assertEquals(outputList.get(id), resolver.resolveDomain(dom));
    }));       
}
```

上面的代码与我们在上一节中看到的代码具有相同的效果。`inputList.stream().map()`提供输入流(输入发生器)。`dynamicTest()`的第一个参数是我们的显示名称生成器(" Resolving: " + `dom`)，而第二个参数是`lambda`，是我们的测试执行器。

输出将与上一节中的输出相同。

## 6。附加示例

在这个例子中，我们进一步探索动态测试的能力，根据测试用例过滤输入:

```
@TestFactory
Stream<DynamicTest> dynamicTestsForEmployeeWorkflows() {
    List<Employee> inputList = Arrays.asList(
      new Employee(1, "Fred"), new Employee(2), new Employee(3, "John"));

    EmployeeDao dao = new EmployeeDao();
    Stream<DynamicTest> saveEmployeeStream = inputList.stream()
      .map(emp -> DynamicTest.dynamicTest(
        "saveEmployee: " + emp.toString(), 
          () -> {
              Employee returned = dao.save(emp.getId());
              assertEquals(returned.getId(), emp.getId());
          }
    ));

    Stream<DynamicTest> saveEmployeeWithFirstNameStream 
      = inputList.stream()
      .filter(emp -> !emp.getFirstName().isEmpty())
      .map(emp -> DynamicTest.dynamicTest(
        "saveEmployeeWithName" + emp.toString(), 
        () -> {
            Employee returned = dao.save(emp.getId(), emp.getFirstName());
            assertEquals(returned.getId(), emp.getId());
            assertEquals(returned.getFirstName(), emp.getFirstName());
        }));

    return Stream.concat(saveEmployeeStream, 
      saveEmployeeWithFirstNameStream);
}
```

`save(Long)`方法只需要`employeeId`。因此，它利用了所有的`Employee`实例。`save(Long, String)`方法除了`employeeId`还需要`firstName`。因此，它过滤掉没有`firstName.`的`Employee`实例

**最后，我们合并两个流，并将所有测试作为一个单独的`Stream`返回。**

现在，让我们来看看输出:

```
saveEmployee: Employee 
  [id=1, firstName=Fred](dynamicTestsForEmployeeWorkflows())
saveEmployee: Employee 
  [id=2, firstName=](dynamicTestsForEmployeeWorkflows())
saveEmployee: Employee 
  [id=3, firstName=John](dynamicTestsForEmployeeWorkflows())
saveEmployeeWithNameEmployee 
  [id=1, firstName=Fred](dynamicTestsForEmployeeWorkflows())
saveEmployeeWithNameEmployee 
  [id=3, firstName=John](dynamicTestsForEmployeeWorkflows())
```

## 7。结论

参数化测试可以取代本文中的许多例子。然而，动态测试不同于参数化测试，因为它们不支持完整的测试生命周期，而参数化测试支持。

此外，动态测试在如何生成输入和如何执行测试方面提供了更多的灵活性。

JUnit 5 更喜欢[扩展而不是特性](https://web.archive.org/web/20220525141341/https://github.com/junit-team/junit5/wiki/Core-Principles)原则。因此，动态测试的主要目的是为第三方框架或扩展提供一个扩展点。

你可以在我们关于 JUnit 5 中重复测试的[文章中读到更多关于 JUnit 5 的其他特性。](/web/20220525141341/https://www.baeldung.com/junit-5-repeated-test)

别忘了在 GitHub 上查看这篇[文章的完整源代码。](https://web.archive.org/web/20220525141341/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5)