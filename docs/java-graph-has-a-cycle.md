# 检查 Java 图是否有循环

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-graph-has-a-cycle>

## 1.概观

在这个快速教程中，我们将学习如何在一个给定的有向图中检测一个循环。

## 2.图表示

对于本教程，我们将坚持使用邻接表[图表示法](/web/20220628093124/https://www.baeldung.com/java-graphs#graph_representations)。

首先，让我们从在 Java 中定义一个`Vertex`开始:

```java
public class Vertex {

    private String label;
    private boolean beingVisited;
    private boolean visited;
    private List<Vertex> adjacencyList;

    public Vertex(String label) {
        this.label = label;
        this.adjacencyList = new ArrayList<>();
    }

    public void addNeighbor(Vertex adjacent) {
        this.adjacencyList.add(adjacent);
    }
    //getters and setters
}
```

**这里，顶点 `v`的`adjacencyList`保存了与`v`相邻的所有顶点的列表。**`addNeighbor()`方法向`v`的邻接表中添加一个相邻顶点。

我们还定义了两个`boolean`参数， **`beingVisited` 和 `visited,`，它们代表节点是当前正在被访问还是已经被访问过。**

一个图可以被认为是一组通过边连接的顶点或节点。

所以，现在让我们快速地用 Java 表示一个`Graph`:

```java
public class Graph {

    private List<Vertex> vertices;

    public Graph() {
        this.vertices = new ArrayList<>();
    }

    public void addVertex(Vertex vertex) {
        this.vertices.add(vertex);
    }

    public void addEdge(Vertex from, Vertex to) {
        from.addNeighbor(to);
    }

   // ...
}
```

我们将使用`addVertex()`和 `addEdge()`方法在我们的图中添加新的顶点和边。

## 3.循环检测

为了检测有向图中的循环，**我们将使用`DFS`遍历的变体:**

*   选取一个未访问的顶点`v`并将其状态标记为`beingVisited`
*   对于`v,` 的每个相邻顶点`u` ,检查:
    *   如果`u`已经处于`beingVisited`状态，这显然意味着**存在一个后沿，因此已经检测到一个周期**
    *   如果`u`仍处于未访问状态，我们将以深度优先的方式递归访问 `u`
*   将顶点`v`的`beingVisited`标志更新为`false`，将其`visited`标志更新为`true`

注意**我们图中的所有顶点最初都处于未访问状态，因为它们的`beingVisited`和`visited`标志都用`false`初始化。**

现在让我们看看我们的 Java 解决方案:

```java
public boolean hasCycle(Vertex sourceVertex) {
    sourceVertex.setBeingVisited(true);

    for (Vertex neighbor : sourceVertex.getAdjacencyList()) {
        if (neighbor.isBeingVisited()) {
            // backward edge exists
            return true;
        } else if (!neighbor.isVisited() && hasCycle(neighbor)) {
            return true;
        }
    }

    sourceVertex.setBeingVisited(false);
    sourceVertex.setVisited(true);
    return false;
}
```

我们可以用图中的任何一个顶点作为源点或起点。

**对于一个断开的图，我们必须添加一个额外的包装方法:**

```java
public boolean hasCycle() {
    for (Vertex vertex : vertices) {
        if (!vertex.isVisited() && hasCycle(vertex)) {
            return true;
        }
    }
    return false;
}
```

这是为了确保我们为了检测一个循环而访问一个不连通的图的每一个部分。

## 4.实施测试

让我们考虑下面的循环有向图:

[![DirectedGraph](img/ccfacce6eb0eb26cf7bdb7f94bd1ded9.png)](/web/20220628093124/https://www.baeldung.com/wp-content/uploads/2019/06/DirectedGraph.png)

我们可以快速编写一个 JUnit 来验证我们针对该图的`hasCycle()`方法:

```java
@Test
public void givenGraph_whenCycleExists_thenReturnTrue() {

    Vertex vertexA = new Vertex("A");
    Vertex vertexB = new Vertex("B");
    Vertex vertexC = new Vertex("C")
    Vertex vertexD = new Vertex("D");

    Graph graph = new Graph();
    graph.addVertex(vertexA);
    graph.addVertex(vertexB);
    graph.addVertex(vertexC);
    graph.addVertex(vertexD);

    graph.addEdge(vertexA, vertexB);
    graph.addEdge(vertexB, vertexC);
    graph.addEdge(vertexC, vertexA);
    graph.addEdge(vertexD, vertexC);

    assertTrue(graph.hasCycle());

}
```

这里，我们的`hasCycle()`方法返回了`true`,表示我们的图是循环的。

## 5.结论

在本教程中，我们学习了如何在 Java 中检查给定的有向图中是否存在循环。

像往常一样，带有示例的代码实现可以在 Github 的[上找到。](https://web.archive.org/web/20220628093124/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-3)