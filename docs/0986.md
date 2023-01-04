# 使用流收集到树集中

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-collect-into-treeset>

## 1.概观

Java 8 中一个重要的新特性是[流 API](/web/20221017060640/https://www.baeldung.com/java-8-streams) 。流允许我们方便地处理来自不同来源的元素，比如数组或集合。

进一步，使用 [`Stream.collect()`](/web/20221017060640/https://www.baeldung.com/java-8-collectors#Collect) 方法和相应的 [`Collectors`](/web/20221017060640/https://www.baeldung.com/java-8-collectors) ，我们可以将元素重新打包成不同的数据结构，如`[Set](/web/20221017060640/https://www.baeldung.com/java-hashset)`、 [`Map`、](/web/20221017060640/https://www.baeldung.com/java-hashmap)、`[List](/web/20221017060640/https://www.baeldung.com/java-arraylist),`等。

在本教程中，我们将探索如何将`Stream `中的元素收集到 [`TreeSet`](/web/20221017060640/https://www.baeldung.com/java-tree-set) 中。

## 2.收集成一个自然有序的`TreeSet`

简单地说，`TreeSet`是一个有序集合。`TreeSet`中的元素使用它们的自然排序或提供的 [`Comparator`](/web/20221017060640/https://www.baeldung.com/java-comparator-comparable) 进行排序。

我们将首先看看如何使用自然排序来收集`Stream`元素。然后，让我们关注使用定制`Comparator`案例收集元素。

为了简单起见，我们将使用单元测试断言来验证我们是否得到了预期的`TreeSet`结果。

### 2.1.将字符串收集到一个`TreeSet`

既然`String `实现了`Comparable`接口，我们先以`String`为例，看看如何在一个`TreeSet`中收集它们:

```
String kotlin = "Kotlin";
String java = "Java";
String python = "Python";
String ruby = "Ruby";
TreeSet<String> myTreeSet = Stream.of(ruby, java, kotlin, python).collect(Collectors.toCollection(TreeSet::new));
assertThat(myTreeSet).containsExactly(java, kotlin, python, ruby);
```

如上面的测试所示，为了将`Stream`元素收集到`TreeSet`中，我们只需**将`TreeSet`的默认构造函数作为[方法引用](/web/20221017060640/https://www.baeldung.com/java-method-references)或[λ表达式](/web/20221017060640/https://www.baeldung.com/java-8-lambda-expressions-tips)传递给 [`Collectors.toCollection()`](/web/20221017060640/https://www.baeldung.com/java-8-collectors#3-collectorstocollection) 方法**。

如果我们执行这个测试，它就通过了。

接下来，让我们看一个类似的自定义类的例子。

### 2.2.收集自然排序的玩家

首先，让我们看看我们的`Player`类:

```
public class Player implements Comparable<Player> {
    private String name;
    private int age;
    private int numberOfPlayed;
    private int numberOfWins;

    public Player(String name, int age, int numberOfPlayed, int numberOfWins) {
        this.name = name;
        this.age = age;
        this.numberOfPlayed = numberOfPlayed;
        this.numberOfWins = numberOfWins;
    }

    @Override
    public int compareTo(Player o) {
        return Integer.compare(age, o.age);
    }

    // getters are omitted
} 
```

如上面的类所示，我们的`Player`类实现了`Comparable`接口。此外，**我们已经在`compareTo()`方法中定义了它的自然排序:玩家的年龄。**

接下来，让我们创建几个`Player`实例:

```
/*                          name  |  age  | num of played | num of wins
                           --------------------------------------------- */
Player kai = new Player(   "Kai",     26,       28,            7);
Player eric = new Player(  "Eric",    28,       30,           11);
Player saajan = new Player("Saajan",  30,      100,           66);
Player kevin = new Player( "Kevin",   24,       50,           49);
```

因为我们将在后面的其他演示中使用这四个玩家对象，所以我们将代码放在类似表格的格式中，以便于检查每个玩家的属性值。

现在，让我们按照它们的自然顺序将它们收集到一个`TreeSet`中，并验证我们是否得到了预期的结果:

```
TreeSet<Player> myTreeSet = Stream.of(saajan, eric, kai, kevin).collect(Collectors.toCollection(TreeSet::new));
assertThat(myTreeSet).containsExactly(kevin, kai, eric, saajan);
```

正如我们所见，代码非常类似于将字符串收集到一个`TreeSet`中。由于`Player`的`compareTo() `方法已经指定了`“age”`属性作为它的自然排序，我们用按照`age`升序排序的玩家来验证结果(`myTreeSet`)。

值得一提的是，我们已经使用了 **[AssertJ](/web/20221017060640/https://www.baeldung.com/introduction-to-assertj) 的`containsExactly()`方法来验证`TreeSet`只包含给定的元素，没有其他的**。

接下来，我们将看看如何使用定制的`Comparator`将这些玩家收集到一个`TreeSet`中。

## 3.用定制的`Comparator`收集到一个`TreeSet`

我们已经看到,`Collectors.toCollection(TreeSet::new)`允许我们按照自然顺序收集`Stream`到`TreeSet`中的元素。`TreeSet`提供了另一个接受`Comparator`对象作为参数的构造函数:

```
public TreeSet(Comparator<? super E> comparator) { ... }
```

因此，**如果我们希望`TreeSet`对元素应用不同的排序，我们可以创建一个`Comparator`对象，并将其传递给上面提到的构造函数**。

接下来，让我们根据这些玩家的获胜次数而不是他们的年龄来收集他们:

```
TreeSet<Player> myTreeSet = Stream.of(saajan, eric, kai, kevin)
  .collect(Collectors.toCollection(() -> new TreeSet<>(Comparator.comparingInt(Player::getNumberOfWins))
));
assertThat(myTreeSet).containsExactly(kai, eric, kevin, saajan);
```

这一次，我们使用 lambda 表达式创建了`TreeSet`实例。此外，我们已经使用`Comparator.comparingInt()`将自己的`Comparator`传递给了`TreeSet`的构造函数。

`Player::getNumberOfWins`引用我们需要比较玩家的属性值。

我们试一试，测试就通过了。

然而，所需的比较逻辑有时并不像示例所示的那样简单，只是比较一个属性的值。例如，我们可能需要比较一些额外计算的结果。

所以最后，让我们再把这些玩家收集到一个`TreeSet`里。但这一次，我们希望它们按胜率(胜率/出牌次数)排序:

```
TreeSet<Player> myTreeSet = Stream.of(saajan, eric, kai, kevin)
  .collect(Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(player -> BigDecimal.valueOf(player.getNumberOfWins())
    .divide(BigDecimal.valueOf(player.getNumberOfPlayed()), 2, RoundingMode.HALF_UP)))));
assertThat(myTreeSet).containsExactly(kai, eric, saajan, kevin);
```

正如上面的测试所示，**我们已经使用了`Comparator.comparing(Function keyExtractor)`方法来指定可比较的排序关键字**。在这个例子中，`keyExtractor`函数是一个 lambda 表达式，它计算玩家的胜率。

此外，如果我们运行测试，它会通过。所以我们得到了预期的`TreeSet`。

## 4.结论

在本文中，我们通过示例讨论了如何通过自然排序和自定义比较器将`Stream `中的元素收集到`TreeSet`中。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221017060640/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-set-2)