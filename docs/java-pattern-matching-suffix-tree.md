# Java 中利用后缀树实现字符串的快速模式匹配

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pattern-matching-suffix-tree>

## 1.概观

在本教程中，我们将探索字符串模式匹配的概念，以及如何使它更快。然后，我们将浏览它在 Java 中的实现。

## 2.字符串的模式匹配

### 2.1.定义

在字符串中，模式匹配是在名为`text`的字符序列中检查名为`a pattern`的给定字符序列的过程。

当模式不是正则表达式时，模式匹配的基本期望是:

*   匹配应该是精确的，而不是部分的
*   结果应该包含所有匹配项，而不仅仅是第一个匹配项
*   结果应该包含文本中每个匹配的位置

### 2.2.搜索模式

让我们用一个例子来理解一个简单的模式匹配问题:

```java
Pattern:   NA
Text:      HAVANABANANA
Match1:    ----NA------
Match2:    --------NA--
Match3:    ----------NA
```

我们可以看到模式`NA`在文本中出现了三次。为了得到这个结果，我们可以考虑一次一个字符地在文本中滑动模式并检查匹配。

然而，这是一种时间复杂度为`O(p*t)`的强力方法，其中`p`是模式的长度，`t`是文本的长度。

假设我们要搜索不止一种模式。然后，时间复杂度也线性增加，因为每个模式将需要单独的迭代。

### 2.3.存储模式的数据结构

