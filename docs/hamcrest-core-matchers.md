# Hamcrest 普通核心匹配器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hamcrest-core-matchers>

## 1.概观

在这个快速教程中，我们将探索来自流行的 [Hamcrest](https://web.archive.org/web/20220601145715/http://hamcrest.org/) 框架的`CoreMatchers `类，以编写简单且更具表现力的测试用例。

这个想法是让断言语句读起来像自然语言。

## 2.Hamcrest 设置

我们可以通过将以下依赖项添加到我们的`pom.xml`文件来使用 Hamcrest 和 Maven:

```java
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>java-hamcrest</artifactId>
    <version>2.0.0.0</version>
    <scope>test</scope>
</dependency>
```

这个库的最新版本总是可以在这里找到。

## 3.普通核心匹配器

### 3.1。`is(T)`和`is(Matcher<T>)`

`is(T)` 使用一个对象作为参数来检查等式，而`is(Matcher<T>)`使用另一个匹配器来让等式语句更有表现力。

**我们可以将它用于几乎所有的方法**:

```java
String testString = "hamcrest core";

assertThat(testString, is("hamcrest core"));
assertThat(testString, is(equalTo("hamcrest core")));
```

### 3.2。`equalTo(T)`

`equalTo(T)`将一个对象作为参数，并检查它与另一个对象是否相等。**经常与`is(Matcher<T>)` :** 一起使用

```java
String actualString = "equalTo match";
List<String> actualList = Lists.newArrayList("equalTo", "match");

assertThat(actualString, is(equalTo("equalTo match")));
assertThat(actualList, is(equalTo(Lists.newArrayList("equalTo", "match"))));
```

我们也可以使用`equalToObject(Object operand) – `来检查相等性，并且不强制两个对象应该是相同的静态类型:

```java
Object original = 100;
assertThat(original, equalToObject(100));
```

### 3.3。`not(T)`和`not(Matcher<T>)`

`not(T)`和`not(Matcher<T>)`用于检查给定对象的不相等性。首先接受一个对象作为参数，然后接受另一个匹配器:

```java
String testString = "troy kingdom";

assertThat(testString, not("german kingdom"));
assertThat(testString, is(not(equalTo("german kingdom"))));
assertThat(testString, is(not(instanceOf(Integer.class))));
```

### 3.4。`nullValue()`和`nullValue(Class<T>)`

`nullValue() `检查被检查对象的`null `值。`nullValue(Class<T>) `检查给定类类型对象的可空性:

```java
Integer nullObject = null;

assertThat(nullObject, is(nullValue()));
assertThat(nullObject, is(nullValue(Integer.class)));
```

### 3.5。`notNullValue()`和`notNullValue(Class<T>)`

**这些都是常用的`is(not(nullValue))`的快捷方式。**这些检查一个对象或类类型的非空相等性:

```java
Integer testNumber = 123;

assertThat(testNumber, is(notNullValue()));
assertThat(testNumber, is(notNullValue(Integer.class)));
```

### 3.6。`instanceOf(Class<?>)`

如果被检查的对象是指定的`Class`类型的实例，则`instanceOf(Class<?>)` 匹配。

**验证，该方法内部调用** `**Class **`类的 `**isIntance(Object) **` **:**

```java
assertThat("instanceOf example", is(instanceOf(String.class)));
```

### 3.7。 `isA**(Class<T> type)**`

`isA(Class<T> type)`是上述`instanceOf(Class<?>)`的快捷方式。**它采用与** `**instanceOf(Class<?>)**`完全相同的论证类型:

```java
assertThat("Drogon is biggest dragon", isA(String.class));
```

### 3.8。`sameInstance()`

如果两个引用变量指向堆中的同一个对象，那么`sameInstance()`匹配:

```java
String string1 = "Viseron";
String string2 = string1;

assertThat(string1, is(sameInstance(string2)));
```

### 3.9。`any(Class<T>)`

`any(Class<T>)`检查该类是否与实际对象的类型相同:

```java
assertThat("test string", is(any(String.class)));
assertThat("test string", is(any(Object.class)));
```

### 3.10。`allOf(Matcher<? extends T>…) and anyOf(Matcher<? extends T>…)`

我们可以使用`allOf(Matcher<? extends T>…)`来断言实际对象是否符合所有指定的条件:

```java
String testString = "Achilles is powerful";
assertThat(testString, allOf(startsWith("Achi"), endsWith("ul"), containsString("Achilles")));
```

`anyOf(Matcher<? extends T>…)`的行为类似于`allOf(Matcher<? extends T>… )`,但是如果被检查的对象匹配任何指定的条件，则匹配:

```java
String testString = "Hector killed Achilles";
assertThat(testString, anyOf(startsWith("Hec"), containsString("baeldung")));
```

### 3.11。`hasItem(T)`和`hasItem(Matcher<? extends T>)`

如果被检查的`Iterable `集合与`hasItem()` 或`hasItem(Matcher<? extends T>)`中的给定对象或匹配器匹配，则这些匹配。

让我们理解这是如何工作的:

```java
List<String> list = Lists.newArrayList("java", "spring", "baeldung");

assertThat(list, hasItem("java"));
assertThat(list, hasItem(isA(String.class)));
```

**同样，** **我们也可以用** `**hasItems(T…)**` **和** `**hasItems(Matcher<? extends T>…)**`对多个项目进行断言:

```java
List<String> list = Lists.newArrayList("java", "spring", "baeldung");

assertThat(list, hasItems("java", "baeldung"));
assertThat(list, hasItems(isA(String.class), endsWith("ing")));
```

### 3.12。`both(Matcher<? extends T>) and either(Matcher<? extends T>)`

顾名思义，当指定的两个条件都匹配被检查对象时，`both(Matcher<? extends T>) `匹配:

```java
String testString = "daenerys targaryen";
assertThat(testString, both(startsWith("daene")).and(containsString("yen")));
```

当指定条件中的任一个与检查对象匹配时,`either(Matcher<? extends T>)`匹配:

```java
String testString = "daenerys targaryen";
assertThat(testString, either(startsWith("tar")).or(containsString("targaryen")));
```

## 4。 `String` **对比**

我们可以使用`containsString(String)`或`containsStringIgnoringCase(String) `来断言实际字符串是否包含测试字符串:

```java
String testString = "Rhaegar Targaryen";
assertThat(testString, containsString("aegar"));
assertThat(testString, containsStringIgnoringCase("AEGAR"));
```

或`startsWith(String)` 和`startsWithIgnoringCase(String) `来断言实际字符串是否以测试字符串开始:

```java
assertThat(testString, startsWith("Rhae"));
assertThat(testString, startsWithIgnoringCase("rhae"));
```

我们还可以使用`endsWith(String) `或`endsWithIgnoringCase(String)`来断言实际字符串是否以测试字符串结尾:

```java
assertThat(testString, endsWith("aryen"));
assertThat(testString, endsWithIgnoringCase("ARYEN"));
```

## 5。结论

在本文中，我们讨论了`Hamcrest `库中`CoreMatchers `类的不同方法。

和往常一样，例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220601145715/https://github.com/eugenp/tutorials/tree/master/testing-modules/hamcrest)