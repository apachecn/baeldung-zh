# 用 Java 创建一个简单的“石头剪子布”游戏

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-rock-paper-scissors>

## 1.概观

在这个简短的教程中，我们将看到如何用 Java 创建一个简单的“石头剪子布”游戏。

## 2.创建我们的“石头剪子布”游戏

我们的游戏将允许玩家输入“石头”、“布”或“剪刀”作为每次移动的值。

首先，让我们为移动创建一个枚举:

```java
enum Move {
    ROCK("rock"),
    PAPER("paper"),
    SCISSORS("scissors");

    private String value;

    //...
}
```

然后，让我们创建一个生成随机整数并返回计算机移动的方法:

```java
private static String getComputerMove() {
    Random random = new Random();
    int randomNumber = random.nextInt(3);
    String computerMove = Move.values()[randomNumber].getValue();
    System.out.println("Computer move: " + computerMove);
    return computerMove;
}
```

以及检查玩家是否获胜的方法:

```java
private static boolean isPlayerWin(String playerMove, String computerMove) {
    return playerMove.equals(Move.ROCK.value) && computerMove.equals(Move.SCISSORS.value)
            || (playerMove.equals(Move.SCISSORS.value) && computerMove.equals(Move.PAPER.value))
            || (playerMove.equals(Move.PAPER.value) && computerMove.equals(Move.ROCK.value));
}
```

最后，我们将使用它们来形成一个完整的程序:

```java
Scanner scanner = new Scanner(System.in);
int wins = 0;
int losses = 0;

System.out.println("Welcome to Rock-Paper-Scissors! Please enter \"rock\", \"paper\", \"scissors\", or \"quit\" to exit.");

while (true) {
    System.out.println("-------------------------");
    System.out.print("Enter your move: ");
    String playerMove = scanner.nextLine();

    if (playerMove.equals("quit")) {
        System.out.println("You won " + wins + " times and lost " + losses + " times.");
        System.out.println("Thanks for playing! See you again.");
        break;
    }

    if (Arrays.stream(Move.values()).noneMatch(x -> x.getValue().equals(playerMove))) {
        System.out.println("Your move isn't valid!");
        continue;
    }

    String computerMove = getComputerMove();

    if (playerMove.equals(computerMove)) {
        System.out.println("It's a tie!");
    } else if (isPlayerWin(playerMove, computerMove)) {
        System.out.println("You won!");
        wins++;
    } else {
        System.out.println("You lost!");
        losses++;
    }
}
```

从上面可以看出，我们使用 Java [`Scanner`](/web/20220524070222/https://www.baeldung.com/java-scanner) 来读取用户输入的值。

让我们玩一会儿，看看输出结果:

```java
Welcome to Rock-Paper-Scissors! Please enter "rock", "paper", "scissors", or "quit" to exit.
-------------------------
Enter your move: rock
Computer move: scissors
You won!
-------------------------
Enter your move: paper
Computer move: paper
It's a tie!
-------------------------
Enter your move: quit
You won 1 times and lost 0 times.
Thanks for playing! See you again. 
```

## 3.结论

在这个快速教程中，我们学习了如何用 Java 创建一个简单的“石头剪子布”游戏。

和往常一样，本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524070222/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-2)