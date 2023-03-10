# 春季批次中的条件流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-batch-conditional-flow>

## 1.介绍

我们使用 [Spring Batch](/web/20220628090616/https://www.baeldung.com/introduction-to-spring-batch) 从读取、转换和写入数据的多个步骤中组合作业。如果一个作业中的步骤有多条路径，类似于在我们的代码中使用一个`if`语句，我们说作业流是`conditional`。

在本教程中，我们将研究两种使用条件流创建 Spring 批处理作业的方法。

## 2.退出状态和批处理状态

当我们用 Spring 的批处理框架指定一个条件步骤时，我们使用的是一个步骤或作业的退出状态。因此，我们需要理解步骤和作业中批处理状态和退出状态之间的区别:

*   **`BatchStatus`是代表步骤/作业状态的枚举，由批处理框架内部使用**
*   可能的值有:`ABANDONED, COMPLETED, FAILED, STARTED, STARTING, STOPPED, STOPPING, UNKNOWN`
*   **`ExitStatus`是执行完成时一个步骤的状态，用于有条件地确定流程**

默认情况下，步骤或作业的`ExitStatus`与其`BatchStatus`相同。我们也可以设置一个自定义的`ExitStatus`来驱动流量。

## 3.条件流

比方说，我们有一个 IOT 设备发送给我们的测量。我们的设备度量是整数数组，如果我们的任何度量包含正整数，我们需要发送通知。

换句话说，我们需要在检测到阳性测量值时发送通知。

### 3.1.`ExitStatus`

重要的是，**我们使用一个步骤的退出状态来驱动条件流**。

**要设置一个步骤的退出状态，我们需要使用`StepExecution`对象的`setExitStatus`方法。**为了做到这一点，我们需要创建一个`ItemProcessor` ，它扩展了`ItemListenerSupport `，并获得了步骤的`StepExecution`。

当我们找到一个正数时，我们用它来设置步骤的退出状态为`NOTIFY`。**当我们基于批处理作业中的数据确定我们的退出状态时，我们可以使用一个`ItemProcessor`。**

让我们看看我们的`NumberInfoClassifier`,看看我们需要的三种方法:

```java
public class NumberInfoClassifier extends ItemListenerSupport<NumberInfo, Integer>
  implements ItemProcessor<NumberInfo, Integer> {

    private StepExecution stepExecution;

    @BeforeStep
    public void beforeStep(StepExecution stepExecution) {
        this.stepExecution = stepExecution;
        this.stepExecution.setExitStatus(new ExitStatus(QUIET));
    }

    @Override
    public Integer process(NumberInfo numberInfo) throws Exception {
        return Integer.valueOf(numberInfo.getNumber());
    }

    @Override
    public void afterProcess(NumberInfo item, Integer result) {
        super.afterProcess(item, result);
        if (item.isPositive()) {
            stepExecution.setExitStatus(new ExitStatus(NOTIFY));
        }
    }
}
```

注意:在这个例子中，我们使用`ItemProcessor`来设置`ExitStatus`，但是我们也可以在我们的步骤的`ItemReader`或`ItemWriter`中轻松地这样做。

最后，当我们创建我们的作业时，我们告诉我们的`JobBuilderFactory`为任何以状态`NOTIFY`退出的步骤发送通知:

```java
jobBuilderFactory.get("Number generator - second dataset")
    .start(dataProviderStep)
    .on("NOTIFY").to(notificationStep)
    .end()
    .build();
```

还要注意，当我们有额外的条件分支和多个退出代码时，我们可以用`JobBuilderFacotry`的`from` 和 `on`方法将它们添加到我们的作业中:

```java
jobBuilderFactory.get("Number generator - second dataset")
    .start(dataProviderStep)
    .on("NOTIFY").to(notificationStep)
    .from(step).on("LOG_ERROR").to(errorLoggingStep)
    .end()
    .build();
```

现在，只要我们的`ItemProcessor`看到一个正数，它就会指示我们的作业运行`notificationStep`，它只是向`System.out`打印一条消息:

```java
Second Dataset Processor 11
Second Dataset Processor -2
Second Dataset Processor -3
[Number generator - second dataset] contains interesting data!! 
```

如果我们有一个没有正数的数据集，我们将看不到我们的`notificationStep`消息:

```java
Second Dataset Processor -1
Second Dataset Processor -2
Second Dataset Processor -3 
```

### 3.2.使用`JobExecutionDecider`的程序分支

或者，我们可以使用实现`JobExecutionDecider`的类来确定作业流。如果我们有决定执行流程的外部因素，这就特别有用**。**

要使用这个方法，我们首先需要修改我们的`ItemProcessor`来移除`ItemListenerSupport `接口和`@BeforeStep`方法:

```java
public class NumberInfoClassifierWithDecider extends ItemListenerSupport<NumberInfo, Integer>
  implements ItemProcessor<NumberInfo, Integer> {

    @Override
    public Integer process(NumberInfo numberInfo) throws Exception {
        return Integer.valueOf(numberInfo.getNumber());
    }
} 
```

接下来，我们创建一个决定步骤通知状态的 decider 类:

```java
public class NumberInfoDecider implements JobExecutionDecider {

    private boolean shouldNotify() {
        return true;
    }

    @Override
    public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {
        if (shouldNotify()) {
            return new FlowExecutionStatus(NOTIFY);
        } else {
            return new FlowExecutionStatus(QUIET);
        }
    }
} 
```

然后，我们设置我们的`Job`在流程中使用决策器:

```java
jobBuilderFactory.get("Number generator - third dataset")
    .start(dataProviderStep)
    .next(new NumberInfoDecider()).on("NOTIFY").to(notificationStep)
    .end()
    .build(); 
```

## 4.结论

在这个快速教程中，我们探索了用 Spring Batch 实现条件流的两个选项。首先，我们看了如何使用`ExitStatus`来控制我们的工作流。

然后，我们看了看如何通过定义自己的`JobExecutionDecider`以编程方式控制流程。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220628090616/https://github.com/eugenp/tutorials/tree/master/spring-batch)