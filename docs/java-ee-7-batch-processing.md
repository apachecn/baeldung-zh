# Java EE 7 批处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ee-7-batch-processing>

## 1。简介

想象一下，我们必须手动完成诸如处理工资单、计算利息和生成账单等任务。这将变得非常无聊，容易出错，并且是一个永无止境的手动任务列表！

在本教程中，我们将了解 Java 批处理( [JSR 352](https://web.archive.org/web/20220524022111/https://jcp.org/en/jsr/detail?id=352) )，它是 Jakarta EE 平台的一部分，也是自动化此类任务的一个很好的规范。**它为应用程序开发人员提供了一个开发健壮批处理系统的模型，这样他们就可以专注于业务逻辑。**

## 2.Maven 依赖性

由于 JSR 352 只是一个规范，我们需要包括[它的 API](https://web.archive.org/web/20220524022111/https://search.maven.org/artifact/javax.batch/javax.batch-api) 和[实现](https://web.archive.org/web/20220524022111/https://search.maven.org/search?q=org.jberet)，比如`[jberet](https://web.archive.org/web/20220524022111/https://github.com/jberet/jsr352)`:

```java
<dependency>
    <groupId>javax.batch</groupId>
    <artifactId>javax.batch-api</artifactId>
    <version>1.0.1</version>
</dependency>
<dependency>
    <groupId>org.jberet</groupId>
    <artifactId>jberet-core</artifactId>
    <version>1.0.2.Final</version>
</dependency>
<dependency>
    <groupId>org.jberet</groupId>
    <artifactId>jberet-support</artifactId>
    <version>1.0.2.Final</version>
</dependency>
<dependency>
    <groupId>org.jberet</groupId>
    <artifactId>jberet-se</artifactId>
    <version>1.0.2.Final</version>
</dependency>
```

我们还将添加一个内存数据库，这样我们就可以看到一些更真实的场景。

## 3。关键概念

JSR 352 引入了几个概念，我们可以这样看:

[![](img/8f5ffd5a97ed969d26ad79c4e1990eb2.png)](/web/20220524022111/https://www.baeldung.com/wp-content/uploads/2018/12/Screenshot-2018-11-07-at-11.55.06-AM.png)

让我们首先定义每一个部分:

*   从左边开始，我们有`JobOperator`。它**管理作业处理的所有方面，例如启动、停止和重启**
*   接下来，我们有了`Job`。作业是步骤的逻辑集合；它封装了整个批处理过程
*   一个作业将包含 1 到 n 个`Step` s。每个步骤都是一个独立的、连续的工作单元。一个步骤由`reading `输入、`processing`输入和`writing `输出组成
*   最后，同样重要的是，我们有`JobRepository `来存储作业的运行信息。它有助于跟踪作业、它们的状态以及它们的完成结果

步骤比这个更详细，所以接下来让我们来看看。首先，我们将查看`Chunk`步骤，然后查看`Batchlet`步骤。

## 4。创建块

如前所述，块是一种步骤`.`,我们经常使用块来表达一个反复执行的操作，比如一组项目。这有点像来自 Java 流的中间操作。

当描述一个块时，我们需要表达从哪里获取项目，如何处理它们，以及之后将它们发送到哪里。

### 4.1.阅读项目

**要读取条目，我们需要实现`ItemReader.`**

在这种情况下，我们将创建一个只发出数字 1 到 10 的读取器:

```java
@Named
public class SimpleChunkItemReader extends AbstractItemReader {
    private Integer[] tokens;
    private Integer count;

    @Inject
    JobContext jobContext;

    @Override
    public Integer readItem() throws Exception {
        if (count >= tokens.length) { 
            return null;
        }

        jobContext.setTransientUserData(count);
        return tokens[count++];
    }

    @Override
    public void open(Serializable checkpoint) throws Exception {
        tokens = new Integer[] { 1,2,3,4,5,6,7,8,9,10 };
        count = 0;
    }
}
```

现在，我们只是从类的内部状态来阅读。但是，当然， **`readItem `可以从数据库**，从文件系统，或者其他外部资源中提取。

请注意，我们正在使用`JobContext#setTransientUserData()`保存一些内部状态，这在以后会派上用场。

**另外，请注意`checkpoint `参数**。我们还会继续的。

### 4.2.处理项目

当然，我们分块的原因是我们想要对我们的项目执行某种操作！

每当我们从项目处理器返回`null` 时，我们就从批处理中删除该项目。

所以，让我们在这里说，我们只想保留偶数。我们可以使用一个通过返回`null`来拒绝奇数的`ItemProcessor`:

```java
@Named
public class SimpleChunkItemProcessor implements ItemProcessor {
    @Override
    public Integer processItem(Object t) {
        Integer item = (Integer) t;
        return item % 2 == 0 ? item : null;
    }
}
```

对于我们的`ItemReader`发出的每个项目， **`processItem` 将被调用一次。**

### 4.3.书写项目

最后，作业将调用`ItemWriter`,这样我们就可以编写转换后的项目:

```java
@Named
public class SimpleChunkWriter extends AbstractItemWriter {
    List<Integer> processed = new ArrayList<>();
    @Override
    public void writeItems(List<Object> items) throws Exception {
        items.stream().map(Integer.class::cast).forEach(processed::add);
    }
} 
```

**`items`有多长？**稍后，我们将定义一个块的大小，这将决定发送到`writeItems`的列表的大小。

### 4.4.在作业中定义块

现在，我们使用 JSL 或作业规范语言将所有这些放在一个 XML 文件中。请注意，我们将列出我们的阅读器、处理器、分块器以及块大小:

```java
<job id="simpleChunk">
    <step id="firstChunkStep" >
        <chunk item-count="3">
            <reader ref="simpleChunkItemReader"/>
            <processor ref="simpleChunkItemProcessor"/>
            <writer ref="simpleChunkWriter"/>
        </chunk>    
    </step>
</job>
```

块大小是块中的进程提交给作业存储库的频率，这对于在部分系统失败的情况下保证完成很重要。

我们需要将这个文件放在`META-INF/batch-jobs` 中。`jar `文件和在`WEB-INF/classes/META-INF/batch-jobs`办理`.war`文件的*。*

我们给了我们的工作 id `“simpleChunk”, `,所以让我们在单元测试中尝试一下。

**现在，任务是异步执行的，这使得它们很难测试。**在示例中，请务必查看我们的`BatchTestHelper `，它会进行轮询并等待任务完成:

```java
@Test
public void givenChunk_thenBatch_completesWithSuccess() throws Exception {
    JobOperator jobOperator = BatchRuntime.getJobOperator();
    Long executionId = jobOperator.start("simpleChunk", new Properties());
    JobExecution jobExecution = jobOperator.getJobExecution(executionId);
    jobExecution = BatchTestHelper.keepTestAlive(jobExecution);
    assertEquals(jobExecution.getBatchStatus(), BatchStatus.COMPLETED);
} 
```

这就是大块。现在，让我们来看看蝙蝠。

## 5.创建 Batchlet

不是所有的东西都适合迭代模型。例如，我们可能有一个任务，我们只需要**调用一次，运行完成，然后返回一个退出状态。**

batchlet 的合同非常简单:

```java
@Named
public class SimpleBatchLet extends AbstractBatchlet {

    @Override
    public String process() throws Exception {
        return BatchStatus.COMPLETED.toString();
    }
}
```

JSL 也是如此:

```java
<job id="simpleBatchLet">
    <step id="firstStep" >
        <batchlet ref="simpleBatchLet"/>
    </step>
</job>
```

我们可以使用与之前相同的方法进行测试:

```java
@Test
public void givenBatchlet_thenBatch_completeWithSuccess() throws Exception {
    JobOperator jobOperator = BatchRuntime.getJobOperator();
    Long executionId = jobOperator.start("simpleBatchLet", new Properties());
    JobExecution jobExecution = jobOperator.getJobExecution(executionId);
    jobExecution = BatchTestHelper.keepTestAlive(jobExecution);
    assertEquals(jobExecution.getBatchStatus(), BatchStatus.COMPLETED);
}
```

因此，我们已经看到了实现这些步骤的几种不同方法。

现在让我们看看标记和保证进度的机制。

## 6。自定义检查点

失败肯定会发生在工作当中。我们是应该从头开始，还是从我们停下的地方开始？

顾名思义，`checkpoints`帮助我们定期设置书签，以防失败。

**默认情况下，块处理的结尾是一个自然的检查点**。

但是，我们可以用自己的`CheckpointAlgorithm`对其进行定制:

```java
@Named
public class CustomCheckPoint extends AbstractCheckpointAlgorithm {

    @Inject
    JobContext jobContext;

    @Override
    public boolean isReadyToCheckpoint() throws Exception {
        int counterRead = (Integer) jobContext.getTransientUserData();
        return counterRead % 5 == 0;
    }
}
```

还记得我们之前在瞬态数据中放置的计数吗？在这里，**我们可以用** `**JobContext#getTransientUserData**`把它拉出来，说明我们希望每处理 5 个数字就提交一次。

如果没有这个，提交将发生在每个块的末尾，或者在我们的例子中，每三个数字发生一次。

**然后，我们将它与我们的块**下的 XML 中的`checkout-algorithm `指令进行匹配:

```java
<job id="customCheckPoint">
    <step id="firstChunkStep" >
        <chunk item-count="3" checkpoint-policy="custom">
            <reader ref="simpleChunkItemReader"/>
            <processor ref="simpleChunkItemProcessor"/>
            <writer ref="simpleChunkWriter"/>
            <checkpoint-algorithm ref="customCheckPoint"/>
        </chunk>    
    </step>
</job>
```

让我们测试代码，再次注意一些样板步骤隐藏在`BatchTestHelper`中:

```java
@Test
public void givenChunk_whenCustomCheckPoint_thenCommitCountIsThree() throws Exception {
    // ... start job and wait for completion

    jobOperator.getStepExecutions(executionId)
      .stream()
      .map(BatchTestHelper::getCommitCount)
      .forEach(count -> assertEquals(3L, count.longValue()));
    assertEquals(jobExecution.getBatchStatus(), BatchStatus.COMPLETED);
}
```

因此，我们可能期望提交计数为 2，因为我们有 10 个项目，并且将提交配置为每 5 个项目提交一次。但是，**框架在结尾**又做了一次最终读提交，以确保所有的事情都被处理了，这就是为什么我们有了 3。

接下来，让我们看看如何处理错误。

## 7。异常处理

默认情况下，**作业操作员会在出现异常时将我们的作业标记为*失败*。**

让我们改变我们的项目阅读器，以确保它失败:

```java
@Override
public Integer readItem() throws Exception {
    if (tokens.hasMoreTokens()) {
        String tempTokenize = tokens.nextToken();
        throw new RuntimeException();
    }
    return null;
}
```

然后测试:

```java
@Test
public void whenChunkError_thenBatch_CompletesWithFailed() throws Exception {
    // ... start job and wait for completion
    assertEquals(jobExecution.getBatchStatus(), BatchStatus.FAILED);
}
```

**但是，我们可以用多种方式覆盖这个默认行为:**

*   `skip-limit` 指定该步骤在失败前将忽略的异常数
*   `retry-limit `指定作业操作员在失败前应重试该步骤的次数
*   `skippable-exception-class` 指定块处理将忽略的一组异常

因此，我们可以编辑我们的作业，使其忽略`RuntimeException`，以及其他几个，这只是为了说明:

```java
<job id="simpleErrorSkipChunk" >
    <step id="errorStep" >
        <chunk checkpoint-policy="item" item-count="3" skip-limit="3" retry-limit="3">
            <reader ref="myItemReader"/>
            <processor ref="myItemProcessor"/>
            <writer ref="myItemWriter"/>
            <skippable-exception-classes>
                <include class="java.lang.RuntimeException"/>
                <include class="java.lang.UnsupportedOperationException"/>
            </skippable-exception-classes>
            <retryable-exception-classes>
                <include class="java.lang.IllegalArgumentException"/>
                <include class="java.lang.UnsupportedOperationException"/>
            </retryable-exception-classes>
        </chunk>
    </step>
</job>
```

现在我们的代码将通过:

```java
@Test
public void givenChunkError_thenErrorSkipped_CompletesWithSuccess() throws Exception {
   // ... start job and wait for completion
   jobOperator.getStepExecutions(executionId).stream()
     .map(BatchTestHelper::getProcessSkipCount)
     .forEach(skipCount -> assertEquals(1L, skipCount.longValue()));
   assertEquals(jobExecution.getBatchStatus(), BatchStatus.COMPLETED);
}
```

## 8。执行多个步骤

我们之前提到过，一个作业可以有任意数量的步骤，现在让我们来看看。

### 8.1.启动下一步

默认情况下，**每一步都是作业**的最后一步。

为了执行批处理作业中的下一步，我们必须使用步骤定义中的`next`属性明确指定:

```java
<job id="simpleJobSequence">
    <step id="firstChunkStepStep1" next="firstBatchStepStep2">
        <chunk item-count="3">
            <reader ref="simpleChunkItemReader"/>
            <processor ref="simpleChunkItemProcessor"/>
            <writer ref="simpleChunkWriter"/>
        </chunk>    
    </step>
    <step id="firstBatchStepStep2" >
        <batchlet ref="simpleBatchLet"/>
    </step>
</job>
```

如果我们忘记了这个属性，那么序列中的下一步将不会被执行。

我们可以在 API 中看到这是什么样子:

```java
@Test
public void givenTwoSteps_thenBatch_CompleteWithSuccess() throws Exception {
    // ... start job and wait for completion
    assertEquals(2 , jobOperator.getStepExecutions(executionId).size());
    assertEquals(jobExecution.getBatchStatus(), BatchStatus.COMPLETED);
}
```

### 8.2.流

一系列步骤也可以封装成一个`flow`。**当流程结束时，整个流程转移到执行元素**。此外，流内的元素不能过渡到流外的元素。

比方说，我们可以在一个流程中执行两个步骤，然后将该流程转换为一个独立的步骤:

```java
<job id="flowJobSequence">
    <flow id="flow1" next="firstBatchStepStep3">
        <step id="firstChunkStepStep1" next="firstBatchStepStep2">
            <chunk item-count="3">
	        <reader ref="simpleChunkItemReader" />
		<processor ref="simpleChunkItemProcessor" />
		<writer ref="simpleChunkWriter" />
	    </chunk>
	</step>
	<step id="firstBatchStepStep2">
	    <batchlet ref="simpleBatchLet" />
	</step>
    </flow>
    <step id="firstBatchStepStep3">
	 <batchlet ref="simpleBatchLet" />
    </step>
</job>
```

而且我们仍然可以看到每个步骤独立地执行:

```java
@Test
public void givenFlow_thenBatch_CompleteWithSuccess() throws Exception {
    // ... start job and wait for completion

    assertEquals(3, jobOperator.getStepExecutions(executionId).size());
    assertEquals(jobExecution.getBatchStatus(), BatchStatus.COMPLETED);
}
```

### 8.3.决定

我们还有以`decisions`形式的 if/else 支持。**决策提供了** **一种定制的方式来确定步骤、流程和分割**之间的顺序。

像步骤一样，它作用于过渡元素，如`next` ，它可以指导或终止作业的执行。

让我们看看如何配置作业:

```java
<job id="decideJobSequence">
     <step id="firstBatchStepStep1" next="firstDecider">
	 <batchlet ref="simpleBatchLet" />
     </step>	
     <decision id="firstDecider" ref="deciderJobSequence">
        <next on="two" to="firstBatchStepStep2"/>
        <next on="three" to="firstBatchStepStep3"/>
     </decision>
     <step id="firstBatchStepStep2">
	<batchlet ref="simpleBatchLet" />
     </step>	
     <step id="firstBatchStepStep3">
	<batchlet ref="simpleBatchLet" />
     </step>		
</job>
```

任何`decision`元素都需要配置一个实现`Decider`的类。它的工作是作为一个`String`返回一个决定。

`decision`中的每个`next`就像是`switch `语句中的一个`case `。

### 8.4.分裂

非常方便，因为它们允许我们同时执行流:

```java
<job id="splitJobSequence">
   <split id="split1" next="splitJobSequenceStep3">
      <flow id="flow1">
	  <step id="splitJobSequenceStep1">
              <batchlet ref="simpleBatchLet" />
           </step>
      </flow>
      <flow id="flow2">
          <step id="splitJobSequenceStep2">
              <batchlet ref="simpleBatchLet" />
	  </step>
      </flow>
   </split>
   <step id="splitJobSequenceStep3">
      <batchlet ref="simpleBatchLet" />
   </step>
</job>
```

当然，**这意味着订单不能保证**。

让我们确认他们仍然都得到运行。**流程步骤将以任意顺序执行，但隔离步骤将始终在最后:**

```java
@Test
public void givenSplit_thenBatch_CompletesWithSuccess() throws Exception {
    // ... start job and wait for completion
    List<StepExecution> stepExecutions = jobOperator.getStepExecutions(executionId);

    assertEquals(3, stepExecutions.size());
    assertEquals("splitJobSequenceStep3", stepExecutions.get(2).getStepName());
    assertEquals(jobExecution.getBatchStatus(), BatchStatus.COMPLETED);
}
```

## 9。对作业进行分区

我们还可以使用 Java 代码中的批处理属性，这些属性已经在我们的作业中定义了。

它们可以在三个层次上确定范围——作业、步骤和批处理工件。

让我们看一些他们如何消费的例子。

当我们希望在作业级别使用属性时:

```java
@Inject
JobContext jobContext;
...
jobProperties = jobContext.getProperties();
...
```

这也可以在步骤级别消耗:

```java
@Inject
StepContext stepContext;
...
stepProperties = stepContext.getProperties();
...
```

当我们想要在批处理工件级别使用属性时:

```java
@Inject
@BatchProperty(name = "name")
private String nameString;
```

这对于分区来说很方便。

看，通过拆分，我们可以同时运行流。**但是我们也可以`partition`一步进入`n `项集或者设置单独的输入，这允许我们用另一种方式在多线程间分配工作。**

为了理解每个分区应该完成的工作部分，我们可以将属性与分区结合起来:

```java
<job id="injectSimpleBatchLet">
    <properties>
        <property name="jobProp1" value="job-value1"/>
    </properties>
    <step id="firstStep">
        <properties>
            <property name="stepProp1" value="value1" />
        </properties>
	<batchlet ref="injectSimpleBatchLet">
	    <properties>
		<property name="name" value="#{partitionPlan['name']}" />
	    </properties>
	</batchlet>
	<partition>
	    <plan partitions="2">
		<properties partition="0">
		    <property name="name" value="firstPartition" />
		</properties>
		<properties partition="1">
		    <property name="name" value="secondPartition" />
		</properties>
	    </plan>
	</partition>
    </step>
</job>
```

## 10。停止并重启

现在，这就是工作的定义。现在，让我们花一分钟来讨论如何管理它们。

我们已经在单元测试中看到，我们可以从`BatchRuntime`中获得`JobOperator `的实例:

```java
JobOperator jobOperator = BatchRuntime.getJobOperator();
```

然后，我们可以开始工作了:

```java
Long executionId = jobOperator.start("simpleBatchlet", new Properties());
```

但是，我们也可以停止作业:

```java
jobOperator.stop(executionId);
```

最后，我们可以重新启动作业:

```java
executionId = jobOperator.restart(executionId, new Properties());
```

让我们看看如何停止正在运行的作业:

```java
@Test
public void givenBatchLetStarted_whenStopped_thenBatchStopped() throws Exception {
    JobOperator jobOperator = BatchRuntime.getJobOperator();
    Long executionId = jobOperator.start("simpleBatchLet", new Properties());
    JobExecution jobExecution = jobOperator.getJobExecution(executionId);
    jobOperator.stop(executionId);
    jobExecution = BatchTestHelper.keepTestStopped(jobExecution);
    assertEquals(jobExecution.getBatchStatus(), BatchStatus.STOPPED);
}
```

如果一个批处理是`STOPPED`，那么我们可以重新启动它:

```java
@Test
public void givenBatchLetStopped_whenRestarted_thenBatchCompletesSuccess() {
    // ... start and stop the job

    assertEquals(jobExecution.getBatchStatus(), BatchStatus.STOPPED);
    executionId = jobOperator.restart(jobExecution.getExecutionId(), new Properties());
    jobExecution = BatchTestHelper.keepTestAlive(jobOperator.getJobExecution(executionId));

    assertEquals(jobExecution.getBatchStatus(), BatchStatus.COMPLETED);
}
```

## 11。抓取作业

当一个批处理作业被提交，那么**批处理运行时创建一个`JobExecution`的实例来跟踪它**。

为了获得执行 id 的`JobExecution`，我们可以使用`JobOperator#getJobExecution(executionId)`方法。

并且， **`StepExecution`为跟踪一个步骤的执行**提供了有用的信息。

为了获得执行 id 的`StepExecution`，我们可以使用`JobOperator#getStepExecutions(executionId)`方法。

由此，我们可以通过`StepExecution#getMetrics:`得到[关于该步骤的几个度量](https://web.archive.org/web/20220524022111/https://docs.oracle.com/javaee/7/api/javax/batch/runtime/Metric.MetricType.html)

```java
@Test
public void givenChunk_whenJobStarts_thenStepsHaveMetrics() throws Exception {
    // ... start job and wait for completion
    assertTrue(jobOperator.getJobNames().contains("simpleChunk"));
    assertTrue(jobOperator.getParameters(executionId).isEmpty());
    StepExecution stepExecution = jobOperator.getStepExecutions(executionId).get(0);
    Map<Metric.MetricType, Long> metricTest = BatchTestHelper.getMetricsMap(stepExecution.getMetrics());
    assertEquals(10L, metricTest.get(Metric.MetricType.READ_COUNT).longValue());
    assertEquals(5L, metricTest.get(Metric.MetricType.FILTER_COUNT).longValue());
    assertEquals(4L, metricTest.get(Metric.MetricType.COMMIT_COUNT).longValue());
    assertEquals(5L, metricTest.get(Metric.MetricType.WRITE_COUNT).longValue());
    // ... and many more!
}
```

## 12。缺点

JSR 352 功能强大，但在许多方面有所欠缺:

*   似乎缺少可以处理 JSON 等其他格式的阅读器和编写器
*   不支持泛型
*   分区仅支持单一步骤
*   该 API 不提供任何支持调度的东西(尽管 J2EE 有一个单独的调度模块)
*   由于它的异步特性，测试可能是一个挑战
*   这个 API 相当冗长

## 13。结论

在这篇文章中，我们看了 JSR 352，了解了组块、批处理、拆分、流等等。然而，我们仅仅触及了表面。

和往常一样，演示代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524022111/https://github.com/eugenp/tutorials/tree/master/jee-7)