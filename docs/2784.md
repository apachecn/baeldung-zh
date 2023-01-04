# Java Phaser 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-phaser>

## 1。概述

在本文中，我们将关注来自`java.util.concurrent`包的`[Phaser](https://web.archive.org/web/20221126224800/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Phaser.html)` 构造。它与 [`CountDownLatch`](/web/20221126224800/https://www.baeldung.com/java-countdown-latch) 非常相似，允许我们协调线程的执行。与`CountDownLatch`相比，它有一些额外的功能。

`Phaser` 是一个障碍，在继续执行之前，动态数量的线程需要在其上等待。在`CountDownLatch` 中，这个数字不能动态配置，需要在我们创建实例时提供。

## 2。`Phaser` API

`Phaser`允许我们建立逻辑，其中**线程在进入下一步执行**之前需要等待屏障。

我们可以协调多个执行阶段，为每个程序阶段重用一个`Phaser`实例。每个阶段可以有不同数量的线程等待前进到另一个阶段。稍后我们会看一个使用阶段的例子。

为了参与协调，线程需要用`Phaser`实例来`register()` 自己。请注意，这只会增加注册方的数量，我们无法检查当前线程是否已注册——我们必须对实现进行子类化以支持这一点。

线程通过调用`arriveAndAwaitAdvance()`(一种阻塞方法)来表示它到达了屏障。**当到达方的数量等于登记方的数量时，程序将继续执行**，阶段数将增加。我们可以通过调用`getPhase()` 方法得到当前的相数。

当线程完成它的任务时，我们应该调用`arriveAndDeregister()`方法来通知当前线程在这个特定的阶段不应该再被考虑。

## 3。使用`Phaser` API 实现逻辑

假设我们想要协调多个阶段的行动。三个线程将处理第一阶段，两个线程将处理第二阶段。

我们将创建一个实现`Runnable` 接口的`LongRunningAction` 类:

```
class LongRunningAction implements Runnable {
    private String threadName;
    private Phaser ph;

    LongRunningAction(String threadName, Phaser ph) {
        this.threadName = threadName;
        this.ph = ph;
        ph.register();
    }

    @Override
    public void run() {
        ph.arriveAndAwaitAdvance();
        try {
            Thread.sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        ph.arriveAndDeregister();
    }
}
```

当我们的 action 类被实例化时，我们使用`register()` 方法注册到`Phaser` 实例。这将增加使用特定`Phaser.` 的线程数量

对`arriveAndAwaitAdvance()` 的调用将导致当前线程等待屏障。如前所述，当到达方的数量与注册方的数量相同时，执行将继续。

处理完成后，当前线程通过调用`arriveAndDeregister()` 方法注销自己。

让我们创建一个测试用例，在这个用例中，我们将启动三个`LongRunningAction`线程，并在屏障上阻塞。接下来，在动作完成后，我们将创建两个额外的`LongRunningAction` 线程，它们将执行下一阶段的处理。

当从主线程创建`Phaser` 实例时，我们将`1`作为参数传递。这相当于从当前线程中调用`register()` 方法。我们这样做是因为，当我们创建三个工作线程时，主线程是一个协调器，因此`Phaser`需要注册四个线程:

```
ExecutorService executorService = Executors.newCachedThreadPool();
Phaser ph = new Phaser(1);

assertEquals(0, ph.getPhase());
```

初始化后的相位等于零。

`Phaser` 类有一个构造函数，我们可以向它传递一个父实例。在我们有大量参与方会经历巨大的同步争用成本的情况下，这是很有用的。在这种情况下，可以建立`Phasers`的实例，使得子阶段的组共享一个公共父阶段。

接下来，让我们启动三个`LongRunningAction` 动作线程，它们将在 barrier 上等待，直到我们从主线程调用`arriveAndAwaitAdvance()`方法。

请记住，我们已经用`1`初始化了`Phaser`，并调用了`register()`三次。现在，三个动作线程已经宣布它们已经到达了关卡，所以还需要一个对`arriveAndAwaitAdvance()`的调用——来自主线程:

```
executorService.submit(new LongRunningAction("thread-1", ph));
executorService.submit(new LongRunningAction("thread-2", ph));
executorService.submit(new LongRunningAction("thread-3", ph));

ph.arriveAndAwaitAdvance();

assertEquals(1, ph.getPhase());
```

在该阶段完成后，`getPhase()` 方法将返回 1，因为程序完成了执行的第一步。

假设两个线程应该执行下一阶段的处理。我们可以利用`Phaser` 来实现这一点，因为它允许我们动态配置应该在屏障上等待的线程数量。我们正在启动两个新线程，但是这些线程将不会继续执行，直到从主线程调用`arriveAndAwaitAdvance()` (与前一个例子相同):

```
executorService.submit(new LongRunningAction("thread-4", ph));
executorService.submit(new LongRunningAction("thread-5", ph));
ph.arriveAndAwaitAdvance();

assertEquals(2, ph.getPhase());

ph.arriveAndDeregister();
```

此后，`getPhase()` 方法将返回等于 2 的相位数。当我们想结束我们的程序时，我们需要调用`arriveAndDeregister()` 方法，因为主线程仍然注册在`Phaser.` 中。当注销导致注册方的数量变为零时，`Phaser`是`terminated.` 。所有对同步方法的调用将不再阻塞，并将立即返回。

运行该程序将产生以下输出(带有打印行语句的完整源代码可以在代码库中找到):

```
This is phase 0
This is phase 0
This is phase 0
Thread thread-2 before long running action
Thread thread-1 before long running action
Thread thread-3 before long running action
This is phase 1
This is phase 1
Thread thread-4 before long running action
Thread thread-5 before long running action
```

我们看到所有线程都在等待执行，直到屏障打开。只有在前一阶段成功完成后，才会执行下一阶段的执行。

## 4。结论

在本教程中，我们看了一下来自`java.util.concurrent`的`Phaser` 构造，并使用`Phaser` 类实现了多阶段协调逻辑。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221126224800/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。