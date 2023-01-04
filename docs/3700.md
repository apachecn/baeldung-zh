# Java 中的 Trie 数据结构

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/trie-java>

## 1。概述

数据结构是计算机编程中的重要资产，知道何时以及为什么使用它们是非常重要的。

本文简要介绍了 trie(读作“try”)数据结构、它的实现和复杂性分析。

## 2 .排序〔t1〕

trie 树是一种离散的数据结构，在典型的算法课程中并不广为人知或被广泛提及，但却是一种重要的数据结构。

trie(也称为数字树)有时甚至是基数树或前缀树(因为它们可以通过前缀进行搜索)，是一种有序的树结构，它利用了它存储的键(通常是字符串)。

节点在树中的位置定义了与该节点相关联的键，这使得 try 与二分搜索法树不同，在后者中，节点存储仅对应于该节点的键。

一个节点的所有后代都有一个与该节点相关联的公共前缀`String`，而根节点则与一个空的`String.`相关联

这里我们有一个`TrieNode` 的预览，我们将在`Trie:`的实现中使用它

```
public class TrieNode {
    private HashMap<Character, TrieNode> children;
    private String content;
    private boolean isWord;

   // ...
}
```

可能存在 trie 是二叉查找树的情况，但一般来说，这些是不同的。二分搜索法树和特里都是树，但是二分搜索法树中的每个节点总是有两个子节点，而特里的节点则可以有更多子节点。

在 trie 中，每个节点(除了根节点)存储一个字符或一个数字。通过从根节点到特定节点`n`向下遍历 trie，可以形成由 trie 的其他分支共享的字符或数字的公共前缀。

通过从叶节点到根节点遍历 trie，可以形成一个`String`或一个数字序列。

下面是`Trie` 类，它代表了 trie 数据结构的一个实现:

```
public class Trie {
    private TrieNode root;
    //...
}
```

## 3。常见操作

现在，让我们看看如何实现基本操作。

### 3.1。插入元素

我们将描述的第一个操作是插入新节点。

在我们开始实施之前，理解算法很重要:

1.  将当前节点设置为根节点
2.  将当前字母设为单词的首字母
3.  如果当前节点已经存在对当前字母的引用(通过“children”字段中的一个元素)，则将当前节点设置为该引用节点。否则，创建一个新节点，将字母设置为当前字母，并将当前节点初始化为这个新节点
4.  重复步骤 3，直到钥匙被遍历

**该操作的复杂度为`O(n)`，其中`n`代表密钥大小。**

下面是该算法的实现:

```
public void insert(String word) {
    TrieNode current = root;

    for (char l: word.toCharArray()) {
        current = current.getChildren().computeIfAbsent(l, c -> new TrieNode());
    }
    current.setEndOfWord(true);
}
```

现在让我们看看如何使用这个方法在一个 trie 中插入新元素:

```
private Trie createExampleTrie() {
    Trie trie = new Trie();

    trie.insert("Programming");
    trie.insert("is");
    trie.insert("a");
    trie.insert("way");
    trie.insert("of");
    trie.insert("life");

    return trie;
}
```

我们可以通过以下测试来测试 trie 是否已经填充了新节点:

```
@Test
public void givenATrie_WhenAddingElements_ThenTrieNotEmpty() {
    Trie trie = createTrie();

    assertFalse(trie.isEmpty());
}
```

### 3.2。寻找元素

现在让我们添加一个方法来检查一个特定的元素是否已经存在于一个 trie 中:

1.  获取根的子代
2.  遍历`String`的每个字符
3.  检查该字符是否已经是子 trie 的一部分。如果它在 trie 中不存在，那么停止搜索并返回`false`
4.  重复第二步和第三步，直到`String.`中没有任何字符。如果到达`String`的末尾，返回`true`

**这个算法的复杂度是`O(n)`，其中 n 代表密钥的长度。**

Java 实现可能看起来像:

```
public boolean find(String word) {
    TrieNode current = root;
    for (int i = 0; i < word.length(); i++) {
        char ch = word.charAt(i);
        TrieNode node = current.getChildren().get(ch);
        if (node == null) {
            return false;
        }
        current = node;
    }
    return current.isEndOfWord();
}
```

实际上:

```
@Test
public void givenATrie_WhenAddingElements_ThenTrieContainsThoseElements() {
    Trie trie = createExampleTrie();

    assertFalse(trie.containsNode("3"));
    assertFalse(trie.containsNode("vida"));
    assertTrue(trie.containsNode("life"));
}
```

### 3.3。删除一个元素

除了插入和查找元素，显然我们还需要能够删除元素。

对于删除过程，我们需要遵循以下步骤:

1.  检查该元素是否已经是 trie 的一部分
2.  如果找到了该元素，则将其从 trie 中移除

这个算法的复杂度是`O(n)`，其中 n 代表密钥的长度。

让我们快速看一下实现:

```
public void delete(String word) {
    delete(root, word, 0);
}

private boolean delete(TrieNode current, String word, int index) {
    if (index == word.length()) {
        if (!current.isEndOfWord()) {
            return false;
        }
        current.setEndOfWord(false);
        return current.getChildren().isEmpty();
    }
    char ch = word.charAt(index);
    TrieNode node = current.getChildren().get(ch);
    if (node == null) {
        return false;
    }
    boolean shouldDeleteCurrentNode = delete(node, word, index + 1) && !node.isEndOfWord();

    if (shouldDeleteCurrentNode) {
        current.getChildren().remove(ch);
        return current.getChildren().isEmpty();
    }
    return false;
}
```

实际上:

```
@Test
void whenDeletingElements_ThenTreeDoesNotContainThoseElements() {
    Trie trie = createTrie();

    assertTrue(trie.containsNode("Programming"));

    trie.delete("Programming");
    assertFalse(trie.containsNode("Programming"));
}
```

## 4。结论

在本文中，我们看到了对数据结构及其最常见操作和实现的简要介绍。

本文中展示的例子的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220628062326/https://github.com/eugenp/tutorials/tree/master/data-structures)