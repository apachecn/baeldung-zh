# Java 的多维数组列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-multi-dimensional-arraylist>

## 1.概观

创建一个多维的`ArrayList`经常出现在编程过程中。在许多情况下，需要创建二维的`ArrayList`或三维的`ArrayList`。

在本教程中，我们将讨论如何在 Java 中创建多维`ArrayList`。

## 2.二维 [`ArrayList`](/web/20220714092049/https://www.baeldung.com/java-arraylist)

假设我们想要表示一个有 3 个顶点的图,编号为 0 到 2。此外，让我们假设图中有 3 条边(0，1)，(1，2)和(2，0)，其中一对顶点代表一条边。

**我们可以通过创建并填充`ArrayList` s.** 的`ArrayList`来表示二维`ArrayList`中的边

首先，让我们创建一个新的二维`ArrayList`:

```java
int vertexCount = 3;
ArrayList<ArrayList<Integer>> graph = new ArrayList<>(vertexCount);
```

接下来，我们将用另一个`ArrayList`初始化`ArrayList`的每个元素:

```java
for(int i=0; i < vertexCount; i++) {
    graph.add(new ArrayList());
}
```

最后，我们可以将所有的边(0，1)、(1，2)和(2，0)添加到我们的 2d`ArrayList`:

```java
graph.get(0).add(1);
graph.get(1).add(2);
graph.get(2).add(0);
```

让我们也假设我们的图不是一个有向图。因此，我们还需要将边(1，0)、(2，1)和(0，2)添加到我们的 2d`ArrayList`:

```java
graph.get(1).add(0);
graph.get(2).add(1);
graph.get(0).add(2);
```

然后，为了遍历整个图，我们可以使用双 for 循环:

```java
int vertexCount = graph.size();
for (int i = 0; i < vertexCount; i++) {
    int edgeCount = graph.get(i).size();
    for (int j = 0; j < edgeCount; j++) {
        Integer startVertex = i;
        Integer endVertex = graph.get(i).get(j);
        System.out.printf("Vertex %d is connected to vertex %d%n", startVertex, endVertex);
    }
}
```

## 3.三维`ArrayList`

在前一节中，我们创建了一个二维的`ArrayList.`遵循同样的逻辑，让我们创建一个三维的`ArrayList`:

让我们假设我们想要表示一个三维空间。**因此，这个三维空间中的每个点将由三个坐标来表示，比如 X、Y 和 z。**

除此之外，让我们想象每一个点都有一种颜色，红色、绿色、蓝色或黄色。现在，每个点(X，Y，Z)及其颜色都可以用一个三维的`ArrayList.`来表示

为了简单起见，让我们假设我们正在创建一个(2 x 2 x 2)的三维空间。它将有八个点:(0，0，0)，(0，0，1)，(0，1，0)，(0，1，1)，(1，0，0)，(1，0，1)，(1，1，1，0)，和(1，1，1)。

让我们首先初始化变量和 3d`ArrayList`:

```java
int x_axis_length = 2;
int y_axis_length = 2;
int z_axis_length = 2;	
ArrayList<ArrayList<ArrayList<String>>> space = new ArrayList<>(x_axis_length);
```

然后，让我们用`ArrayList<ArrayList<String>>`初始化`ArrayList`的每个元素:

```java
for (int i = 0; i < x_axis_length; i++) {
    space.add(new ArrayList<ArrayList<String>>(y_axis_length));
    for (int j = 0; j < y_axis_length; j++) {
        space.get(i).add(new ArrayList<String>(z_axis_length));
    }
}
```

现在，我们可以给空间中的点添加颜色。让我们为点(0，0，0)和(0，0，1)添加红色:

```java
space.get(0).get(0).add(0,"Red");
space.get(0).get(0).add(1,"Red");
```

然后，让我们为点(0，1，0)和(0，1，1)设置蓝色:

```java
space.get(0).get(1).add(0,"Blue");
space.get(0).get(1).add(1,"Blue");
```

类似地，我们可以继续为其他颜色填充空间中的点。

请注意，坐标为(I，j，k)的点的颜色信息存储在以下 3-D `ArrayList`元素中:

```java
space.get(i).get(j).get(k) 
```

正如我们在这个例子中看到的，`space`变量是一个`ArrayList`。**同样，这个`ArrayList`的每一个元素都是一个二维的`ArrayList`** (类似于我们在第二节看到的)。

**注意，我们的`space` `ArrayList`中的元素的索引代表 X 坐标，而出现在该索引处的每个二维`ArrayList`代表(Y，Z)坐标。**

## 4.结论

在本文中，我们讨论了如何在 Java 中创建多维`ArrayList`。我们看到了如何用一个二维的`ArrayList`来表示一个图形。此外，我们还探索了如何使用三维`ArrayList`来表示三维空间坐标。

第一次，我们使用了一个`ArrayList,`的`ArrayList`，而第二次，我们使用了一个二维`ArrayList`的`ArrayList`。类似地，**创建一个 N 维的`ArrayList,`我们可以扩展同样的概念。**

本教程的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220714092049/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-array-list)