# 用 Java 创建一个数独求解器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sudoku>

## 1。概述

在这篇文章中，我们将看看数独难题和用于解决它的算法。

接下来，我们将用 Java 实现解决方案。第一种解决方案是简单的暴力攻击。第二个将利用[舞蹈环节](https://web.archive.org/web/20220630142927/https://en.wikipedia.org/wiki/Dancing_Links)技术。

让我们记住，我们要关注的是算法，而不是 OOP 设计。

## 2。数独谜题

简而言之，数独是一个组合数字布局难题，其中 9 x 9 单元网格部分填充有 1 到 9 的数字。目标是用剩余的数字填充剩余的空白字段，这样每一行和每一列都只有一个数字。

此外，网格的每个 3 x 3 分区也不能有任何重复的数字。难度自然会随着每个棋盘上空白区域的数量而增加。

### 2.1。测试板

为了让我们的解决方案更有趣，也为了验证算法，我们将使用[“世界上最难的数独”](https://web.archive.org/web/20220630142927/https://www.telegraph.co.uk/news/science/science-news/9359579/Worlds-hardest-sudoku-can-you-crack-it.html)棋盘，它是:

```java
8 . . . . . . . . 
. . 3 6 . . . . . 
. 7 . . 9 . 2 . . 
. 5 . . . 7 . . . 
. . . . 4 5 7 . . 
. . . 1 . . . 3 . 
. . 1 . . . . 6 8 
. . 8 5 . . . 1 . 
. 9 . . . . 4 . .
```

### 2.2。已解板

并且，快速破坏解决方案-正确解决的难题将会给我们以下结果:

```java
8 1 2 7 5 3 6 4 9 
9 4 3 6 8 2 1 7 5 
6 7 5 4 9 1 2 8 3 
1 5 4 2 3 7 8 9 6 
3 6 9 8 4 5 7 2 1 
2 8 7 1 6 9 5 3 4 
5 2 1 9 7 4 3 6 8 
4 3 8 5 2 6 9 1 7 
7 9 6 3 1 8 4 5 2
```

## 3。回溯算法

### 3.1。简介

**[回溯算法](/web/20220630142927/https://www.baeldung.com/cs/backtracking-algorithms)试图通过测试每个单元格的有效解来解决这个难题。**

如果没有违反约束，算法会移动到下一个单元格，填充所有可能的解决方案，并重复所有检查。

如果有违规，那么它会增加单元格的值。一旦单元格的值达到 9，并且仍然存在违规，则算法会返回到前一个单元格，并增加该单元格的值。

它尝试所有可能的解决方案。

### 3.2。解决方案

首先，让我们把我们的棋盘定义为一个二维的整数数组。我们将使用 0 作为空单元格。

```java
int[][] board = {
  { 8, 0, 0, 0, 0, 0, 0, 0, 0 },
  { 0, 0, 3, 6, 0, 0, 0, 0, 0 },
  { 0, 7, 0, 0, 9, 0, 2, 0, 0 },
  { 0, 5, 0, 0, 0, 7, 0, 0, 0 },
  { 0, 0, 0, 0, 4, 5, 7, 0, 0 },
  { 0, 0, 0, 1, 0, 0, 0, 3, 0 },
  { 0, 0, 1, 0, 0, 0, 0, 6, 8 },
  { 0, 0, 8, 5, 0, 0, 0, 1, 0 },
  { 0, 9, 0, 0, 0, 0, 4, 0, 0 } 
};
```

让我们创建一个`solve()`方法，它将`board`作为输入参数，遍历行、列和值，测试每个单元格的有效解:

```java
private boolean solve(int[][] board) {
    for (int row = BOARD_START_INDEX; row < BOARD_SIZE; row++) {
        for (int column = BOARD_START_INDEX; column < BOARD_SIZE; column++) {
            if (board[row][column] == NO_VALUE) {
                for (int k = MIN_VALUE; k <= MAX_VALUE; k++) {
                    board[row][column] = k;
                    if (isValid(board, row, column) && solve(board)) {
                        return true;
                    }
                    board[row][column] = NO_VALUE;
                }
                return false;
            }
        }
    }
    return true;
}
```

我们需要的另一个方法是`isValid()`方法，它将检查数独约束，即检查行、列和 3 x 3 网格是否有效:

```java
private boolean isValid(int[][] board, int row, int column) {
    return (rowConstraint(board, row)
      && columnConstraint(board, column) 
      && subsectionConstraint(board, row, column));
}
```

这三张支票比较相似。首先，让我们从行检查开始:

```java
private boolean rowConstraint(int[][] board, int row) {
    boolean[] constraint = new boolean[BOARD_SIZE];
    return IntStream.range(BOARD_START_INDEX, BOARD_SIZE)
      .allMatch(column -> checkConstraint(board, row, constraint, column));
}
```

接下来，我们使用几乎相同的代码来验证列:

```java
private boolean columnConstraint(int[][] board, int column) {
    boolean[] constraint = new boolean[BOARD_SIZE];
    return IntStream.range(BOARD_START_INDEX, BOARD_SIZE)
      .allMatch(row -> checkConstraint(board, row, constraint, column));
}
```

此外，我们需要验证 3 x 3 子部分:

```java
private boolean subsectionConstraint(int[][] board, int row, int column) {
    boolean[] constraint = new boolean[BOARD_SIZE];
    int subsectionRowStart = (row / SUBSECTION_SIZE) * SUBSECTION_SIZE;
    int subsectionRowEnd = subsectionRowStart + SUBSECTION_SIZE;

    int subsectionColumnStart = (column / SUBSECTION_SIZE) * SUBSECTION_SIZE;
    int subsectionColumnEnd = subsectionColumnStart + SUBSECTION_SIZE;

    for (int r = subsectionRowStart; r < subsectionRowEnd; r++) {
        for (int c = subsectionColumnStart; c < subsectionColumnEnd; c++) {
            if (!checkConstraint(board, r, constraint, c)) return false;
        }
    }
    return true;
}
```

最后，我们需要一个`checkConstraint()`方法:

```java
boolean checkConstraint(
  int[][] board, 
  int row, 
  boolean[] constraint, 
  int column) {
    if (board[row][column] != NO_VALUE) {
        if (!constraint[board[row][column] - 1]) {
            constraint[board[row][column] - 1] = true;
        } else {
            return false;
        }
    }
    return true;
}
```

一旦完成，`isValid()`方法可以简单地返回`true`。

我们现在几乎准备好测试解决方案了。算法完成了。但是，它只返回`true`或`false`。

因此，为了直观地检查电路板，我们只需要打印出结果。显然，这不是算法的一部分。

```java
private void printBoard() {
    for (int row = BOARD_START_INDEX; row < BOARD_SIZE; row++) {
        for (int column = BOARD_START_INDEX; column < BOARD_SIZE; column++) {
            System.out.print(board[row][column] + " ");
        }
        System.out.println();
    }
}
```

我们已经成功实现了解决数独难题的回溯算法！

显然，由于算法天真地一遍又一遍地检查每个可能的组合(即使我们知道特定的解决方案是无效的)，因此还有改进的空间。

## 4。跳舞环节

### 4.1。精确覆盖

让我们看看另一个解决方案。数独可以描述为一个[精确覆盖](https://web.archive.org/web/20220630142927/https://en.wikipedia.org/wiki/Exact_cover)问题，可以用表示两个对象之间关系的关联矩阵来表示。

例如，如果我们取 1 到 7 的数和集合`S = {A, B, C, D, E, F}`，其中:

*   *A* = {1，4，7}
*   *B* = {1，4}
*   *C* = {4，5，7}
*   *D* = {3，5，6}
*   *E* = {2，3，6，7}
*   *F* = {2，7}

我们的目标是选择这样的子集，每个数字只出现一次，没有重复的，因此得名。

我们可以用矩阵来表示这个问题，其中列是数字，行是集合:

```java
 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 
A | 1 | 0 | 0 | 1 | 0 | 0 | 1 |
B | 1 | 0 | 0 | 1 | 0 | 0 | 0 |
C | 0 | 0 | 0 | 1 | 1 | 0 | 1 |
D | 0 | 0 | 1 | 0 | 1 | 1 | 0 |
E | 0 | 1 | 1 | 0 | 0 | 1 | 1 |
F | 0 | 1 | 0 | 0 | 0 | 0 | 1 |
```

子集合 S* = {B，D，F}是精确覆盖:

```java
 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 
B | 1 | 0 | 0 | 1 | 0 | 0 | 0 |
D | 0 | 0 | 1 | 0 | 1 | 1 | 0 |
F | 0 | 1 | 0 | 0 | 0 | 0 | 1 |
```

在所有选定的行中，每列恰好有一个 1。

### 4.2。算法 X

算法 X 是一个寻找精确覆盖问题所有解决方案的`“trial-and-error approach”`，即从我们的示例集合`S = {A, B, C, D, E, F}`开始，寻找子集合`S* = {B, D, F}.`

算法 X 的工作原理如下:

1.  如果矩阵 *A* 没有列，则当前部分解是有效解；
    终止成功，否则选择一列 *c* (确定性地)
2.  选择一行 *r* 使得*A*[r， *c*] = 1(不确定地，即尝试所有可能性)
3.  在部分解决方案中包含行 *r*
4.  对于每一列 *j* 使得*A*[r， *j*] = 1，对于每一行 *i* 使得 *A*[*i* ， *j*] = 1，
    从矩阵中删除行*I**A*和删除
5.  对简化矩阵 *A* 递归重复该算法

算法 X 的一个有效实现是唐纳德·克纳特博士提出的[舞动链接](https://web.archive.org/web/20220630142927/https://www.ocf.berkeley.edu/~jchu/publicportal/sudoku/0011047.pdf)算法(简称 DLX)。

下面的解决方案很大程度上受到了[这个](https://web.archive.org/web/20220630142927/https://github.com/rafalio/dancing-links-java) Java 实现的启发。

### 4.3。精确覆盖问题

首先，我们需要创建一个矩阵，将数独难题表示为一个精确的覆盖问题。该矩阵将具有 9^3 行，即每个可能的数(9 个数)的每个可能的位置(9 行×9 列)一行。

列将代表板(也是 9 x 9)乘以约束的数量。

我们已经定义了三个约束:

*   每一行每一种只有一个号码
*   每列每种类型只有一个数字
*   每个子部分每种类型只有一个数字

此外，还有隐含的第四个约束:

*   一个单元格中只能有一个数字

这总共给出了四个约束，因此在精确的覆盖矩阵中有 9×9×4 列:

```java
private static int BOARD_SIZE = 9;
private static int SUBSECTION_SIZE = 3;
private static int NO_VALUE = 0;
private static int CONSTRAINTS = 4;
private static int MIN_VALUE = 1;
private static int MAX_VALUE = 9;
private static int COVER_START_INDEX = 1;
```

```java
private int getIndex(int row, int column, int num) {
    return (row - 1) * BOARD_SIZE * BOARD_SIZE 
      + (column - 1) * BOARD_SIZE + (num - 1);
}
```

```java
private boolean[][] createExactCoverBoard() {
    boolean[][] coverBoard = new boolean
      [BOARD_SIZE * BOARD_SIZE * MAX_VALUE]
      [BOARD_SIZE * BOARD_SIZE * CONSTRAINTS];

    int hBase = 0;
    hBase = checkCellConstraint(coverBoard, hBase);
    hBase = checkRowConstraint(coverBoard, hBase);
    hBase = checkColumnConstraint(coverBoard, hBase);
    checkSubsectionConstraint(coverBoard, hBase);

    return coverBoard;
}

private int checkSubsectionConstraint(boolean[][] coverBoard, int hBase) {
    for (int row = COVER_START_INDEX; row <= BOARD_SIZE; row += SUBSECTION_SIZE) {
        for (int column = COVER_START_INDEX; column <= BOARD_SIZE; column += SUBSECTION_SIZE) {
            for (int n = COVER_START_INDEX; n <= BOARD_SIZE; n++, hBase++) {
                for (int rowDelta = 0; rowDelta < SUBSECTION_SIZE; rowDelta++) {
                    for (int columnDelta = 0; columnDelta < SUBSECTION_SIZE; columnDelta++) {
                        int index = getIndex(row + rowDelta, column + columnDelta, n);
                        coverBoard[index][hBase] = true;
                    }
                }
            }
        }
    }
    return hBase;
}

private int checkColumnConstraint(boolean[][] coverBoard, int hBase) {
    for (int column = COVER_START_INDEX; column <= BOARD_SIZE; c++) {
        for (int n = COVER_START_INDEX; n <= BOARD_SIZE; n++, hBase++) {
            for (int row = COVER_START_INDEX; row <= BOARD_SIZE; row++) {
                int index = getIndex(row, column, n);
                coverBoard[index][hBase] = true;
            }
        }
    }
    return hBase;
}

private int checkRowConstraint(boolean[][] coverBoard, int hBase) {
    for (int row = COVER_START_INDEX; row <= BOARD_SIZE; r++) {
        for (int n = COVER_START_INDEX; n <= BOARD_SIZE; n++, hBase++) {
            for (int column = COVER_START_INDEX; column <= BOARD_SIZE; column++) {
                int index = getIndex(row, column, n);
                coverBoard[index][hBase] = true;
            }
        }
    }
    return hBase;
}

private int checkCellConstraint(boolean[][] coverBoard, int hBase) {
    for (int row = COVER_START_INDEX; row <= BOARD_SIZE; row++) {
        for (int column = COVER_START_INDEX; column <= BOARD_SIZE; column++, hBase++) {
            for (int n = COVER_START_INDEX; n <= BOARD_SIZE; n++) {
                int index = getIndex(row, column, n);
                coverBoard[index][hBase] = true;
            }
        }
    }
    return hBase;
}
```

接下来，我们需要用我们的初始拼图布局更新新创建的板:

```java
private boolean[][] initializeExactCoverBoard(int[][] board) {
    boolean[][] coverBoard = createExactCoverBoard();
    for (int row = COVER_START_INDEX; row <= BOARD_SIZE; row++) {
        for (int column = COVER_START_INDEX; column <= BOARD_SIZE; column++) {
            int n = board[row - 1][column - 1];
            if (n != NO_VALUE) {
                for (int num = MIN_VALUE; num <= MAX_VALUE; num++) {
                    if (num != n) {
                        Arrays.fill(coverBoard[getIndex(row, column, num)], false);
                    }
                }
            }
        }
    }
    return coverBoard;
}
```

我们现在准备进入下一阶段。让我们创建两个将我们的细胞连接在一起的类。

### 4.4。跳舞节点

跳舞链接算法基于对节点的双向链接列表的以下操作的基本观察进行操作:

```java
node.prev.next = node.next
node.next.prev = node.prev
```

删除节点，同时:

```java
node.prev = node
node.next = node
```

恢复节点。

DLX 的每个节点都与左右上下的节点相连。

`DancingNode`类将拥有添加或删除节点所需的所有操作:

```java
class DancingNode {
    DancingNode L, R, U, D;
    ColumnNode C;

    DancingNode hookDown(DancingNode node) {
        assert (this.C == node.C);
        node.D = this.D;
        node.D.U = node;
        node.U = this;
        this.D = node;
        return node;
    }

    DancingNode hookRight(DancingNode node) {
        node.R = this.R;
        node.R.L = node;
        node.L = this;
        this.R = node;
        return node;
    }

    void unlinkLR() {
        this.L.R = this.R;
        this.R.L = this.L;
    }

    void relinkLR() {
        this.L.R = this.R.L = this;
    }

    void unlinkUD() {
        this.U.D = this.D;
        this.D.U = this.U;
    }

    void relinkUD() {
        this.U.D = this.D.U = this;
    }

    DancingNode() {
        L = R = U = D = this;
    }

    DancingNode(ColumnNode c) {
        this();
        C = c;
    }
}
```

### 4.5。列节点

`ColumnNode`类将列链接在一起:

```java
class ColumnNode extends DancingNode {
    int size;
    String name;

    ColumnNode(String n) {
        super();
        size = 0;
        name = n;
        C = this;
    }

    void cover() {
        unlinkLR();
        for (DancingNode i = this.D; i != this; i = i.D) {
            for (DancingNode j = i.R; j != i; j = j.R) {
                j.unlinkUD();
                j.C.size--;
            }
        }
    }

    void uncover() {
        for (DancingNode i = this.U; i != this; i = i.U) {
            for (DancingNode j = i.L; j != i; j = j.L) {
                j.C.size++;
                j.relinkUD();
            }
        }
        relinkLR();
    }
}
```

### 4.6。求解器

接下来，我们需要创建一个由我们的`DancingNode`和`ColumnNode`对象组成的网格:

```java
private ColumnNode makeDLXBoard(boolean[][] grid) {
    int COLS = grid[0].length;

    ColumnNode headerNode = new ColumnNode("header");
    List<ColumnNode> columnNodes = new ArrayList<>();

    for (int i = 0; i < COLS; i++) {
        ColumnNode n = new ColumnNode(Integer.toString(i));
        columnNodes.add(n);
        headerNode = (ColumnNode) headerNode.hookRight(n);
    }
    headerNode = headerNode.R.C;

    for (boolean[] aGrid : grid) {
        DancingNode prev = null;
        for (int j = 0; j < COLS; j++) {
            if (aGrid[j]) {
                ColumnNode col = columnNodes.get(j);
                DancingNode newNode = new DancingNode(col);
                if (prev == null) prev = newNode;
                col.U.hookDown(newNode);
                prev = prev.hookRight(newNode);
                col.size++;
            }
        }
    }

    headerNode.size = COLS;

    return headerNode;
}
```

我们将使用启发式搜索来查找列并返回矩阵的子集:

```java
private ColumnNode selectColumnNodeHeuristic() {
    int min = Integer.MAX_VALUE;
    ColumnNode ret = null;
    for (
      ColumnNode c = (ColumnNode) header.R; 
      c != header; 
      c = (ColumnNode) c.R) {
        if (c.size < min) {
            min = c.size;
            ret = c;
        }
    }
    return ret;
}
```

最后，我们可以递归地搜索答案:

```java
private void search(int k) {
    if (header.R == header) {
        handleSolution(answer);
    } else {
        ColumnNode c = selectColumnNodeHeuristic();
        c.cover();

        for (DancingNode r = c.D; r != c; r = r.D) {
            answer.add(r);

            for (DancingNode j = r.R; j != r; j = j.R) {
                j.C.cover();
            }

            search(k + 1);

            r = answer.remove(answer.size() - 1);
            c = r.C;

            for (DancingNode j = r.L; j != r; j = j.L) {
                j.C.uncover();
            }
        }
        c.uncover();
    }
}
```

如果没有更多的列，那么我们可以打印出已解决的数独板。

## 5。基准测试

我们可以通过在同一台计算机上运行这两种不同的算法来进行比较(这样我们可以避免组件、CPU 或 RAM 速度等方面的差异。).实际时间会因电脑而异。

然而，我们应该能够看到相对的结果，这将告诉我们哪个算法运行得更快。

回溯算法需要大约 250 毫秒来解决董事会。

如果我们将其与耗时约 50 毫秒的舞蹈链接进行比较，我们可以看到明显的赢家。在解决这个特殊的例子时，舞动链接的速度大约快了五倍。

## 6。结论

在本教程中，我们讨论了用核心 Java 解决数独难题的两种方法。回溯算法是一种强力算法，可以很容易地解决标准的 9×9 难题。

还讨论了稍微复杂一点的跳舞链接算法。两者都能在几秒钟内解决最难的难题。

最后，和往常一样，讨论中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220630142927/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-2)