我们可以通过将模式存储在一个 [trie 数据结构](/web/20221206132420/https://www.baeldung.com/trie-java)中来缩短搜索时间，该数据结构以其快速的条目检索而闻名。

我们知道 trie 数据结构以树状结构存储字符串的字符。因此，对于两个字符串`{NA, NAB}`，我们将得到一个有两条路径的树:

[![prefix2](img/c870a60d261612aefd5432285b0d48b9.png)](/web/20221206132420/https://www.baeldung.com/wp-content/uploads/2020/03/prefix2.png)

创建一个 trie 使得沿着文本滑动一组模式并在一次迭代中检查匹配成为可能。

注意，我们使用了`$`字符来表示字符串的结尾。

### 2.4.存储文本的后缀 Trie 数据结构

另一方面，`suffix trie`是使用单个字符串的所有可能后缀构建的 trie 数据结构**。**

对于前面的例子`HAVANABANANA`，我们可以构造一个后缀 trie:

[![suffixtrie2](img/25ecd824a7c656ccfbcc41382f21a261.png)](/web/20221206132420/https://www.baeldung.com/wp-content/uploads/2020/03/suffixtrie2.png)

为文本创建后缀尝试，通常作为预处理步骤的一部分完成。之后，通过找到匹配模式序列的路径，可以快速完成模式搜索。

然而，已知后缀 trie 会消耗大量空间，因为字符串的每个字符都存储在一个边中。

我们将在下一节看到后缀 trie 的改进版本。

## 3.后缀树

后缀`tree`简单来说就是一个**压缩后缀`trie`** 。这意味着，通过连接边缘，我们可以存储一组字符，从而大大减少存储空间。

因此，我们可以为相同的文本`HAVANABANANA`创建一个后缀树:

[![suffixtree2](img/d856e596e80044ff5e41c766881e1097.png)](/web/20221206132420/https://www.baeldung.com/wp-content/uploads/2020/03/suffixtree2.png)

从根到叶的每一条路径都代表字符串`HAVANABANANA`的一个后缀。

后缀树**也存储后缀在叶节点**中的位置。例如，`BANANA$`是从第七个位置开始的后缀。因此，使用从零开始的编号，它的值将是 6。同样，`A->BANANA$`是另一个从位置五开始的后缀，正如我们在上图中看到的。

因此，客观地看，我们可以看到**当我们能够得到一条从根节点开始的路径，其边在位置上完全匹配给定的模式**时，就发生了模式匹配。

如果路径在叶节点结束，我们得到一个后缀匹配。否则，我们得到的只是一个子串匹配。例如，模式`NA`是`HAVANABANA[NA]`的后缀和`HAVA[NA]BANANA`的子串。

在下一节中，我们将看到如何用 Java 实现这个数据结构。

## 4.数据结构

让我们创建一个后缀树数据结构。我们需要两个域类。

首先，我们需要一个**类来表示树节点**。它需要存储树的边及其子节点。此外，当它是叶节点时，它需要存储后缀的位置值。

所以，让我们创建我们的`Node`类:

```java
public class Node {
    private String text;
    private List<Node> children;
    private int position;

    public Node(String word, int position) {
        this.text = word;
        this.position = position;
        this.children = new ArrayList<>();
    }

    // getters, setters, toString()
}
```

其次，我们需要一个**类来表示树并存储根节点**。它还需要存储生成后缀的完整文本。

因此，我们有一个`SuffixTree`类:

```java
public class SuffixTree {
    private static final String WORD_TERMINATION = "$";
    private static final int POSITION_UNDEFINED = -1;
    private Node root;
    private String fullText;

    public SuffixTree(String text) {
        root = new Node("", POSITION_UNDEFINED);
        fullText = text;
    }
}
```

## 5.添加数据的帮助器方法

在我们编写存储数据的核心逻辑之前，让我们添加几个助手方法。这些以后会证明是有用的。

让我们修改我们的`SuffixTree`类，添加一些构建树所需的方法。

### 5.1.添加子节点

首先，让我们有一个方法`addChildNode`到**向任何给定的父节点**添加一个新的子节点:

```java
private void addChildNode(Node parentNode, String text, int index) {
    parentNode.getChildren().add(new Node(text, index));
}
```

### 5.2.寻找两个字符串的最长公共前缀

其次，我们将编写一个简单的实用方法`getLongestCommonPrefix`到**找到两个字符串**的最长公共前缀:

```java
private String getLongestCommonPrefix(String str1, String str2) {
    int compareLength = Math.min(str1.length(), str2.length());
    for (int i = 0; i < compareLength; i++) {
        if (str1.charAt(i) != str2.charAt(i)) {
            return str1.substring(0, i);
        }
    }
    return str1.substring(0, compareLength);
}
```

### 5.3.拆分节点

第三，让我们有一个方法**从一个给定的父节点**中切割出一个子节点。在这个过程中，父节点的`text`值将被截断，右截断的字符串成为子节点的`text`值。此外，父节点的子节点将被转移到子节点。

从下图中我们可以看到，`ANA`后来被拆分为`A->NA.`，新的后缀`ABANANA$`可以被添加为`A->BANANA$`:

[![suffixtree2-splitnode](img/b4b94fff932c096832894cb53ce77830.png)](/web/20221206132420/https://www.baeldung.com/wp-content/uploads/2020/03/suffixtree2-splitnode.png)

简而言之，这是一种方便的方法，在插入新节点时会派上用场:

```java
private void splitNodeToParentAndChild(Node parentNode, String parentNewText, String childNewText) {
    Node childNode = new Node(childNewText, parentNode.getPosition());

    if (parentNode.getChildren().size() > 0) {
        while (parentNode.getChildren().size() > 0) {
            childNode.getChildren()
              .add(parentNode.getChildren().remove(0));
        }
    }

    parentNode.getChildren().add(childNode);
    parentNode.setText(parentNewText);
    parentNode.setPosition(POSITION_UNDEFINED);
}
```

## 6.遍历的助手方法

现在让我们创建遍历树的逻辑。我们将使用这种方法来构建树和搜索模式。

### 6.1.部分匹配与完全匹配

首先，让我们通过考虑由几个后缀组成的树来理解部分匹配和完全匹配的概念:

[![suffixtree2 traverse1](img/a1dd859b7f43fa2cfa5eaa9016e693fe.png)](/web/20221206132420/https://www.baeldung.com/wp-content/uploads/2020/03/suffixtree2-traverse1.png)

为了添加新的后缀`ANABANANA$`，我们检查是否存在可以修改或扩展以容纳新值的节点。为此，我们将新文本与所有节点进行比较，发现现有节点`[A]VANABANANA$`匹配第一个字符。所以，这就是我们需要修改的节点，这个匹配可以称为部分匹配。

另一方面，让我们考虑在同一棵树上搜索模式`VANE` 。我们知道它与前三个字符上的`[VAN]ABANANA$`部分匹配。如果所有四个字符都匹配，我们可以称之为完全匹配。**对于模式搜索，完全匹配是必要的**。

总而言之，我们将在构建树时使用部分匹配，在搜索模式时使用完全匹配。我们将使用一个标志`isAllowPartialMatch`来表示我们在每种情况下需要的匹配类型。

### 6.2.遍历树

现在，让我们编写遍历树的逻辑，只要我们能够在位置上匹配给定的模式:

```java
List<Node> getAllNodesInTraversePath(String pattern, Node startNode, boolean isAllowPartialMatch) {
    // ...
}
```

**我们将递归调用这个函数，并返回在路径**中找到的所有`nodes`的列表。

我们从比较模式文本的第一个字符和节点文本开始:

```java
if (pattern.charAt(0) == nodeText.charAt(0)) {
    // logic to handle remaining characters       
} 
```

对于部分匹配，如果模式短于或等于节点文本的长度，我们将当前节点添加到我们的`nodes`列表并在此处停止:

```java
if (isAllowPartialMatch && pattern.length() <= nodeText.length()) {
    nodes.add(currentNode);
    return nodes;
} 
```

然后，我们将该节点文本的剩余字符与模式的剩余字符进行比较。如果模式与节点文本的位置不匹配，我们就到此为止。当前节点包含在`nodes`列表中只是为了部分匹配:

```java
int compareLength = Math.min(nodeText.length(), pattern.length());
for (int j = 1; j < compareLength; j++) {
    if (pattern.charAt(j) != nodeText.charAt(j)) {
        if (isAllowPartialMatch) {
            nodes.add(currentNode);
        }
        return nodes;
    }
} 
```

如果模式匹配节点文本，我们将当前节点添加到我们的`nodes`列表中:

```java
nodes.add(currentNode);
```

但是如果模式的字符比节点文本多，我们需要检查子节点。为此，我们进行了一个递归调用，将`currentNode`作为开始节点，将`pattern`的剩余部分作为新模式。如果这个调用返回的节点列表不为空，它将被追加到我们的`nodes`列表中。如果对于完全匹配的场景它是空的，这意味着存在不匹配，因此为了表明这一点，我们添加了一个`null`项。而我们返回`nodes`:

```java
if (pattern.length() > compareLength) {
    List nodes2 = getAllNodesInTraversePath(pattern.substring(compareLength), currentNode, 
      isAllowPartialMatch);
    if (nodes2.size() > 0) {
        nodes.addAll(nodes2);
    } else if (!isAllowPartialMatch) {
        nodes.add(null);
    }
}
return nodes;
```

将所有这些放在一起，让我们创建`getAllNodesInTraversePath`:

```java
private List<Node> getAllNodesInTraversePath(String pattern, Node startNode, boolean isAllowPartialMatch) {
    List<Node> nodes = new ArrayList<>();
    for (int i = 0; i < startNode.getChildren().size(); i++) {
        Node currentNode = startNode.getChildren().get(i);
        String nodeText = currentNode.getText();
        if (pattern.charAt(0) == nodeText.charAt(0)) {
            if (isAllowPartialMatch && pattern.length() <= nodeText.length()) {
                nodes.add(currentNode);
                return nodes;
            }

            int compareLength = Math.min(nodeText.length(), pattern.length());
            for (int j = 1; j < compareLength; j++) {
                if (pattern.charAt(j) != nodeText.charAt(j)) {
                    if (isAllowPartialMatch) {
                        nodes.add(currentNode);
                    }
                    return nodes;
                }
            }

            nodes.add(currentNode);
            if (pattern.length() > compareLength) {
                List<Node> nodes2 = getAllNodesInTraversePath(pattern.substring(compareLength), 
                  currentNode, isAllowPartialMatch);
                if (nodes2.size() > 0) {
                    nodes.addAll(nodes2);
                } else if (!isAllowPartialMatch) {
                    nodes.add(null);
                }
            }
            return nodes;
        }
    }
    return nodes;
}
```

## 7.算法

### 7.1.存储数据

我们现在可以编写逻辑来存储数据。让我们从在`SuffixTree`类上定义一个新方法`addSuffix`开始:

```java
private void addSuffix(String suffix, int position) {
    // ...
}
```

呼叫者将提供后缀的位置。

接下来，让我们编写处理后缀的逻辑。**首先，我们需要检查是否存在与后缀**部分匹配的路径，至少通过调用我们的助手方法`getAllNodesInTraversePath`并将`isAllowPartialMatch`设置为`true`。如果路径不存在，我们可以将后缀作为子路径添加到根目录:

```java
List<Node> nodes = getAllNodesInTraversePath(pattern, root, true);
if (nodes.size() == 0) {
    addChildNode(root, suffix, position);
}
```

然而，**如果路径存在，就意味着我们需要修改一个现有的节点**。该节点将是`nodes` 列表中的最后一个节点。我们还需要弄清楚这个现有节点的新文本应该是什么。如果`nodes`列表只有一项，那么我们使用`suffix`。否则，我们从`suffix`中排除直到最后一个节点的公共前缀，以获得`newText`:

```java
Node lastNode = nodes.remove(nodes.size() - 1);
String newText = suffix;
if (nodes.size() > 0) {
    String existingSuffixUptoLastNode = nodes.stream()
        .map(a -> a.getText())
        .reduce("", String::concat);
    newText = newText.substring(existingSuffixUptoLastNode.length());
}
```

为了修改现有节点，让我们创建一个新方法`extendNode,`，我们将从`addSuffix`方法中停止的地方调用它。这个方法有两个主要的责任。一种是将现有节点分解为父节点和子节点，另一种是向新创建的父节点添加子节点。我们分解父节点只是为了让它成为所有子节点的公共节点。所以，我们的新方法已经准备好了:

```java
private void extendNode(Node node, String newText, int position) {
    String currentText = node.getText();
    String commonPrefix = getLongestCommonPrefix(currentText, newText);

    if (commonPrefix != currentText) {
        String parentText = currentText.substring(0, commonPrefix.length());
        String childText = currentText.substring(commonPrefix.length());
        splitNodeToParentAndChild(node, parentText, childText);
    }

    String remainingText = newText.substring(commonPrefix.length());
    addChildNode(node, remainingText, position);
}
```

我们现在可以回到添加后缀的方法，现在所有的逻辑都已就绪:

```java
private void addSuffix(String suffix, int position) {
    List<Node> nodes = getAllNodesInTraversePath(suffix, root, true);
    if (nodes.size() == 0) {
        addChildNode(root, suffix, position);
    } else {
        Node lastNode = nodes.remove(nodes.size() - 1);
        String newText = suffix;
        if (nodes.size() > 0) {
            String existingSuffixUptoLastNode = nodes.stream()
                .map(a -> a.getText())
                .reduce("", String::concat);
            newText = newText.substring(existingSuffixUptoLastNode.length());
        }
        extendNode(lastNode, newText, position);
    }
}
```

最后，让我们修改我们的`SuffixTree`构造函数来生成后缀，并调用我们之前的方法`addSuffix`来迭代地将它们添加到我们的数据结构中:

```java
public void SuffixTree(String text) {
    root = new Node("", POSITION_UNDEFINED);
    for (int i = 0; i < text.length(); i++) {
        addSuffix(text.substring(i) + WORD_TERMINATION, i);
    }
    fullText = text;
}
```

### 7.2.搜索数据

定义了存储数据的后缀树结构后，**我们现在可以编写执行搜索的逻辑**。

我们从在`SuffixTree`类上添加一个新方法`searchText`开始，将`pattern`作为输入进行搜索:

```java
public List<String> searchText(String pattern) {
    // ...
}
```

接下来，为了检查后缀树中是否存在`pattern`,我们调用我们的助手方法`getAllNodesInTraversePath` ,只为精确匹配设置标志，不像在添加数据时我们允许部分匹配:

```java
List<Node> nodes = getAllNodesInTraversePath(pattern, root, false);
```

然后我们得到与我们的模式匹配的节点列表。列表中的最后一个节点表示模式与之完全匹配的节点。因此，我们的下一步将是获取源自最后一个匹配节点的所有叶节点，并获取这些叶节点中存储的位置。

让我们创建一个单独的方法`getPositions`来做这件事。我们将检查给定的节点是否存储了后缀的最后一部分，以决定是否需要返回它的位置值。并且，我们将对给定节点的每个子节点递归地这样做:

```java
private List<Integer> getPositions(Node node) {
    List<Integer> positions = new ArrayList<>();
    if (node.getText().endsWith(WORD_TERMINATION)) {
        positions.add(node.getPosition());
    }
    for (int i = 0; i < node.getChildren().size(); i++) {
        positions.addAll(getPositions(node.getChildren().get(i)));
    }
    return positions;
}
```

一旦我们有了位置集，下一步就是用它来标记存储在后缀树中的文本模式。位置值指示后缀开始的位置，模式的长度指示从起点偏移多少个字符。应用这个逻辑，让我们创建一个简单的实用方法:

```java
private String markPatternInText(Integer startPosition, String pattern) {
    String matchingTextLHS = fullText.substring(0, startPosition);
    String matchingText = fullText.substring(startPosition, startPosition + pattern.length());
    String matchingTextRHS = fullText.substring(startPosition + pattern.length());
    return matchingTextLHS + "[" + matchingText + "]" + matchingTextRHS;
}
```

现在，我们已经准备好了支持方法。因此，**我们可以将它们添加到我们的搜索方法中，并完成逻辑**:

```java
public List<String> searchText(String pattern) {
    List<String> result = new ArrayList<>();
    List<Node> nodes = getAllNodesInTraversePath(pattern, root, false);

    if (nodes.size() > 0) {
        Node lastNode = nodes.get(nodes.size() - 1);
        if (lastNode != null) {
            List<Integer> positions = getPositions(lastNode);
            positions = positions.stream()
              .sorted()
              .collect(Collectors.toList());
            positions.forEach(m -> result.add((markPatternInText(m, pattern))));
        }
    }
    return result;
}
```

## 8.测试

现在我们已经有了算法，让我们测试它。

首先，让我们在`SuffixTree`中存储一个文本:

```java
SuffixTree suffixTree = new SuffixTree("havanabanana"); 
```

接下来，让我们搜索一个有效的模式`a`:

```java
List<String> matches = suffixTree.searchText("a");
matches.stream().forEach(m -> LOGGER.debug(m));
```

运行该代码得到了预期的六个匹配项:

```java
h[a]vanabanana
hav[a]nabanana
havan[a]banana
havanab[a]nana
havanaban[a]na
havanabanan[a]
```

接下来，让我们**搜索另一个有效的模式`nab`** :

```java
List<String> matches = suffixTree.searchText("nab");
matches.stream().forEach(m -> LOGGER.debug(m)); 
```

运行代码只得到一个预期的匹配:

```java
hava[nab]anana
```

最后，让我们**搜索一个无效模式`nag`** :

```java
List<String> matches = suffixTree.searchText("nag");
matches.stream().forEach(m -> LOGGER.debug(m));
```

运行代码不会给我们带来任何结果。我们看到匹配必须是精确的，而不是部分的。

因此，我们的模式搜索算法已经能够满足我们在本教程开始时提出的所有期望。

## 9.时间复杂度

当给定长度为`t`的文本构建后缀树时，**时间复杂度为`O(t)`** 。

那么，对于搜索长度为`p,` **的模式，时间复杂度为`O(p)`** 。回想一下，对于蛮力搜索，它是`O(p*t)`。因此，**模式搜索在文本**预处理后变得更快。

## 10.结论

在本文中，我们首先了解了三种数据结构的概念——trie、后缀 trie 和后缀树。然后我们看到了后缀树是如何被用来紧凑地存储后缀的。

后来，我们看到了如何使用后缀树来存储数据和执行模式搜索。

和往常一样，带有测试的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221206132420/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-searching)