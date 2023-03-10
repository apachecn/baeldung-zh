# Java 中的循环屏障

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cyclic-barrier>

## 1。简介

`CyclicBarriers`是 Java 5 引入的同步结构，作为`java.util.concurrent`包的一部分。

在本文中，我们将在并发场景中探索这种实现。

## 2。Java 并发-同步器

`java.util.concurrent`包包含几个帮助管理一组相互协作的线程的类。其中包括:

*   `CyclicBarrier`
*   `Phaser`
*   `CountDownLatch`
*   `Exchanger`
*   `Semaphore`
*   `SynchronousQueue`

这些类为线程间常见的交互模式提供了现成的功能。

如果我们有一组相互通信的线程，并且类似于一个公共模式，**，我们可以简单地重用适当的库类(也称为`Synchronizers`)，而不是试图使用一组锁和条件对象**以及`synchronized`关键字来提出一个定制的方案。

让我们把注意力放在未来的`CyclicBarrier` 上。

## 3。`CyclicBarrier`

`CyclicBarrier`是一个同步器，允许一组线程等待彼此到达一个共同的执行点，也称为`barrier`。

> 在程序中，我们有固定数量的线程，它们必须等待彼此到达一个公共点，然后才能继续执行。

**这个屏障被称为`cyclic` ，因为它可以在等待线程被释放后被重用。**

## 4。用途

`CyclicBarrier`的构造函数很简单。它使用一个整数来表示需要调用 barrier 实例上的`await()`方法的线程数量，以表示到达了公共执行点:

```java
public CyclicBarrier(int parties)
```

需要同步它们的执行的线程也被称为`parties`，调用`await()` 方法是我们可以注册某个线程已经到达障碍点的方式。

此调用是同步的，调用此方法的线程会暂停执行，直到指定数量的线程在屏障上调用了相同的方法。**这种所需线程数已调用`await()`的情况，称为`tripping the barrier`。**

或者，我们可以将第二个参数传递给构造函数，这是一个`Runnable`实例。这个逻辑将由触发屏障的最后一个线程运行:

```java
public CyclicBarrier(int parties, Runnable barrierAction)
```

## 5。实施

为了了解`CyclicBarrier`的实际情况，让我们考虑以下场景:

有一个操作，固定数量的线程执行并将相应的结果存储在一个列表中。当所有线程完成执行它们的动作时，其中一个线程(通常是最后一个触发屏障的线程)开始处理每个线程获取的数据。

让我们实现发生所有动作的主类:

```java
public class CyclicBarrierDemo {

    private CyclicBarrier cyclicBarrier;
    private List<List<Integer>> partialResults
     = Collections.synchronizedList(new ArrayList<>());
    private Random random = new Random();
    private int NUM_PARTIAL_RESULTS;
    private int NUM_WORKERS;

    // ...
}
```

这个类非常简单——`NUM_WORKERS`是将要执行的线程数量,`NUM_PARTIAL_RESULTS`是每个工作线程将要产生的结果数量。

最后，我们有一个`partialResults`列表，它将存储每个工作线程的结果。请注意，这个列表是一个`SynchronizedList`，因为多个线程将同时向它写入，而`add()`方法在普通的`ArrayList`上不是线程安全的。

现在让我们实现每个工作线程的逻辑:

```java
public class CyclicBarrierDemo {

    // ...

    class NumberCruncherThread implements Runnable {

        @Override
        public void run() {
            String thisThreadName = Thread.currentThread().getName();
            List<Integer> partialResult = new ArrayList<>();

            // Crunch some numbers and store the partial result
            for (int i = 0; i < NUM_PARTIAL_RESULTS; i++) {    
                Integer num = random.nextInt(10);
                System.out.println(thisThreadName
                  + ": Crunching some numbers! Final result - " + num);
                partialResult.add(num);
            }

            partialResults.add(partialResult);
            try {
                System.out.println(thisThreadName 
                  + " waiting for others to reach barrier.");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                // ...
            } catch (BrokenBarrierException e) {
                // ...
            }
        }
    }

}
```

我们现在将实现当障碍被触发时运行的逻辑。

为了简单起见，我们只需将部分结果列表中的所有数字相加:

```java
public class CyclicBarrierDemo {

    // ...

    class AggregatorThread implements Runnable {

        @Override
        public void run() {

            String thisThreadName = Thread.currentThread().getName();

            System.out.println(
              thisThreadName + ": Computing sum of " + NUM_WORKERS 
              + " workers, having " + NUM_PARTIAL_RESULTS + " results each.");
            int sum = 0;

            for (List<Integer> threadResult : partialResults) {
                System.out.print("Adding ");
                for (Integer partialResult : threadResult) {
                    System.out.print(partialResult+" ");
                    sum += partialResult;
                }
                System.out.println();
            }
            System.out.println(thisThreadName + ": Final result = " + sum);
        }
    }
}
```

最后一步是构建`CyclicBarrier`并用`main()`方法开始:

```java
public class CyclicBarrierDemo {

    // Previous code

    public void runSimulation(int numWorkers, int numberOfPartialResults) {
        NUM_PARTIAL_RESULTS = numberOfPartialResults;
        NUM_WORKERS = numWorkers;

        cyclicBarrier = new CyclicBarrier(NUM_WORKERS, new AggregatorThread());

        System.out.println("Spawning " + NUM_WORKERS
          + " worker threads to compute "
          + NUM_PARTIAL_RESULTS + " partial results each");

        for (int i = 0; i < NUM_WORKERS; i++) {
            Thread worker = new Thread(new NumberCruncherThread());
            worker.setName("Thread " + i);
            worker.start();
        }
    }

    public static void main(String[] args) {
        CyclicBarrierDemo demo = new CyclicBarrierDemo();
        demo.runSimulation(5, 3);
    }
} 
```

在上面的代码中，我们用 5 个线程初始化了循环屏障，每个线程产生 3 个整数作为计算的一部分，并将它们存储在结果列表中。

一旦障碍被触发，触发障碍的最后一个线程执行 AggregatorThread 中指定的逻辑，即–将线程产生的所有数字相加。

## 6。结果

下面是上述程序的一次执行的输出——每次执行可能会产生不同的结果，因为线程可能以不同的顺序产生:

```java
Spawning 5 worker threads to compute 3 partial results each
Thread 0: Crunching some numbers! Final result - 6
Thread 0: Crunching some numbers! Final result - 2
Thread 0: Crunching some numbers! Final result - 2
Thread 0 waiting for others to reach barrier.
Thread 1: Crunching some numbers! Final result - 2
Thread 1: Crunching some numbers! Final result - 0
Thread 1: Crunching some numbers! Final result - 5
Thread 1 waiting for others to reach barrier.
Thread 3: Crunching some numbers! Final result - 6
Thread 3: Crunching some numbers! Final result - 4
Thread 3: Crunching some numbers! Final result - 0
Thread 3 waiting for others to reach barrier.
Thread 2: Crunching some numbers! Final result - 1
Thread 2: Crunching some numbers! Final result - 1
Thread 2: Crunching some numbers! Final result - 0
Thread 2 waiting for others to reach barrier.
Thread 4: Crunching some numbers! Final result - 9
Thread 4: Crunching some numbers! Final result - 3
Thread 4: Crunching some numbers! Final result - 5
Thread 4 waiting for others to reach barrier.
Thread 4: Computing final sum of 5 workers, having 3 results each.
Adding 6 2 2 
Adding 2 0 5 
Adding 6 4 0 
Adding 1 1 0 
Adding 9 3 5 
Thread 4: Final result = 46 
```

如上面的输出所示，`Thread 4`是触发障碍的那个，并且还执行最终的聚合逻辑。线程实际上也没有必要按照上面的例子所示的启动顺序运行。

## 7。结论

在本文中，我们看到了什么是`CyclicBarrier`,以及它在什么样的情况下是有帮助的。

我们还实现了一个场景，在这个场景中，在继续执行其他程序逻辑之前，我们需要固定数量的线程来到达固定的执行点。

和往常一样，教程的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129014012/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced)