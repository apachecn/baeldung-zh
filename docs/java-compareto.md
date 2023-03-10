# 实现 compareTo 方法的指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compareto>

## 1.概观

作为 Java 开发人员，我们经常需要对集合中组合在一起的元素进行排序。 **Java 允许我们用任何类型的数据实现各种排序算法**。

例如，我们可以按字母顺序、反字母顺序或基于长度对字符串进行排序。

在本教程中，我们将探索`Comparable`接口及其`compareTo`方法，它支持排序。我们将研究包含核心类和定制类对象的排序集合。

我们还将提到正确实现`compareTo`的规则，以及需要避免的破坏模式。

## 2.*可比的*界面

[`Comparable`接口](/web/20221208143921/https://www.baeldung.com/java-comparator-comparable)对实现它的每个类的对象实施**排序**。

`compareTo`是由`Comparable`接口定义的唯一方法。它通常被称为自然比较法。

### 2.1.实施`compareTo`

`compareTo`方法**将当前对象与作为参数**发送的对象进行比较。

在实现它时，我们需要确保该方法返回:

*   如果当前对象大于参数对象，则为正整数
*   如果当前对象小于参数对象，则为负整数
*   如果当前对象等于参数对象，则为零

在数学中，我们称之为符号或符号函数:

[![Signum function](img/71d4932b19d20ac534980b8d2372dfc4.png)](/web/20221208143921/https://www.baeldung.com/wp-content/uploads/2021/02/2021-01-24-10_27_03-notation-What-does-sgn-mean_-Mathematics-Stack-Exchange.png)

### 2.2.示例实现

让我们看看`compareTo`方法是如何在核心`Integer`类中实现的:

```java
@Override
public int compareTo(Integer anotherInteger) {
    return compare(this.value, anotherInteger.value);
}

public static int compare (int x, int y) {
    return (x < y) ? -1 : ((x == y) ? 0 : 1);
}
```

### 2.3.断裂的减法模式

有人可能会说，我们可以用一个聪明的减法一行程序来代替:

```java
@Override
public int compareTo(BankAccount anotherAccount) {
    return this.balance - anotherAccount.balance;
}
```

让我们考虑一个例子，我们期望正的账户余额大于负的账户余额:

```java
BankAccount accountOne = new BankAccount(1900000000);
BankAccount accountTwo = new BankAccount(-2000000000);
int comparison = accountOne.compareTo(accountTwo);
assertThat(comparison).isNegative();
```

然而，一个整数不够大，不足以存储差值，从而给我们错误的结果。当然，这个**模式由于可能的整数溢出而被破坏，并且需要被避免**。

正确的解决方法是用比较代替减法。我们也可以重用核心`Integer`类的正确实现:

```java
@Override
public int compareTo(BankAccount anotherAccount) {
    return Integer.compare(this.balance, anotherAccount.balance);
}
```

### 2.4.实施规则

为了正确实施`compareTo`方法，我们需要遵守以下数学规则:

*   `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`
*   `(x.compareTo(y) > 0 && y.compareTo(z) > 0)` 暗指`x.compareTo(z) > 0`
*   `x.compareTo(y) == 0`暗示着`sgn(x.compareTo(z)) == sgn(y.compareTo(z))`

虽然不是必需的，但也强烈建议**保持`compareTo`实现与** [`**equals**`](/web/20221208143921/https://www.baeldung.com/java-comparing-objects#:~:text=Object%23equals%20Method&text=This%20method%20is%20defined%20in,equality%20means%20for%20our%20objects.) **方法实现** `:`一致

*   `x.compareTo(e2) == 0`应具有与`x.equals(y)`相同的布尔值

这将确保我们可以安全地使用有序集合和有序映射中的对象。

### 2.5.与`equals`的一致性

让我们看看当`compareTo`和`equals`实现不一致时会发生什么。

在我们的例子中，`compareTo`方法检查进球得分，而`equals`方法检查球员姓名:

```java
@Override
public int compareTo(FootballPlayer anotherPlayer) {
    return this.goalsScored - anotherPlayer.goalsScored;
}

@Override
public boolean equals(Object object) {
    if (this == object)
        return true;
    if (object == null || getClass() != object.getClass())
        return false;
    FootballPlayer player = (FootballPlayer) object;
    return name.equals(player.name);
}
```

在排序集合或排序映射中使用此类时，这可能会导致意外行为:

```java
FootballPlayer messi = new FootballPlayer("Messi", 800);
FootballPlayer ronaldo = new FootballPlayer("Ronaldo", 800);

TreeSet<FootballPlayer> set = new TreeSet<>();
set.add(messi);
set.add(ronaldo);

assertThat(set).hasSize(1);
assertThat(set).doesNotContain(ronaldo);
```

排序后的集合使用`compareTo`而不是`equals` 方法执行所有元素比较。因此，从它的角度来看，这两个玩家似乎是等价的，它不会添加第二个玩家。

## 3.分类收藏

接口`Comparable`的主要目的是**启用集合或数组**中分组元素的自然排序。

我们可以使用 Java 实用方法 [`Collections.sort`或`Arrays.sort`](/web/20221208143921/https://www.baeldung.com/java-sorting) 对所有实现`Comparable` 的对象进行排序。

### 3.1.核心 Java 类

大多数核心 Java 类，如`String`、`Integer`或`Double`，已经实现了`Comparable`接口。

因此，对它们进行排序非常简单，因为我们可以重用它们现有的自然排序实现。

按自然顺序对数字进行排序将导致升序排列:

```java
int[] numbers = new int[] {5, 3, 9, 11, 1, 7};
Arrays.sort(numbers);
assertThat(numbers).containsExactly(1, 3, 5, 7, 9, 11);
```

另一方面，字符串的自然排序将导致字母顺序:

```java
String[] players = new String[] {"ronaldo",  "modric", "ramos", "messi"};
Arrays.sort(players);
assertThat(players).containsExactly("messi", "modric", "ramos", "ronaldo");
```

### 3.2.自定义类别

相反，对于任何可排序的**自定义类，我们需要手动实现`Comparable` 接口**。

如果我们试图对没有实现`Comparable`的对象集合进行排序，Java 编译器将抛出一个错误。

如果我们对数组进行同样的尝试，它在编译过程中不会失败。但是，这将导致类强制转换运行时异常:

```java
HandballPlayer duvnjak = new HandballPlayer("Duvnjak", 197);
HandballPlayer hansen = new HandballPlayer("Hansen", 196);
HandballPlayer[] players = new HandballPlayer[] {duvnjak, hansen};
assertThatExceptionOfType(ClassCastException.class).isThrownBy(() -> Arrays.sort(players));
```

### 3.3.`TreeMap`和`TreeSet`

[`TreeMap`](/web/20221208143921/https://www.baeldung.com/java-treemap) 和 [`TreeSet`](/web/20221208143921/https://www.baeldung.com/java-tree-set) 是 Java 集合框架中的两个实现，**帮助我们对它们的元素**进行自动排序。

我们可以在一个排序的映射中使用实现`Comparable`接口的对象，或者作为一个排序集合中的元素。

让我们来看一个自定义类的示例，该类根据球员的进球数来比较他们:

```java
@Override
public int compareTo(FootballPlayer anotherPlayer) {
    return Integer.compare(this.goalsScored, anotherPlayer.goalsScored);
}
```

在我们的示例中，键是根据在 *compareTo* 实现中定义的标准自动排序的:

```java
FootballPlayer ronaldo = new FootballPlayer("Ronaldo", 900);
FootballPlayer messi = new FootballPlayer("Messi", 800);
FootballPlayer modric = new FootballPlayer("modric", 100);

Map<FootballPlayer, String> players = new TreeMap<>();
players.put(ronaldo, "forward");
players.put(messi, "forward");
players.put(modric, "midfielder");

assertThat(players.keySet()).containsExactly(modric, messi, ronaldo);
```

## 4.`Comparator`备选方案

除了自然排序， **Java 还允许我们以灵活的方式定义特定的排序逻辑。**

[`Comparator`接口](/web/20221208143921/https://www.baeldung.com/java-comparator-comparable)允许从我们正在排序的对象中分离出多个不同的比较策略:

```java
FootballPlayer ronaldo = new FootballPlayer("Ronaldo", 900);
FootballPlayer messi = new FootballPlayer("Messi", 800);
FootballPlayer modric = new FootballPlayer("Modric", 100);

List<FootballPlayer> players = Arrays.asList(ronaldo, messi, modric);
Comparator<FootballPlayer> nameComparator = Comparator.comparing(FootballPlayer::getName);
Collections.sort(players, nameComparator);

assertThat(players).containsExactly(messi, modric, ronaldo);
```

当我们不想或者不能修改我们想要排序的对象的源代码时，这通常也是一个不错的选择。

## 5.结论

在本文中，我们研究了如何**使用`Comparable`接口为我们的 Java 类定义自然排序算法**。我们查看了一个常见的中断模式，并定义了如何正确实现`compareTo`方法。

我们还探索了对包含核心类和定制类的集合进行排序。接下来，我们考虑在有序集合和有序映射中使用的类中实现`compareTo` 方法。

最后，我们看了几个应该使用`Comparator`接口的用例。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143921/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-3)