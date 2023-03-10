# 在 Java 中混合集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-shuffle-collection>

## 1。概述

在这篇简短的文章中，我们将看到如何在 Java 中洗牌。Java 有一个内置的方法来混洗`List`对象——我们也将把它用于其他集合。

## 2。洗牌列表

**我们将使用`java.util.Collections.shuffle`** 的方法，其中 将 a `List`作为输入，并将其就地洗牌。所谓就地，我们的意思是它打乱了输入中传递的相同列表，而不是用打乱的元素创建一个新列表。

让我们看一个简单的例子，展示如何洗牌:

```java
List<String> students = Arrays.asList("Foo", "Bar", "Baz", "Qux");
Collections.shuffle(students);
```

**还有第二个版本的`java.util.Collections.shuffle`** **，它也接受自定义来源的[随机性](/web/20220929201254/https://www.baeldung.com/cs/randomness)作为输入。**如果我们的应用有这样的要求，这可以用来使洗牌成为一个确定性的过程。

让我们使用第二种变体在两个列表上实现相同的洗牌:

```java
List<String> students_1 = Arrays.asList("Foo", "Bar", "Baz", "Qux");
List<String> students_2 = Arrays.asList("Foo", "Bar", "Baz", "Qux");

int seedValue = 10;

Collections.shuffle(students_1, new Random(seedValue));
Collections.shuffle(students_2, new Random(seedValue));

assertThat(students_1).isEqualTo(students_2);
```

**当使用相同的随机来源(从相同的种子值初始化)时，两次洗牌产生的随机数序列将是相同的。**因此，在洗牌之后，两个列表将包含完全相同顺序的元素。

## 3。无序集合的洗牌元素

**我们可能也想打乱其他集合，例如`Set, Map,`或`Queue`，但是所有这些集合都是无序的**——它们不保持任何特定的顺序。

一些实现，比如`LinkedHashMap`，或者带有`Comparator`的`Set`——确实保持固定的顺序，因此我们也不能打乱它们。

然而，**我们仍然可以随机访问它们的元素，首先将它们转换成一个`List`，然后洗牌这个`List`。**

让我们来看一个快速洗牌的例子:

```java
Map<Integer, String> studentsById = new HashMap<>();
studentsById.put(1, "Foo");
studentsById.put(2, "Bar");
studentsById.put(3, "Baz");
studentsById.put(4, "Qux");

List<Map.Entry<Integer, String>> shuffledStudentEntries
 = new ArrayList<>(studentsById.entrySet());
Collections.shuffle(shuffledStudentEntries);

List<String> shuffledStudents = shuffledStudentEntries.stream()
  .map(Map.Entry::getValue)
  .collect(Collectors.toList());
```

类似地，我们可以打乱一个`Set`的元素:

```java
Set<String> students = new HashSet<>(
  Arrays.asList("Foo", "Bar", "Baz", "Qux"));
List<String> studentList = new ArrayList<>(students);
Collections.shuffle(studentList);
```

## 4。结论

在这个快速教程中，我们看到了如何在 Java 中使用`java.util.Collections.shuffle` 来洗牌。

这自然直接与`List,`一起工作，我们也可以间接利用它来随机化其他集合中元素的顺序。 我们还可以通过提供一个自定义的随机性来源来控制洗牌过程，并使其具有确定性。 

像往常一样，本文中演示的所有代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220929201254/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-2)