# 从 Spring 属性文件注入数组和列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-inject-arrays-lists>

## 1.概观

在这个快速教程中，我们将学习如何从一个 Spring 属性文件中向一个数组或`List`注入值。

## 2.默认行为

我们将从一个简单的`application.properties`文件开始:

```
arrayOfStrings=Baeldung,dot,com
```

让我们看看当我们将变量类型设置为`String[]`时 Spring 是如何表现的:

```
@Value("${arrayOfStrings}")
private String[] arrayOfStrings;
```

```
@Test
void whenContextIsInitialized_thenInjectedArrayContainsExpectedValues() {
    assertArrayEquals(new String[] {"Baeldung", "dot", "com"}, arrayOfStrings);
}
```

我们可以看到 Spring 正确地假设我们的分隔符是逗号，并相应地初始化数组。

我们还应该注意到，默认情况下，只有当我们有逗号分隔的值时，注入数组才能正常工作。

## 3.注入列表

如果我们尝试以同样的方式注入一个`List`，我们会得到一个令人惊讶的结果:

```
@Value("${arrayOfStrings}")
private List<String> unexpectedListOfStrings;
```

```
@Test
void whenContextIsInitialized_thenInjectedListContainsUnexpectedValues() {
    assertEquals(Collections.singletonList("Baeldung,dot,com"), unexpectedListOfStrings);
}
```

**我们的`List`包含一个元素，它等于我们在属性文件中设置的值。**

为了正确地注入一个`List`，我们需要使用一种叫做 [Spring 表达式语言](/web/20220628151332/https://www.baeldung.com/spring-expression-language) (SpEL)的特殊语法:

```
@Value("#{'${arrayOfStrings}'.split(',')}")
private List<String> listOfStrings;
```

```
@Test
void whenContextIsInitialized_thenInjectedListContainsExpectedValues() {
    assertEquals(Arrays.asList("Baeldung", "dot", "com"), listOfStrings);
}
```

我们可以看到，我们的表达以`#`开始，而不是我们习惯用`@Value`开始的`$`。

**我们还应该注意，我们正在调用一个`split`方法，这使得表达式比通常的注入稍微复杂一些。**

如果我们想让我们的表达式简单一点，我们可以用一种特殊的格式来声明我们的属性:

```
listOfStrings={'Baeldung','dot','com'}
```

Spring 将识别这种格式，我们将能够使用稍微简单一些的表达式来注入我们的`List`:

```
@Value("#{${listOfStrings}}")
private List<String> listOfStringsV2;
```

```
@Test
void whenContextIsInitialized_thenInjectedListV2ContainsExpectedValues() {
    assertEquals(Arrays.asList("Baeldung", "dot", "com"), listOfStringsV2);
}
```

## 4.使用自定义分隔符

让我们创建一个类似的属性，但这一次，我们将使用不同的分隔符:

```
listOfStringsWithCustomDelimiter=Baeldung;dot;com
```

**正如我们在注入`Lists`时看到的，我们可以使用一个特殊的表达式来指定我们想要的分隔符:**

```
@Value("#{'${listOfStringsWithCustomDelimiter}'.split(';')}")
private List<String> listOfStringsWithCustomDelimiter;
```

```
@Test
void whenContextIsInitialized_thenInjectedListWithCustomDelimiterContainsExpectedValues() {
    assertEquals(Arrays.asList("Baeldung", "dot", "com"), listOfStringsWithCustomDelimiter);
}
```

## 5.注射其他类型

让我们来看看以下属性:

```
listOfBooleans=false,false,true
listOfIntegers=1,2,3,4
listOfCharacters=a,b,c
```

**我们可以看到 Spring 支持开箱即用的基本类型，所以我们不需要做任何特殊的解析:**

```
@Value("#{'${listOfBooleans}'.split(',')}")
private List<Boolean> listOfBooleans;

@Value("#{'${listOfIntegers}'.split(',')}")
private List<Integer> listOfIntegers;

@Value("#{'${listOfCharacters}'.split(',')}")
private List<Character> listOfCharacters;
```

```
@Test
void whenContextIsInitialized_thenInjectedListOfBasicTypesContainsExpectedValues() {
    assertEquals(Arrays.asList(false, false, true), listOfBooleans);
    assertEquals(Arrays.asList(1, 2, 3, 4), listOfIntegers);
    assertEquals(Arrays.asList('a', 'b', 'c'), listOfCharacters);
}
```

这仅通过 SpEL 支持，所以我们不能以同样的方式注入数组。

## 6.以编程方式读取属性

为了以编程方式读取属性，我们首先需要获得我们的`Environment`对象的实例:

```
@Autowired
private Environment environment;
```

**然后，我们可以简单地使用`getProperty`方法，通过指定它的键和预期类型来读取任何属性:**

```
@Test
void whenReadingFromSpringEnvironment_thenPropertiesHaveExpectedValues() {
    String[] arrayOfStrings = environment.getProperty("arrayOfStrings", String[].class);
    List<String> listOfStrings = (List<String>)environment.getProperty("arrayOfStrings", List.class);

    assertArrayEquals(new String[] {"Baeldung", "dot", "com"}, arrayOfStrings);
    assertEquals(Arrays.asList("Baeldung", "dot", "com"), listOfStrings);
}
```

## 7.结论

在这个快速教程中，我们通过快速实用的例子学习了如何轻松地注入数组和`List`。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220628151332/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-2)