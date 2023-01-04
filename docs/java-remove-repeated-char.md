# 从字符串中删除重复字符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-repeated-char>

## 1.概观

在本教程中，我们将讨论 Java 中如何从字符串中删除重复字符的几种技术。

对于每种技术，**我们还将简要讨论它的时间和空间复杂度。**

## 2.使用`distinct`

让我们从使用 Java 8 中引入的`distinct`方法删除字符串中的重复项开始。

下面，我们从一个给定的字符串对象中获取一个`Int` `S` `tream`的实例。然后，我们使用`distinct`方法删除重复的内容。最后，我们调用`forEach`方法来循环不同的字符，并将它们附加到我们的 [`StringBuilder`](/web/20220913031431/https://www.baeldung.com/java-string-builder-string-buffer) :

```java
StringBuilder sb = new StringBuilder();
str.chars().distinct().forEach(c -> sb.append((char) c));
```

**时间复杂度:**`O(n)`–循环的运行时间与输入字符串的大小成正比

**辅助空间:**`O(n)`–因为`distinct` 在内部使用了一个`LinkedHashSet`,我们也将结果字符串存储在一个`StringBuilder` 对象中

**保持顺序:**是的——因为`LinkedHashSet `保持其元素的顺序

虽然 Java 8 很好地为我们完成了这项任务，但是让我们将它与我们自己的努力进行比较。

## 3.使用`indexOf`

从字符串中删除重复项的简单方法是**遍历输入，并使用 [`indexOf` 方法](/web/20220913031431/https://www.baeldung.com/string/index-of)检查当前字符是否已经存在于结果字符串**中:

```java
StringBuilder sb = new StringBuilder();
int idx;
for (int i = 0; i < str.length(); i++) {
    char c = str.charAt(i);
    idx = str.indexOf(c, i + 1);
    if (idx == -1) {
        sb.append(c);
    }
} 
```

**时间复杂度:**`O(n * n)`–对于每个字符，`indexOf`方法遍历剩余的字符串

**辅助空间:**`O(n)`–线性空间是必需的，因为我们使用`StringBuilder`来存储结果

**维持秩序:**是

这种方法与第一种方法具有相同的空间复杂度，但是**执行起来要慢得多。**

## 4.使用字符数组

我们也可以从字符串中删除重复的字符，方法是**将其转换成一个`char`数组，然后循环遍历每个字符，并将其与所有后续字符**进行比较。

正如我们在下面看到的，我们正在创建两个`for`循环，并且我们正在检查每个元素是否在字符串中重复。如果发现重复，我们不会将其附加到`StringBuilder`:

```java
char[] chars = str.toCharArray();
StringBuilder sb = new StringBuilder();
boolean repeatedChar;
for (int i = 0; i < chars.length; i++) {
    repeatedChar = false;
    for (int j = i + 1; j < chars.length; j++) {
        if (chars[i] == chars[j]) {
            repeatedChar = true;
            break;
        }
    }
    if (!repeatedChar) {
        sb.append(chars[i]);
    }
} 
```

**时间复杂度:**`O(n * n)`–我们有一个内部循环和一个外部循环都遍历输入字符串

**辅助空格:**`O(n)`–线性空格是必需的，因为`chars`变量存储了字符串输入的新副本，我们还使用了`StringBuilder`来保存结果

**维持秩序:**是

同样，与核心 Java 产品相比，我们的第二次尝试表现不佳，但让我们看看下一次尝试的结果。

## 5.使用排序

或者，可以通过对输入字符串进行排序，将重复的字符分组，从而消除重复的字符。**为了做到这一点，我们必须将字符串转换成一个`char a` rray，并使用`Arrays`对其进行排序。`sort` 方法`.`最后，我们将迭代排序后的`char`数组。**

在每次迭代中，我们都要将数组中的每个元素与前一个元素进行比较。如果元素不同，那么我们将把当前字符附加到`StringBuilder:`

```java
StringBuilder sb = new StringBuilder();
if(!str.isEmpty()) {
    char[] chars = str.toCharArray();
    Arrays.sort(chars);

    sb.append(chars[0]);
    for (int i = 1; i < chars.length; i++) {
        if (chars[i] != chars[i - 1]) {
            sb.append(chars[i]);
        }
    }
}
```

**时间复杂度:**`O(n log n)`–排序使用了[双支点快速排序](/web/20220913031431/https://www.baeldung.com/arrays-sortobject-vs-sortint)，在许多数据集上提供 O(n log n)的性能

**辅助空格:**`O(n)`–自`toCharArray` 方法制作输入`String`的副本

**维持秩序:**否

让我们最后再试一次。

## 6.使用`Set`

另一种从字符串中删除重复字符的方法是使用`Set`。**如果我们不关心输出字符串中字符的顺序，我们可以使用 [`HashSet`](/web/20220913031431/https://www.baeldung.com/java-hashset) 。** **否则，我们可以用一个`LinkedHashSet `来维持插入顺序。**

在这两种情况下，我们将遍历输入字符串并将每个字符添加到`Set`中。一旦字符被插入到集合中，我们将迭代它以将它们添加到`StringBuilder `并返回结果字符串:

```java
StringBuilder sb = new StringBuilder();
Set<Character> linkedHashSet = new LinkedHashSet<>();

for (int i = 0; i < str.length(); i++) {
    linkedHashSet.add(str.charAt(i));
}

for (Character c : linkedHashSet) {
    sb.append(c);
} 
```

**时间复杂度:**`O(n)`–循环的运行时间与输入字符串的大小成正比

**辅助空格:**`O(n)`—`Set`需要的空格取决于输入字符串的大小；此外，我们使用`StringBuilder`来存储结果

**维持秩序:** `LinkedHashSet – `是，`HashSet `–否

现在，我们已经匹配了核心 Java 方法！发现这与`distinct` 已经做的非常相似并不令人震惊。

## 7.结论

在本文中，我们介绍了几种在 Java 中从字符串中删除重复字符的方法。我们还研究了每种方法的时间和空间复杂度。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220913031431/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms)