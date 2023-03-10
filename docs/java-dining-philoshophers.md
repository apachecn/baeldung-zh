# Java 中的哲学家进餐问题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-dining-philoshophers>

## 1。简介

[哲学家进餐问题](/web/20220628082340/https://www.baeldung.com/cs/dining-philosophers)是用来**描述多线程环境中的同步问题并举例说明解决这些问题的技术**的经典问题之一。Dijkstra 首先阐述了这个问题，并提出了关于计算机访问磁带驱动器外围设备的问题。

目前的公式是由东尼·霍尔提出的，他也因发明了快速排序算法而闻名。在本文中，我们分析了这个众所周知的问题，并编写了一个流行的解决方案。

## 2。问题

[![dp0](img/4ef905b40a970816d8024d26d708ee3d.png)](/web/20220628082340/https://www.baeldung.com/wp-content/uploads/2017/05/dp0.png)

上图代表了这个问题。有五个沉默的哲学家(P1-P5)围坐在一张圆桌旁，吃着东西，思考着。

有五把叉子供他们分享(1-5)，为了能够吃东西，哲学家需要双手都有叉子。吃完后，他把它们都放下，然后它们可以被另一个重复同样循环的哲学家捡起来。

> 目标是想出一个方案/协议，帮助哲学家实现他们的目标，吃和思考，而不会饿死。

## 3。解决方案

最初的解决方案是让每个哲学家遵循以下协议:

```java
while(true) { 
    // Initially, thinking about life, universe, and everything
    think();

    // Take a break from thinking, hungry now
    pick_up_left_fork();
    pick_up_right_fork();
    eat();
    put_down_right_fork();
    put_down_left_fork();

    // Not hungry anymore. Back to thinking!
} 
```

正如上面伪代码所描述的，每个哲学家最初都在思考。过了一段时间后，哲学家饿了，想吃东西。

在这一点上，**他伸手去拿他两边的叉子，一旦他都拿到了，就开始吃东西**。一旦吃完，哲学家就放下叉子，这样他的邻居就可以吃了。

## 4。实施

我们将每个哲学家建模为实现`Runnable`接口的类，这样我们就可以将它们作为单独的线程运行。每个`Philosopher`都可以使用左右两侧的两个叉子:

```java
public class Philosopher implements Runnable {

    // The forks on either side of this Philosopher 
    private Object leftFork;
    private Object rightFork;

    public Philosopher(Object leftFork, Object rightFork) {
        this.leftFork = leftFork;
        this.rightFork = rightFork;
    }

    @Override
    public void run() {
        // Yet to populate this method
    }

}
```

我们也有一个方法来指导`Philosopher`执行一个动作——吃饭、思考，或者拿起叉子准备吃饭:

```java
public class Philosopher implements Runnable {

    // Member variables, standard constructor

    private void doAction(String action) throws InterruptedException {
        System.out.println(
          Thread.currentThread().getName() + " " + action);
        Thread.sleep(((int) (Math.random() * 100)));
    }

    // Rest of the methods written earlier
}
```

如上面的代码所示，每个动作都是通过将调用线程挂起一段随机的时间来模拟的，因此执行顺序不仅仅是由时间决定的。

现在，让我们实现一个`Philosopher`的核心逻辑。

为了模拟获取一个 fork，我们需要锁定它，这样就不会有两个`Philosopher`线程同时获取它。

为了实现这一点，我们使用`synchronized`关键字来获取 fork 对象的内部监视器，并防止其他线程也这样做。Java 中的关键字`synchronized`的指南可以在[这里](/web/20220628082340/https://www.baeldung.com/java-synchronized)找到。我们现在继续在`Philosopher`类中实现`run()`方法:

```java
public class Philosopher implements Runnable {

   // Member variables, methods defined earlier

    @Override
    public void run() {
        try {
            while (true) {

                // thinking
                doAction(System.nanoTime() + ": Thinking");
                synchronized (leftFork) {
                    doAction(
                      System.nanoTime() 
                        + ": Picked up left fork");
                    synchronized (rightFork) {
                        // eating
                        doAction(
                          System.nanoTime() 
                            + ": Picked up right fork - eating"); 

                        doAction(
                          System.nanoTime() 
                            + ": Put down right fork");
                    }

                    // Back to thinking
                    doAction(
                      System.nanoTime() 
                        + ": Put down left fork. Back to thinking");
                }
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return;
        }
    }
} 
```

这个方案恰恰实现了前面描述的那个:a `Philosopher`想了一会儿，然后决定吃饭。

在这之后，他拿到他左右两边的叉子，开始吃东西。吃完后，他放下叉子。我们还为每个动作添加了时间戳，这将有助于我们理解事件发生的顺序。

为了启动整个过程，我们编写了一个客户机，它创建 5 个`Philosophers`线程并启动它们:

```java
public class DiningPhilosophers {

    public static void main(String[] args) throws Exception {

        Philosopher[] philosophers = new Philosopher[5];
        Object[] forks = new Object[philosophers.length];

        for (int i = 0; i < forks.length; i++) {
            forks[i] = new Object();
        }

        for (int i = 0; i < philosophers.length; i++) {
            Object leftFork = forks[i];
            Object rightFork = forks[(i + 1) % forks.length];

            philosophers[i] = new Philosopher(leftFork, rightFork);

            Thread t 
              = new Thread(philosophers[i], "Philosopher " + (i + 1));
            t.start();
        }
    }
}
```

我们将每一个分叉都建模为通用的 Java 对象，并根据哲学家的数量来创建它们。我们向每个`Philosopher`传递他试图使用`synchronized`关键字锁定的左右分支。

运行这段代码会产生类似如下的输出。您的输出很可能与下面给出的不同，主要是因为在不同的时间间隔调用了`sleep()`方法:

```java
Philosopher 1 8038014601251: Thinking
Philosopher 2 8038014828862: Thinking
Philosopher 3 8038015066722: Thinking
Philosopher 4 8038015284511: Thinking
Philosopher 5 8038015468564: Thinking
Philosopher 1 8038016857288: Picked up left fork
Philosopher 1 8038022332758: Picked up right fork - eating
Philosopher 3 8038028886069: Picked up left fork
Philosopher 4 8038063952219: Picked up left fork
Philosopher 1 8038067505168: Put down right fork
Philosopher 2 8038089505264: Picked up left fork
Philosopher 1 8038089505264: Put down left fork. Back to thinking
Philosopher 5 8038111040317: Picked up left fork
```

所有的`Philosopher`最初开始思考，我们看到`Philosopher 1`开始拿起左右叉子，然后吃东西，并开始放下它们，之后“哲学家 5”拿起叉子。

## 5。问题与解决方案:[死锁](/web/20220628082340/https://www.baeldung.com/cs/os-deadlock)

虽然看起来上面的解决方案是正确的，但是有一个死锁的问题出现了。

> 死锁是这样一种情况，当每个进程都在等待获取某个其他进程持有的资源时，系统的进程就停止了。

我们可以通过运行上面的代码几次来确认这一点，并检查一些时候，代码只是挂起。以下是演示上述问题的示例输出:

```java
Philosopher 1 8487540546530: Thinking
Philosopher 2 8487542012975: Thinking
Philosopher 3 8487543057508: Thinking
Philosopher 4 8487543318428: Thinking
Philosopher 5 8487544590144: Thinking
Philosopher 3 8487589069046: Picked up left fork
Philosopher 1 8487596641267: Picked up left fork
Philosopher 5 8487597646086: Picked up left fork
Philosopher 4 8487617680958: Picked up left fork
Philosopher 2 8487631148853: Picked up left fork
```

在这种情况下，每个`Philosopher`已经获得了他的左叉，但是不能获得他的右叉，因为他的邻居已经获得了它。这种情况通常被称为**循环等待**，是导致死锁和阻止系统进程的条件之一。

### 6。解决死锁

正如我们在上面看到的，死锁的主要原因是循环等待条件，在这种情况下，每个进程都在等待一个被其他进程占用的资源。因此，为了避免死锁情况，我们需要确保循环等待条件被打破。有几种方法可以实现这一点，最简单的方法如下:

> 所有的哲学家都先伸手拿他们的左叉，除了一个先伸手拿他的右叉的人。

我们通过对代码进行相对较小的更改，在现有代码中实现了这一点:

```java
public class DiningPhilosophers {

    public static void main(String[] args) throws Exception {

        final Philosopher[] philosophers = new Philosopher[5];
        Object[] forks = new Object[philosophers.length];

        for (int i = 0; i < forks.length; i++) {
            forks[i] = new Object();
        }

        for (int i = 0; i < philosophers.length; i++) {
            Object leftFork = forks[i];
            Object rightFork = forks[(i + 1) % forks.length];

            if (i == philosophers.length - 1) {

                // The last philosopher picks up the right fork first
                philosophers[i] = new Philosopher(rightFork, leftFork); 
            } else {
                philosophers[i] = new Philosopher(leftFork, rightFork);
            }

            Thread t 
              = new Thread(philosophers[i], "Philosopher " + (i + 1));
            t.start();
        }
    }
} 
```

变化出现在上面代码的第 17-19 行，我们引入了一个条件，使得最后一个哲学家先用他的右叉，而不是左叉。这打破了循环等待条件，我们可以避免死锁。

下面的输出显示了一种情况，其中所有的`Philosopher`都有机会思考和吃饭，而不会导致死锁:

```java
Philosopher 1 88519839556188: Thinking
Philosopher 2 88519840186495: Thinking
Philosopher 3 88519840647695: Thinking
Philosopher 4 88519840870182: Thinking
Philosopher 5 88519840956443: Thinking
Philosopher 3 88519864404195: Picked up left fork
Philosopher 5 88519871990082: Picked up left fork
Philosopher 4 88519874059504: Picked up left fork
Philosopher 5 88519876989405: Picked up right fork - eating
Philosopher 2 88519935045524: Picked up left fork
Philosopher 5 88519951109805: Put down right fork
Philosopher 4 88519997119634: Picked up right fork - eating
Philosopher 5 88519997113229: Put down left fork. Back to thinking
Philosopher 5 88520011135846: Thinking
Philosopher 1 88520011129013: Picked up left fork
Philosopher 4 88520028194269: Put down right fork
Philosopher 4 88520057160194: Put down left fork. Back to thinking
Philosopher 3 88520067162257: Picked up right fork - eating
Philosopher 4 88520067158414: Thinking
Philosopher 3 88520160247801: Put down right fork
Philosopher 4 88520249049308: Picked up left fork
Philosopher 3 88520249119769: Put down left fork. Back to thinking 
```

可以通过多次运行代码来验证系统没有出现之前出现的死锁情况。

## 7。结论

在本文中，我们探讨了著名的哲学家进餐问题以及循环等待和死锁的概念。我们编写了一个导致死锁的简单解决方案，并做了一个简单的更改来打破循环等待并避免死锁。这只是一个开始，更复杂的解决方案确实存在。

这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628082340/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced)