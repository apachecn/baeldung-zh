# 使用 Java-LSH 的 Java 中的局部敏感散列

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/locality-sensitive-hashing>

## 1。概述

[位置敏感散列(LSH)](https://web.archive.org/web/20220524032250/https://en.wikipedia.org/wiki/Locality-sensitive_hashing) 算法对输入项进行散列，以便相似的项很有可能被映射到相同的存储桶。

在这篇简短的文章中，我们将使用`java-lsh` 库来演示这个算法的一个简单用例。

## 2。Maven 依赖关系

首先，我们需要将 Maven 依赖项添加到 [`java-lsh`](https://web.archive.org/web/20220524032250/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22info.debatty%22%20AND%20a%3A%22java-lsh%22) 库中:

```java
<dependency>
    <groupId>info.debatty</groupId>
    <artifactId>java-lsh</artifactId>
    <version>0.10</version>
</dependency>
```

## 3。位置敏感哈希用例

LSH 有许多可能的应用，但我们将考虑一个特殊的例子。

假设我们有一个文档数据库，想要实现一个能够识别相似文档的搜索引擎。

我们可以将 LSH 作为解决方案的一部分:

*   每个文档都可以被转换成数字或布尔的向量——例如，我们可以使用`[word2vect](https://web.archive.org/web/20220524032250/https://en.wikipedia.org/wiki/Word2vec)` 算法将单词和文档转换成数字的向量
*   一旦我们有了表示每个文档的向量，我们就可以使用 LSH 算法来计算每个向量的散列，并且由于 LSH 的特性，被表示为相似向量的文档将具有相似或相同的散列
*   因此，给定一个特定的文档向量，我们可以找到`N`个具有相似散列的向量，并将相应的文档返回给最终用户

## 4。示例

我们将使用`java-lsh`库来计算输入向量的散列值。我们不会讨论转换本身，因为这是一个超出本文范围的大话题。

然而，假设我们有三个输入向量，它们是从一组三个文档转换而来的，以可用作 LSH 算法的输入的形式呈现:

```java
boolean[] vector1 = new boolean[] {true, true, true, true, true};
boolean[] vector2 = new boolean[] {false, false, false, true, false};
boolean[] vector3 = new boolean[] {false, false, true, true, false}; 
```

请注意，在生产应用程序中，输入向量的数量应该多得多，才能利用 LSH 算法，但是为了便于演示，我们将只使用三个向量。

值得注意的是，第一个向量与第二个和第三个向量有很大不同，而第二个和第三个向量彼此非常相似。

让我们创建一个`LSHMinHash`类的实例。我们需要将输入向量的大小传递给它——所有输入向量的大小应该相等。我们还需要指定我们想要多少个哈希桶，以及 LSH 应该执行多少阶段的计算(迭代):

```java
int sizeOfVectors = 5;
int numberOfBuckets = 10;
int stages = 4;

LSHMinHash lsh = new LSHMinHash(stages, numberOfBuckets, sizeOfVectors); 
```

我们指定所有将被算法散列的向量应该在十个桶中被散列。我们还希望有四次 LSH 迭代来计算哈希值。

为了计算每个向量的散列，我们将向量传递给`hash()`方法:

```java
int[] firstHash = lsh.hash(vector1);
int[] secondHash = lsh.hash(vector2);
int[] thirdHash = lsh.hash(vector3);

System.out.println(Arrays.toString(firstHash));
System.out.println(Arrays.toString(secondHash));
System.out.println(Arrays.toString(thirdHash)); 
```

运行该代码将产生类似于以下内容的输出:

```java
[0, 0, 1, 0]
[9, 3, 9, 8]
[1, 7, 8, 8] 
```

查看每个输出数组，我们可以看到在四次迭代的每一次中为相应的输入向量计算的哈希值。第一行显示第一个向量的散列结果，第二行显示第二个向量的散列结果，第三行显示第三个向量的散列结果。

经过四次迭代后，LSH 得出了我们预期的结果——LSH 为第二个和第三个向量计算了相同的哈希值(8 ),这两个向量彼此相似，而为第一个向量计算了不同的哈希值(0 ),这两个向量不同。

LSH 是一种基于概率的算法，因此我们不能确定两个相似的向量会落在同一个哈希桶中。然而，当我们有足够多的输入向量时，**该算法产生的结果将有很高的概率将相似的向量分配给相同的桶**。

当我们处理海量数据集时，LSH 可以是一个方便的算法。

## 5。结论

在这篇简短的文章中，我们看了一个本地敏感散列算法的应用，并展示了如何在`java-lsh` 库的帮助下使用它。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220524032250/https://github.com/eugenp/tutorials/tree/master/libraries)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。