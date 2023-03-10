# 用谷歌真相测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/google-truth>

## 1。概述

`[Truth](https://web.archive.org/web/20221128115819/https://google.github.io/truth/)`是一个**流畅灵活的开源测试框架，旨在使测试断言和失败消息更具可读性。**

在本文中，我们将探索`Truth`框架的关键特性，并实现示例来展示它的能力。

## 2。Maven 依赖关系

首先，我们需要将`truth`和`truth-java8-extension`添加到我们的`pom.xml:`中

```java
<dependency>
    <groupId>com.google.truth</groupId>
    <artifactId>truth</artifactId>
    <version>0.32</version>
</dependency>
<dependency>
    <groupId>com.google.truth.extensions</groupId>
    <artifactId>truth-java8-extension</artifactId>
    <version>0.32</version>
    <scope>test</scope>
</dependency>
```

你可以在 Maven Central 上找到最新版本的 [truth](https://web.archive.org/web/20221128115819/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.truth%22%20AND%20a%3A%22truth%22) 和 [truth-java8-extension](https://web.archive.org/web/20221128115819/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.truth.extensions%22%20AND%20a%3A%22truth-java8-extension%22) 。

## 3。简介

允许我们为各种类编写可读的断言和失败消息:

*   **标准 Java**——原语、数组、字符串、对象、集合、可抛出对象、类等。
*   **Java 8**–`Optional`和`Stream`实例
*   **番石榴**—`Optional`、`Multimap`、`Multiset`、`Table`物件
*   **自定义类型**——通过扩展`Subject` 类，我们将在后面看到

通过`Truth`和`Truth8`类，该库提供了用于编写断言的实用方法，这些断言在`subject`上工作，即测试中的值或对象。

一旦主题是已知的， **`Truth`可以在编译时推理关于主题**的什么命题是已知的。这允许它返回我们的值周围的包装器，这些包装器声明特定于特定主题的命题方法。

例如，当断言一个列表时，`Truth`返回一个定义了像`contains()` 和`containsAnyOf()`等方法的`IterableSubject`实例。当断言一个`Map`时，它返回一个`MapSubject`，声明像`containsEntry()`和`containsKey()`这样的方法。

## 4。入门

为了开始编写断言，让我们首先导入`Truth`的入口点:

```java
import static com.google.common.truth.Truth.*;
import static com.google.common.truth.Truth8.*;
```

现在，让我们编写一个简单的类，我们将在下面的几个示例中使用它:

```java
public class User {
    private String name = "John Doe";
    private List<String> emails
      = Arrays.asList("[[email protected]](/web/20221128115819/https://www.baeldung.com/cdn-cgi/l/email-protection)", "[[email protected]](/web/20221128115819/https://www.baeldung.com/cdn-cgi/l/email-protection)");

    public boolean equals(Object obj) {
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        User other = (User) obj;
        return Objects.equals(this.name, other.name);
    }

    // standard constructors, getters and setters
}
```

注意自定义的`equals()`方法，其中我们声明两个`User`对象相等，如果它们的名称是。

## 5。标准 Java 断言

在这一节中，我们将看到如何为标准 Java 类型编写测试断言的详细例子。

### 5.1。`Object`断言

`Truth`提供了用于在对象上执行断言的`Subject`包装器。`Subject`也是库中所有其他包装器的父类，它声明了确定`Object`(在我们的例子中是`User`)是否等于另一个对象的方法:

```java
@Test
public void whenComparingUsers_thenEqual() {
    User aUser = new User("John Doe");
    User anotherUser = new User("John Doe");

    assertThat(aUser).isEqualTo(anotherUser);
}
```

或者如果它等于列表中的给定对象:

```java
@Test
public void whenComparingUser_thenInList() {
    User aUser = new User();

    assertThat(aUser).isIn(Arrays.asList(1, 3, aUser, null));
}
```

或者如果不是:

```java
@Test
public void whenComparingUser_thenNotInList() {
    // ...

    assertThat(aUser).isNotIn(Arrays.asList(1, 3, "Three"));
}
```

如果为空或不为空:

```java
@Test
public void whenComparingUser_thenIsNull() {
    User aUser = null;

    assertThat(aUser).isNull();
}

@Test
public void whenComparingUser_thenNotNull() {
    User aUser = new User();

    assertThat(aUser).isNotNull();
}
```

或者如果它是特定类的实例:

```java
@Test
public void whenComparingUser_thenInstanceOf() {
    // ...

    assertThat(aUser).isInstanceOf(User.class);
}
```

在`Subject`类中还有其他断言方法。要发现它们，请参考 [`Subject`文档](https://web.archive.org/web/20221128115819/https://google.github.io/truth/api/latest/com/google/common/truth/Subject.html)。

在接下来的章节中，**我们将关注每种特定类型** `Truth`支架的最相关方法。但是，请记住，也可以应用`Subject`类中的所有方法。

### 5.2。`Integer`、`Float,`和`Double`断言

可以比较`Integer`、`Float,`和`Double`实例的相等性:

```java
@Test
public void whenComparingInteger_thenEqual() {
    int anInt = 10;

    assertThat(anInt).isEqualTo(10);
}
```

如果它们更大:

```java
@Test
public void whenComparingFloat_thenIsBigger() {
    float aFloat = 10.0f;

    assertThat(aFloat).isGreaterThan(1.0f);
}
```

或更小:

```java
@Test
public void whenComparingDouble_thenIsSmaller() {
    double aDouble = 10.0f;

    assertThat(aDouble).isLessThan(20.0);
}
```

此外，还可以检查`, Float`和`Double`实例，看它们是否在预期精度内:

```java
@Test
public void whenComparingDouble_thenWithinPrecision() {
    double aDouble = 22.18;

    assertThat(aDouble).isWithin(2).of(23d);
}

@Test
public void whenComparingFloat_thenNotWithinPrecision() {
    float aFloat = 23.04f;

    assertThat(aFloat).isNotWithin(1.3f).of(100f);
}
```

### 5.3。`BigDecimal`断言

除了常见的断言，这种类型可以忽略其规模进行比较:

```java
@Test
public void whenComparingBigDecimal_thenEqualIgnoringScale() {
    BigDecimal aBigDecimal = BigDecimal.valueOf(1000, 3);

    assertThat(aBigDecimal).isEqualToIgnoringScale(new BigDecimal(1.0));
}
```

### 5.4。`Boolean`断言

只提供了两种相关的方法，`isTrue()`和`isFalse()`:

```java
@Test
public void whenCheckingBoolean_thenTrue() {
    boolean aBoolean = true;

    assertThat(aBoolean).isTrue();
}
```

### 5.5。`String`断言

我们可以测试`String`是否以特定的文本开始:

```java
@Test
public void whenCheckingString_thenStartsWith() {
    String aString = "This is a string";

    assertThat(aString).startsWith("This");
}
```

此外，我们可以检查字符串是否包含给定的字符串，它是否以期望值结尾，或者它是否为空。源代码中提供了这些和其他方法的测试用例。

### 5.6。数组断言

我们可以检查`Array` s，看看它们是否等于其他数组:

```java
@Test
public void whenComparingArrays_thenEqual() {
    String[] firstArrayOfStrings = { "one", "two", "three" };
    String[] secondArrayOfStrings = { "one", "two", "three" };

    assertThat(firstArrayOfStrings).isEqualTo(secondArrayOfStrings);
}
```

或者如果它们是空的:

```java
@Test
public void whenCheckingArray_thenEmpty() {
    Object[] anArray = {};

    assertThat(anArray).isEmpty();
}
```

### 5.7。`Comparable`断言

除了测试一个 `Comparable`是否大于或小于另一个实例，我们还可以检查它们是否至少是一个给定值:

```java
@Test
public void whenCheckingComparable_thenAtLeast() {
    Comparable<Integer> aComparable = 5;

    assertThat(aComparable).isAtLeast(1);
}
```

此外，我们可以测试它们是否在特定范围内:

```java
@Test
public void whenCheckingComparable_thenInRange() {
    // ...

    assertThat(aComparable).isIn(Range.closed(1, 10));
}
```

或者在特定列表中:

```java
@Test
public void whenCheckingComparable_thenInList() {
    // ...

    assertThat(aComparable).isIn(Arrays.asList(4, 5, 6));
}
```

我们还可以根据类的`compareTo()`方法测试两个`Comparable`实例是否等价。

首先，让我们修改我们的`User`类来实现`Comparable`接口:

```java
public class User implements Comparable<User> {
    // ...

    public int compareTo(User o) {
        return this.getName().compareToIgnoreCase(o.getName());
    }
}
```

现在，让我们断言两个同名的用户是等价的:

```java
@Test
public void whenComparingUsers_thenEquivalent() {
    User aUser = new User();
    aUser.setName("John Doe");

    User anotherUser = new User();
    anotherUser.setName("john doe");

    assertThat(aUser).isEquivalentAccordingToCompareTo(anotherUser);
}
```

### 5.8。`Iterable`断言

除了断言一个`Iterable`实例的大小，不管它是空的还是没有重复的，大多数关于`Iterable`的典型断言是它包含一些元素:

```java
@Test
public void whenCheckingIterable_thenContains() {
    List<Integer> aList = Arrays.asList(4, 5, 6);

    assertThat(aList).contains(5);
}
```

它包含另一个`Iterable`的任何元素:

```java
@Test
public void whenCheckingIterable_thenContainsAnyInList() {
    List<Integer> aList = Arrays.asList(1, 2, 3);

    assertThat(aList).containsAnyIn(Arrays.asList(1, 5, 10));
}
```

主体具有相同的元素，以相同的顺序，像另一个:

```java
@Test
public void whenCheckingIterable_thenContainsExactElements() {
    List<String> aList = Arrays.asList("10", "20", "30");
    List<String> anotherList = Arrays.asList("10", "20", "30");

    assertThat(aList)
      .containsExactlyElementsIn(anotherList)
      .inOrder();
}
```

如果使用定制比较器订购:

```java
@Test
public void givenComparator_whenCheckingIterable_thenOrdered() {
    Comparator<String> aComparator
      = (a, b) -> new Float(a).compareTo(new Float(b));

    List<String> aList = Arrays.asList("1", "012", "0020", "100");

    assertThat(aList).isOrdered(aComparator);
}
```

### 5.9。`Map`断言

除了断言一个`Map`实例是否为空，或者具有特定的大小；我们可以检查它是否有特定的条目:

```java
@Test
public void whenCheckingMap_thenContainsEntry() {
    Map<String, Object> aMap = new HashMap<>();
    aMap.put("one", 1L);

    assertThat(aMap).containsEntry("one", 1L);
}
```

如果它有特定的键:

```java
@Test
public void whenCheckingMap_thenContainsKey() {
    // ...

    assertThat(map).containsKey("one");
}
```

或者如果它与另一个`Map`具有相同的条目:

```java
@Test
public void whenCheckingMap_thenContainsEntries() {
    Map<String, Object> aMap = new HashMap<>();
    aMap.put("first", 1L);
    aMap.put("second", 2.0);
    aMap.put("third", 3f);

    Map<String, Object> anotherMap = new HashMap<>(aMap);

    assertThat(aMap).containsExactlyEntriesIn(anotherMap);
}
```

### 5.10。`Exception`断言

对于`Exception`对象，只提供了两种重要的方法。

我们可以编写针对异常原因的断言:

```java
@Test
public void whenCheckingException_thenInstanceOf() {
    Exception anException
      = new IllegalArgumentException(new NumberFormatException());

    assertThat(anException)
      .hasCauseThat()
      .isInstanceOf(NumberFormatException.class);
}
```

或者它的信息:

```java
@Test
public void whenCheckingException_thenCauseMessageIsKnown() {
    Exception anException
      = new IllegalArgumentException("Bad value");

    assertThat(anException)
      .hasMessageThat()
      .startsWith("Bad");
}
```

### 5.11。`Class`断言

对于`Class`断言，只有一个重要的方法可以用来测试一个类是否可以分配给另一个类:

```java
@Test
public void whenCheckingClass_thenIsAssignable() {
    Class<Double> aClass = Double.class;

    assertThat(aClass).isAssignableTo(Number.class);
}
```

## 6。Java 8 断言

`Optional`和`Stream`是`Truth`支持的仅有的两种 Java 8 类型。

### 6.1。`Optional` 断言

有三种重要的方法来验证一个`Optional`。

我们可以测试它是否有特定的值:

```java
@Test
public void whenCheckingJavaOptional_thenHasValue() {
    Optional<Integer> anOptional = Optional.of(1);

    assertThat(anOptional).hasValue(1);
}
```

如果该值存在:

```java
@Test
public void whenCheckingJavaOptional_thenPresent() {
    Optional<String> anOptional = Optional.of("Baeldung");

    assertThat(anOptional).isPresent();
}
```

或者，如果该值不存在:

```java
@Test
public void whenCheckingJavaOptional_thenEmpty() {
    Optional anOptional = Optional.empty();

    assertThat(anOptional).isEmpty();
}
```

### 6.2。`Stream`断言

一个`Stream`的断言与一个`Iterable`的断言非常相似。

例如，我们可以测试一个特定的`Stream`是否以相同的顺序包含了一个`Iterable`的所有对象:

```java
@Test
public void whenCheckingStream_thenContainsInOrder() {
    Stream<Integer> anStream = Stream.of(1, 2, 3);

    assertThat(anStream)
      .containsAllOf(1, 2, 3)
      .inOrder();
}
```

更多例子，请参考`Iterable`断言部分。

## 7。番石榴断言

在这一节中，我们将看到`Truth`中支持的番石榴类型的断言示例。

### 7.1。`Optional`断言

一个番石榴`Optional`还有三个重要的断言方法。`hasValue()`和`isPresent()`方法的行为与 Java 8 `Optional`完全一样。

但是我们不用`isEmpty()`来断言`Optional`不存在，而是使用`isAbsent()`:

```java
@Test
public void whenCheckingGuavaOptional_thenIsAbsent() {
    Optional anOptional = Optional.absent();

    assertThat(anOptional).isAbsent();
}
```

### 7.2。`Multimap`断言

`Multimap`和标准的`Map`断言非常相似。

一个显著的区别是，我们可以在一个`Multimap`中获得一个键的多个值，并对这些值做出断言。

下面是一个测试“one”键的值的大小是否为 2 的示例:

```java
@Test
public void whenCheckingGuavaMultimap_thenExpectedSize() {
    Multimap<String, Object> aMultimap = ArrayListMultimap.create();
    aMultimap.put("one", 1L);
    aMultimap.put("one", 2.0);

    assertThat(aMultimap)
      .valuesForKey("one")
      .hasSize(2);
}
```

更多例子，请参考`Map`断言部分。

### 7.3。`Multiset`断言

`Multiset`对象的断言包括`Iterable`的断言和一个额外的方法，用于验证一个键是否有特定的出现次数:

```java
@Test
public void whenCheckingGuavaMultiset_thenExpectedCount() {
    TreeMultiset<String> aMultiset = TreeMultiset.create();
    aMultiset.add("baeldung", 10);

    assertThat(aMultiset).hasCount("baeldung", 10);
}
```

### 7.4。`Table`断言

除了检查它的大小或它在哪里是空的，我们还可以检查一个`Table`来验证它是否包含给定行和列的特定映射:

```java
@Test
public void whenCheckingGuavaTable_thenContains() {
    Table<String, String, String> aTable = TreeBasedTable.create();
    aTable.put("firstRow", "firstColumn", "baeldung");

    assertThat(aTable).contains("firstRow", "firstColumn");
}
```

或者如果它包含特定的单元格:

```java
@Test
public void whenCheckingGuavaTable_thenContainsCell() {
    Table<String, String, String> aTable = getDummyGuavaTable();

    assertThat(aTable).containsCell("firstRow", "firstColumn", "baeldung");
}
```

此外，我们可以检查它是否包含给定的行、列或值。参见相关测试用例的源代码。

## 8。自定义故障信息和标签

当断言失败时，`Truth`会显示非常易读的消息，指出到底哪里出错了。但是，有时有必要向这些消息添加更多信息，以提供有关所发生事件的更多细节。

`Truth`允许我们定制那些故障消息:

```java
@Test
public void whenFailingAssertion_thenCustomMessage() {
    assertWithMessage("TEST-985: Secret user subject was NOT null!")
      .that(new User())
      .isNull();
}
```

运行测试后，我们得到以下输出:

```java
TEST-985: Secret user subject was NOT null!:
  Not true that <[[email protected]](/web/20221128115819/https://www.baeldung.com/cdn-cgi/l/email-protection)> is null
```

此外，我们可以添加一个自定义标签，在错误消息中显示在我们的主题之前。当对象没有有用的字符串表示时，这可能会很方便:

```java
@Test
public void whenFailingAssertion_thenMessagePrefix() {
    User aUser = new User();

    assertThat(aUser)
      .named("User [%s]", aUser.getName())
      .isNull();
}
```

如果我们运行测试，我们可以看到以下输出:

```java
Not true that User [John Doe]
  (<[[email protected]](/web/20221128115819/https://www.baeldung.com/cdn-cgi/l/email-protection)>) is null
```

## 9。分机

扩展`Truth`意味着我们可以添加对定制类型的支持。为此，我们需要创建一个类:

*   扩展`Subject`类或它的一个子类
*   定义一个接受两个参数的构造函数——一个`FailureStrategy`和一个自定义类型的实例
*   声明一个`SubjectFactory`类型的字段，`Truth`将使用它来创建我们的自定义主题的实例
*   实现一个静态的`assertThat()`方法，该方法接受我们的自定义类型
*   公开我们的测试断言 API

现在我们知道了如何扩展`Truth`，让我们创建一个添加对类型`User`对象支持的类:

```java
public class UserSubject
  extends ComparableSubject<UserSubject, User> {

    private UserSubject(
      FailureStrategy failureStrategy, User target) {
        super(failureStrategy, target);
    }

    private static final
      SubjectFactory<UserSubject, User> USER_SUBJECT_FACTORY
      = new SubjectFactory<UserSubject, User>() {

        public UserSubject getSubject(
          FailureStrategy failureStrategy, User target) {
            return new UserSubject(failureStrategy, target);
        }
    };

    public static UserSubject assertThat(User user) {
        return Truth.assertAbout(USER_SUBJECT_FACTORY).that(user);
    }

    public void hasName(String name) {
        if (!actual().getName().equals(name)) {
            fail("has name", name);
        }
    }

    public void hasNameIgnoringCase(String name) {
        if (!actual().getName().equalsIgnoreCase(name)) {
            fail("has name ignoring case", name);
        }
    }

    public IterableSubject emails() {
        return Truth.assertThat(actual().getEmails());
    }
}
```

现在，我们可以静态导入自定义主题的`assertThat()`方法，并编写一些测试:

```java
@Test
public void whenCheckingUser_thenHasName() {
    User aUser = new User();

    assertThat(aUser).hasName("John Doe");
}

@Test
public void whenCheckingUser_thenHasNameIgnoringCase() {
    // ...

    assertThat(aUser).hasNameIgnoringCase("john doe");
}

@Test
public void givenUser_whenCheckingEmails_thenExpectedSize() {
    // ...

    assertThat(aUser)
      .emails()
      .hasSize(2);
}
```

## 10。结论

在本教程中，我们探索了`Truth`给我们编写更多可读测试和失败消息的可能性。

我们展示了受支持的 Java 和 Guava 类型的最流行的断言方法、定制的失败消息，并使用定制主题扩展了`Truth`。

和往常一样，本文的完整源代码可以在 Github 上找到[。](https://web.archive.org/web/20221128115819/https://github.com/eugenp/tutorials/tree/master/testing-modules/assertion-libraries)