# Java 中的深度优先搜索

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-depth-first-search>

## 1。概述

在本教程中，我们将探索 Java 中的深度优先搜索。

[深度优先搜索(DFS)](/web/20221206182338/https://www.baeldung.com/cs/depth-first-traversal-methods) 是一种用于树和[图数据结构](/web/20221206182338/https://www.baeldung.com/cs/graphs)的遍历算法。深度优先的**搜索在移动到探索另一个分支之前深入每个分支**。

在接下来的部分中，我们将首先看一下树的实现，然后是图。

要想知道如何在 Java 中实现这些结构，看看我们之前关于[二叉树](/web/20221206182338/https://www.baeldung.com/java-binary-tree)和[图](/web/20221206182338/https://www.baeldung.com/java-graphs)的教程。

## 2。树深度优先搜索

使用 DFS 遍历树有三种不同的顺序:

1.  前序遍历
2.  有序遍历
3.  后序遍历

### 2.1.前序遍历

**在前序遍历中，我们首先遍历根，然后是左右子树。**

我们可以使用递归简单地**实现前序遍历:**

*   访问`current`节点
*   遍历`left`子树
*   遍历`right`子树

```java
public void traversePreOrder(Node node) {
    if (node != null) {
        visit(node.value);
        traversePreOrder(node.left);
        traversePreOrder(node.right);
    }
}
```

我们也可以在没有递归的情况下实现前序遍历。

**要实现一个迭代的前序遍历，我们需要一个`Stack`** ，我们将经历以下步骤:

*   将`root`推入我们的`tack`
*   当`stack`不为空时
    *   弹出`current`节点
    *   访问`current`节点
    *   推`right`子，然后`left`子到`stack`

```java
public void traversePreOrderWithoutRecursion() {
    Stack<Node> stack = new Stack<Node>();
    Node current = root;
    stack.push(root);
    while(!stack.isEmpty()) {
        current = stack.pop();
        visit(current.value);

        if(current.right != null) {
            stack.push(current.right);
        }    
        if(current.left != null) {
            stack.push(current.left);
        }
    }        
}
```

### 2.2.有序遍历

对于有序遍历，**我们首先遍历左子树，然后是根，最后是右子树**。

二叉查找树的有序遍历意味着以值的升序遍历节点。

我们可以使用递归简单地实现有序遍历:

```java
public void traverseInOrder(Node node) {
    if (node != null) {
        traverseInOrder(node.left);
        visit(node.value);
        traverseInOrder(node.right);
    }
}
```

我们也可以**实现无递归的有序遍历**:

*   用`root`初始化`current`节点
*   而`current`不为空或者 s `tack`不为空
    *   继续将`left`子节点推到`stack,`上，直到到达`current`节点最左边的子节点
    *   从`stack`弹出并访问最左边的节点
    *   将`current`设置为弹出节点的`right` 子节点

```java
public void traverseInOrderWithoutRecursion() {
    Stack stack = new Stack<>();
    Node current = root;

    while (current != null || !stack.isEmpty()) {
        while (current != null) {
            stack.push(current);
            current = current.left;
        }

        Node top = stack.pop();
        visit(top.value);
        current = top.right;
    }
}
```

### 2.3.后序遍历

最后，在后序遍历中，**我们在遍历根树**之前先遍历左边和右边的子树。

我们可以遵循我们之前的**递归解**:

```java
public void traversePostOrder(Node node) {
    if (node != null) {
        traversePostOrder(node.left);
        traversePostOrder(node.right);
        visit(node.value);
    }
}
```

或者，我们也可以**实现无递归的后序遍历**:

*   在`s` `tack`中推送`root`节点
*   当 s `tack`不为空时
    *   检查我们是否已经遍历了左边和右边的子树
    *   如果没有，则将`right`子节点和`left`子节点推到`stack`上

```java
public void traversePostOrderWithoutRecursion() {
    Stack<Node> stack = new Stack<Node>();
    Node prev = root;
    Node current = root;
    stack.push(root);

    while (!stack.isEmpty()) {
        current = stack.peek();
        boolean hasChild = (current.left != null || current.right != null);
        boolean isPrevLastChild = (prev == current.right || 
          (prev == current.left && current.right == null));

        if (!hasChild || isPrevLastChild) {
            current = stack.pop();
            visit(current.value);
            prev = current;
        } else {
            if (current.right != null) {
                stack.push(current.right);
            }
            if (current.left != null) {
                stack.push(current.left);
            }
        }
    }   
}
```

## 3。图形深度优先搜索

图和树的主要区别在于**图可能包含循环**。

所以为了避免循环搜索，我们会在访问每个节点时对其进行标记。

我们将看到图 DFS 的两种实现，有递归的和没有递归的。

### 3.1.递归图 DFS

首先，让我们从简单的递归开始:

*   我们将从给定的节点开始
*   将`current`节点标记为已访问
*   访问`current`节点
*   遍历未访问的相邻顶点

```java
public void dfs(int start) {
    boolean[] isVisited = new boolean[adjVertices.size()];
    dfsRecursive(start, isVisited);
}

private void dfsRecursive(int current, boolean[] isVisited) {
    isVisited[current] = true;
    visit(current);
    for (int dest : adjVertices.get(current)) {
        if (!isVisited[dest])
            dfsRecursive(dest, isVisited);
    }
}
```

### 3.2。无递归图 DFS

我们也可以在没有递归的情况下实现图 DFS。我们将简单地使用一个`Stack`:

*   我们将从给定的节点开始
*   将`start`节点推入`stack`
*   趁着`Stack`不空
    *   将`current`节点标记为已访问
    *   访问`current`节点
    *   推动未访问的相邻顶点

```java
public void dfsWithoutRecursion(int start) {
    Stack<Integer> stack = new Stack<Integer>();
    boolean[] isVisited = new boolean[adjVertices.size()];
    stack.push(start);
    while (!stack.isEmpty()) {
        int current = stack.pop();
        if(!isVisited[current]){
            isVisited[current] = true;
            visit(current);
            for (int dest : adjVertices.get(current)) {
                if (!isVisited[dest])
                    stack.push(dest);
            }
    }
}
```

### 3.4.拓扑排序

图深度优先搜索有很多应用。DFS 的一个著名应用是拓扑排序。

有向图的拓扑排序是其顶点的线性排序，使得对于每条边，源节点在目的节点之前。

为了进行拓扑排序，我们需要对刚刚实现的 DFS 进行简单的添加:

*   我们需要将访问过的顶点保存在堆栈中，因为拓扑排序是访问过的顶点的逆序
*   我们仅在遍历了所有的邻居之后，才把被访问的节点推到堆栈中

```java
public List<Integer> topologicalSort(int start) {
    LinkedList<Integer> result = new LinkedList<Integer>();
    boolean[] isVisited = new boolean[adjVertices.size()];
    topologicalSortRecursive(start, isVisited, result);
    return result;
}

private void topologicalSortRecursive(int current, boolean[] isVisited, LinkedList<Integer> result) {
    isVisited[current] = true;
    for (int dest : adjVertices.get(current)) {
        if (!isVisited[dest])
            topologicalSortRecursive(dest, isVisited, result);
    }
    result.addFirst(current);
}
```

## 4。结论

在本文中，我们讨论了树和图数据结构的深度优先搜索。

完整的源代码可以在 [GitHub](https://web.archive.org/web/20221206182338/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-searching) 上获得。