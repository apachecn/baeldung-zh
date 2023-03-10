# 用 Java 实现 2048 求解器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/2048-java-solver>

## 1。简介

**最近，我们看了一个解决游戏 2048 的[算法。](/web/20220630011049/https://www.baeldung.com/cs/2048-algorithm)**我们从理论的角度讨论了这一点，并没有任何实际的代码。

在这里，我们要用 Java 写一个实现。这将作为人类和电脑玩家一起玩，展示一个更好的游戏可以玩得多好。

## 2。初始设置

我们首先需要的是一个可以玩游戏的设置，看看进展如何。

这将为我们提供玩游戏所需的所有构造，并完全实现计算机播放器——反正它只放置随机的瓷砖。这给了我们实现一个“人类”玩家来玩游戏的机会。

### 2.1。游戏板

首先，我们需要一个游戏板。这是一个可以放置数字的单元格网格。

为了使一些事情更容易处理，**让我们从一个简单的单元格位置表示开始**。这实际上只是一对坐标的包装:

```java
public class Cell {
    private final int x;
    private final int y;

    // constructor, getters, and toString
}
```

**我们现在可以写一个类来代表董事会本身**。这将把值存储在一个简单的二维数组中，但是允许我们通过上面的`Cell`类来访问它们:

```java
public class Board {
    private final int[][] board;
    private final int score;

    public Board(int size) {
        this.board = new int[size][];
        this.score = 0;

        for (int x = 0; x < size; ++x) {
            this.board[x] = new int[size];
            for (int y = 0; y < size; ++y) {
                board[x][y] = 0;
            }
        }
    }

    public int getSize() {
        return board.length;
    }

    public int getScore() {
        return score;
    }

    public int getCell(Cell cell) {
        return board[cell.getX()][cell.getY()];
    }

    public boolean isEmpty(Cell cell) {
        return getCell(cell) == 0;
    }

    public List<Cell> emptyCells() {
        List<Cell> result = new ArrayList<>();
        for (int x = 0; x < board.length; ++x) {
            for (int y = 0; y < board[x].length; ++y) {
                Cell cell = new Cell(x, y);
                if (isEmpty(cell)) {
                    result.add(cell);
                }
            }
        }
        return result;
    }
}
```

这是一个不可变的类，它代表了一个棋盘，并让我们询问它来找出当前的状态。它还记录当前的分数，我们稍后会讲到。

### 2.2。电脑播放器和放置瓷砖

现在我们有了一个游戏板，我们希望能够玩它。我们首先想要的是电脑播放器，因为这是一个纯粹的随机播放器，而且以后会完全按照需要使用。

电脑玩家所做的不过是在一个格子里放一块瓷砖，所以我们需要一些方法在我们的棋盘上实现它。我们希望保持这一点不变，因此放置一个图块将在新状态下生成一个全新的棋盘。

首先，**我们想要一个构造函数来获取实际的棋盘状态**，而不是我们之前的那个构造一个空白的棋盘:

```java
private Board(int[][] board, int score) {
    this.score = score;
    this.board = new int[board.length][];

    for (int x = 0; x < board.length; ++x) {
        this.board[x] = Arrays.copyOf(board[x], board[x].length);
    }
}
```

这是`private`,因此它只能被同一个类中的其他方法使用。这有助于我们封装电路板。

接下来，我们将添加一个方法来放置瓷砖。这将返回一个全新的板，除了在给定的单元格中有给定的数字之外，它与当前的板完全相同:

```java
public Board placeTile(Cell cell, int number) {
    if (!isEmpty(cell)) {
        throw new IllegalArgumentException("That cell is not empty");
    }

    Board result = new Board(this.board, this.score);
    result.board[cell.getX()][cell.getY()] = number;
    return result;
}
```

最后，我们将编写一个新的类来代表一个计算机玩家。这将有一个单一的方法，该方法将获取当前的板并返回新的板:

```java
public class Computer {
    private final SecureRandom rng = new SecureRandom();

    public Board makeMove(Board input) {
        List<Cell> emptyCells = input.emptyCells();

        double numberToPlace = rng.nextDouble();
        int indexToPlace = rng.nextInt(emptyCells.size());
        Cell cellToPlace = emptyCells.get(indexToPlace);

        return input.placeTile(cellToPlace, numberToPlace >= 0.9 ? 4 : 2);
    }
}
```

这个函数从棋盘上获取每个空白单元格的列表，随机选择一个，然后在里面放一个数字。我们将随机决定在 10%的时间里将“4”放入单元格，而在其他 90%的时间里将“2”放入单元格。

### 2.2。一个“人类”玩家和移动瓷砖

接下来我们需要的是一个“人类”玩家。这不是最终目标，而是一个纯粹随机的玩家，每次移动时都会选择一个随机的方向来移动牌。这将成为我们打造最佳球员的基础。

首先，我们需要定义一系列可能的移动:

```java
public enum Move {
    UP,
    DOWN,
    LEFT,
    RIGHT
}
```

**接下来，我们需要扩充`Board`类，以支持通过在这些方向之一移动瓷砖来移动。**为了降低这里的复杂性，我们想旋转棋盘，这样我们就可以一直向同一个方向移动方块。

这意味着我们需要一种方法来调换和颠倒棋盘:

```java
private static int[][] transpose(int[][] input) {
    int[][] result = new int[input.length][];

    for (int x = 0; x < input.length; ++x) {
        result[x] = new int[input[0].length];
        for (int y = 0; y < input[0].length; ++y) {
            result[x][y] = input[y][x];
        }
    }

    return result;
}

private static int[][] reverse(int[][] input) {
    int[][] result = new int[input.length][];

    for (int x = 0; x < input.length; ++x) {
        result[x] = new int[input[0].length];
        for (int y = 0; y < input[0].length; ++y) {
            result[x][y] = input[x][input.length - y - 1];
        }
    }

    return result;
}
```

调换棋盘会交换所有的行和列，这样上边就变成了左边。反转棋盘只是简单地镜像它，使左边变成右边。

**接下来，我们向`Board`添加一个方法，在给定的方向上移动，并在新的状态下返回一个新的`Board`。**

我们首先制作一份董事会状态的副本，然后我们可以使用它:

```java
public Board move(Move move) {
    int newScore = 0;

    // Clone the board
    int[][] tiles = new int[this.board.length][];
    for (int x = 0; x < this.board.length; ++x) {
        tiles[x] = Arrays.copyOf(this.board[x], this.board[x].length);
    }
```

接下来，我们操纵我们的副本，以便我们总是将瓷砖向上移动:

```java
if (move == Move.LEFT || move == Move.RIGHT) {
    tiles = transpose(tiles);

}
if (move == Move.DOWN || move == Move.RIGHT) {
    tiles = reverse(tiles);
}
```

我们还需要另一个方块数组——这一次我们将把最终结果构建到其中——以及一个跟踪这次移动获得的新分数的跟踪器:

```java
int[][] result = new int[tiles.length][];
int newScore = 0;
```

现在我们已经准备好开始移动瓷砖，并且我们已经操纵了事物，所以我们总是在同一个方向上工作，我们可以开始了。

我们可以独立于其他列移动每一列。我们只需要迭代列并重复，从构建我们正在移动的瓦片的另一个副本开始。

这次我们将它们构建到一个`LinkedList`中，因为我们希望能够轻松地从中取出值。我们也只添加有数字的实际瓷砖，跳过空瓷砖。

这实现了我们的移动，但还没有合并瓷砖:

```java
for (int x = 0; x < tiles.length; ++x) {
    LinkedList<Integer> thisRow = new LinkedList<>();
    for (int y = 0; y < tiles[0].length; ++y) {
        if (tiles[x][y] > 0) {
            thisRow.add(tiles[x][y]);
        }
    }
```

接下来，我们需要合并瓷砖。**我们需要与上面的分开做；否则，我们会冒多次合并同一个图块的风险。**

这是通过从上面构建另一个`LinkedList`瓷砖来实现的，但这一次我们进行合并:

```java
LinkedList<Integer> newRow = new LinkedList<>();
while (thisRow.size() >= 2) {
    int first = thisRow.pop();
    int second = thisRow.peek();
    if (second == first) {
        int newNumber = first * 2;
        newRow.add(newNumber);
        newScore += newNumber;
        thisRow.pop();
    } else {
        newRow.add(first);
    }
}
newRow.addAll(thisRow);
```

在这里，我们也在计算这一步的新分数。这是合并后创建的图块的总和。

我们现在可以将它构建到结果数组中。一旦我们用完了列表中的图块，剩余的图块将被填充值“0”以表示它们是空白的:

```java
 result[x] = new int[tiles[0].length];
    for (int y = 0; y < tiles[0].length; ++y) {
        if (newRow.isEmpty()) {
            result[x][y] = 0;
        } else {
            result[x][y] = newRow.pop();
        }
    }
}
```

一旦我们完成移动瓷砖，我们需要操纵他们再次回到正确的旋转。这与我们之前做的完全相反:

```java
if (move == Move.DOWN || move == Move.RIGHT) {
    result = reverse(result);
}
if (move == Move.LEFT || move == Move.RIGHT) {
    result = transpose(result);
}
```

最后，我们可以用这组新的牌和新计算的分数构建并返回一个新的板:

```java
 return new Board(result, this.score + newScore);
}
```

我们现在可以写我们随机的“人类”玩家了。这无非是生成一个随机移动并调用上面的方法来进行移动:

```java
public class Human {
    private SecureRandom rng = new SecureRandom();

    public Board makeMove(Board input) {
        Move move = Move.values()[rng.nextInt(4)];
        return input.move(move);
    }
}
```

### 2.3。玩游戏

**我们有足够的组件来玩这个游戏，尽管不是很成功。**然而，很快我们将会改进`Human`职业的游戏方式，这将会让我们很容易地看到不同之处。

首先，我们需要一种打印游戏板的方法。

对于这个例子，我们只是要打印到控制台，所以`System.out.print`就足够了。对于一个真实的游戏，我们想要做更好的图形:

```java
private static void printBoard(Board board) {
    StringBuilder topLines = new StringBuilder();
    StringBuilder midLines = new StringBuilder();
    for (int x = 0; x < board.getSize(); ++x) {
        topLines.append("+--------");
        midLines.append("|        ");
    }
    topLines.append("+");
    midLines.append("|");

    for (int y = 0; y < board.getSize(); ++y) {
        System.out.println(topLines);
        System.out.println(midLines);
        for (int x = 0; x < board.getSize(); ++x) {
            Cell cell = new Cell(x, y);
            System.out.print("|");
            if (board.isEmpty(cell)) {
                System.out.print("        ");
            } else {
                StringBuilder output = new StringBuilder(Integer.toString(board.getCell(cell)));
                while (output.length() < 8) {
                    output.append(" ");
                    if (output.length() < 8) {
                        output.insert(0, " ");
                    }
                }
                System.out.print(output);
            }
        }
        System.out.println("|");
        System.out.println(midLines);
    }
    System.out.println(topLines);
    System.out.println("Score: " + board.getScore());
}
```

我们差不多准备好了。我们只需要把事情安排好。

这意味着创建棋盘、两个玩家，并让计算机进行两次初始移动，也就是说，在棋盘上放置两个随机数:

```java
Board board = new Board(4);
Computer computer = new Computer();
Human human = new Human();
for (int i = 0; i < 2; ++i) {
    board = computer.makeMove(board);
}
```

现在我们有了真正的游戏循环。**这将是人类和电脑玩家轮流的重复，只有当没有空单元格时才停止:**

```java
printBoard(board);
do {
    System.out.println("Human move");
    System.out.println("==========");
    board = human.makeMove(board);
    printBoard(board);

    System.out.println("Computer move");
    System.out.println("=============");
    board = computer.makeMove(board);
    printBoard(board);
} while (!board.emptyCells().isEmpty());

System.out.println("Final Score: " + board.getScore());
```

在这一点上，如果我们运行这个程序，我们会看到一个 2048 年的随机游戏正在进行。

## 3。实现 2048 播放器

一旦我们有了一个玩游戏的基础，我们就可以开始实现“人类”玩家，玩一个比随便选择一个方向更好的游戏。

### 3.1。模拟移动

我们在这里实现的算法基于 [Expectimax](https://web.archive.org/web/20220630011049/https://en.wikipedia.org/wiki/Expectiminimax) 算法。因此，**算法的核心是模拟每一个可能的移动，给每一个分配一个分数，并选择一个做得最好的。**

我们将大量使用 [Java 8 Streams](/web/20220630011049/https://www.baeldung.com/java-streams) 来帮助构建这段代码，原因我们将在后面看到。

我们将从重写我们的`Human`类中的`makeMove()`方法开始:

```java
public Board makeMove(Board input) {
    return Arrays.stream(Move.values())
      .map(input::move)
      .max(Comparator.comparingInt(board -> generateScore(board, 0)))
      .orElse(input);
}
```

**对于我们可以进入的每一个可能的方向，我们生成新的棋盘，然后开始计分算法**——传入这个棋盘，深度为 0。然后我们选择得分最高的棋步。

我们的`generateScore()`方法然后模拟每一个可能的计算机移动——也就是说，将“2”或“4”放入每个空单元格——然后看看接下来会发生什么:

```java
private int generateScore(Board board, int depth) {
    if (depth >= 3) {
        return calculateFinalScore(board);
    }
    return board.emptyCells().stream()
      .flatMap(cell -> Stream.of(new Pair<>(cell, 2), new Pair<>(cell, 4)))
      .mapToInt(move -> {
          Board newBoard = board.placeTile(move.getFirst(), move.getSecond());
          int boardScore = calculateScore(newBoard, depth + 1);
          return (int) (boardScore * (move.getSecond() == 2 ? 0.9 : 0.1));
      })
      .sum();
}
```

如果我们已经达到了我们的深度极限，那么我们将立即停下来，计算这块板有多好的最终分数；否则，我们继续模拟。

然后，我们的`calculateScore()`方法是我们模拟的继续，运行等式的人移动侧。

这与上面的`makeMove()`方法非常相似，但是我们返回的是正在进行的分数，而不是实际的棋盘:

```java
private int calculateScore(Board board, int depth) {
    return Arrays.stream(Move.values())
      .map(board::move)
      .mapToInt(newBoard -> generateScore(newBoard, depth))
      .max()
      .orElse(0);
}
```

### 3.2。给决赛板打分

我们现在的情况是，我们可以模拟人类和电脑玩家的来回移动，当我们模拟够了就停下来。我们需要能够为每个模拟分支中的最终板生成一个分数，以便我们可以看到哪个分支是我们想要追求的。

我们的评分是一个因素的组合，每个因素我们都将应用于棋盘上的每一行和每一列。这些都加在一起，然后返回总数。

因此，我们需要生成一个行和列的列表来进行评分:

```java
List<List<Integer>> rowsToScore = new ArrayList<>();
for (int i = 0; i < board.getSize(); ++i) {
    List<Integer> row = new ArrayList<>();
    List<Integer> col = new ArrayList<>();

    for (int j = 0; j < board.getSize(); ++j) {
        row.add(board.getCell(new Cell(i, j)));
        col.add(board.getCell(new Cell(j, i)));
    }

    rowsToScore.add(row);
    rowsToScore.add(col);
}
```

然后我们拿出我们建立的列表，给每个列表打分，并将分数相加。这是我们将要填写的占位符:

```java
return rowsToScore.stream()
    .mapToInt(row -> {
        int score = 0;
        return score;
    })
    .sum();
```

最后，我们需要实际生成我们的分数。**这属于上述λ，是几个不同的因素共同作用的结果**:

*   每行的固定分数
*   该行中每个数字的总和
*   行中所有可能的合并
*   行中的每个空单元格
*   行的单调性。这表示该行按数字升序排列的数量。

在我们计算分数之前，我们需要建立一些额外的数据。

首先，我们需要一个删除了空白单元格的数字列表:

```java
List<Integer> preMerged = row.stream()
  .filter(value -> value != 0)
  .collect(Collectors.toList());
```

然后，我们可以根据这个新列表进行一些计数，给出具有相同编号的相邻单元格的数量，严格按照升序编号和降序编号:

```java
int numMerges = 0;
int monotonicityLeft = 0;
int monotonicityRight = 0;
for (int i = 0; i < preMerged.size() - 1; ++i) {
    Integer first = preMerged.get(i);
    Integer second = preMerged.get(i + 1);
    if (first.equals(second)) {
        ++numMerges;
    } else if (first > second) {
        monotonicityLeft += first - second;
    } else {
        monotonicityRight += second - first;
    }
}
```

**现在我们可以计算这一行的分数:**

```java
int score = 1000;
score += 250 * row.stream().filter(value -> value == 0).count();
score += 750 * numMerges;
score -= 10 * row.stream().mapToInt(value -> value).sum();
score -= 50 * Math.min(monotonicityLeft, monotonicityRight);
return score;
```

这里选择的数字相对来说是任意的。不同的数字会对游戏的好坏产生影响，在我们如何玩的过程中优先考虑不同的因素。

## 4。对算法的改进

到目前为止，我们所做的是有效的，我们可以看到它玩得很好，但速度很慢。每个人移动大约需要 1 分钟。我们可以做得更好。

### 4.1。并行处理

我们能做的最明显的事情就是并行工作。这是使用 Java 流的一个巨大好处——我们可以通过向每个流添加一条语句来并行完成这项工作。

仅这一项改变就能让我们每次移动的时间减少到 20 秒左右。

### 4.2。修剪不可打的树枝

下一件我们可以做的事是修剪掉任何不适合比赛的树枝。也就是说，任何时候人的移动都会导致棋盘不变。几乎可以肯定，这些分支将导致更糟糕的结果——它们实际上给了计算机自由行动的空间——但它们耗费了我们的处理时间来追求它们。

为此，我们需要在我们的`Board`上实现一个 equals 方法，以便我们可以比较它们:

```java
@Override
public boolean equals(Object o) {
    if (this == o) {
        return true;
    }
    if (o == null || getClass() != o.getClass()) {
        return false;
    }
    Board board1 = (Board) o;
    return Arrays.deepEquals(board, board1.board);
}
```

然后，我们可以在流管道中添加一些过滤器，停止处理任何没有改变的东西。

```java
return Arrays.stream(Move.values())
    .parallel()
    .map(board::move)
    .filter(moved -> !moved.equals(board))
    ........
```

这对游戏的早期部分影响很小——当填充的格子很少时，可以修剪的走法也很少。然而，后来，这开始产生更大的影响，将移动时间减少到只有几秒钟。

## 5。总结

在这里，我们建立了一个玩游戏 2048 的框架。然后，我们写了一个解算器，这样我们可以玩得更好。这里看到的所有例子都可以在 GitHub 的[上找到。](https://web.archive.org/web/20220630011049/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-6)

为什么不尝试改变规则，看看他们如何影响游戏。