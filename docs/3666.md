# 在 Java 中压缩集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collections-zip>

## 1.介绍

在本教程中，我们将演示如何将两个集合压缩成一个逻辑集合。

**`zip”`操作与标准的“串联”或“合并”**略有不同。虽然“concat”或“merge”操作只是在现有集合的末尾添加新的集合，但“`zip”`操作将从每个集合中取出一个元素，并将它们组合起来。

核心库并不隐式支持"`zip”`,但是肯定有第三方库支持这个有用的操作。

考虑两个列表，一个包含人名，另一个包含他们的年龄。

```
List<String> names = new ArrayList<>(Arrays.asList("John", "Jane", "Jack", "Dennis"));

List<Integer> ages = new ArrayList<>(Arrays.asList(24, 25, 27));
```

压缩之后，我们得到了由这两个集合中的相应元素构造的姓名-年龄对。

## 2.使用 Java 8 `IntStream`

使用核心 Java，我们可以使用`IntStream`生成索引，然后使用它们从两个集合中提取相应的元素:

```
IntStream
  .range(0, Math.min(names.size(), ages.size()))
  .mapToObj(i -> names.get(i) + ":" + ages.get(i))
  // ...
```

## 3.使用番石榴流

从版本 21 开始，Google Guava 在`Streams`类中提供了一个 zip helper 方法。这消除了创建和映射索引的所有麻烦，并减少了输入和操作的语法:

```
Streams
  .zip(names.stream(), ages.stream(), (name, age) -> name + ":" + age)
  // ...
```

## 4.使用`jOOλ` `(jOOL)`

`jOOL`还在 Java 8 Lambda 上提供了一些有趣的附加功能，在`Tuple1`到`Tuple16,`的支持下，压缩操作变得有趣多了:

```
Seq
  .of("John","Jane", "Dennis")
  .zip(Seq.of(24,25,27));
```

这将产生一个包含压缩元素`Tuples`的`Seq`的结果:

```
(tuple(1, "a"), tuple(2, "b"), tuple(3, "c"))
```

`jOOL's zip`方法给出了提供自定义转换功能的灵活性:

```
Seq
  .of(1, 2, 3)
  .zip(Seq.of("a", "b", "c"), (x, y) -> x + ":" + y);
```

或者，如果一个人希望只压缩索引，他可以使用由`jOOL:`提供的`zipWithIndex`方法

```
Seq.of("a", "b", "c").zipWithIndex();
```

## 5.结论

在这个快速教程中，我们看了一下如何执行`zip`操作。

和往常一样，文章中的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20221126224448/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections)