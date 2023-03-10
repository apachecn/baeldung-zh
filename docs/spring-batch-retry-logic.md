# 在 Spring 批处理中配置重试逻辑

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-batch-retry-logic>

## 1.概观

默认情况下，Spring 批处理作业会因执行过程中出现的任何错误而失败。然而，有时我们可能希望提高应用程序的弹性来处理间歇性故障。

在这个快速教程中，**我们将探索如何在 Spring 批处理框架**中配置重试逻辑。

## 2.一个示例用例

假设我们有一个读取输入 CSV 文件的批处理作业:

```java
username, userid, transaction_date, transaction_amount
sammy, 1234, 31/10/2015, 10000
john, 9999, 3/12/2015, 12321
```

然后，它通过点击 REST 端点获取用户的`age`和`postCode`属性来处理每条记录:

```java
public class RetryItemProcessor implements ItemProcessor<Transaction, Transaction> {

    @Override
    public Transaction process(Transaction transaction) throws IOException {
        log.info("RetryItemProcessor, attempting to process: {}", transaction);
        HttpResponse response = fetchMoreUserDetails(transaction.getUserId());
        //parse user's age and postCode from response and update transaction
        ...
        return transaction;
    }
    ...
}
```

最后，它生成一个合并输出`XML`:

```java
<transactionRecord>
    <transactionRecord>
        <amount>10000.0</amount>
        <transactionDate>2015-10-31 00:00:00</transactionDate>
        <userId>1234</userId>
        <username>sammy</username>
        <age>10</age>
        <postCode>430222</postCode>
    </transactionRecord>
    ...
</transactionRecord>
```

## 3.向`ItemProcessor`添加重试次数

现在，如果到 REST 端点的连接由于网络速度慢而超时，该怎么办呢？如果是这样，我们的批处理作业将会失败。

在这种情况下，我们更希望失败的项目处理被重试几次。因此，**让我们配置我们的批处理作业，以便在失败的情况下最多执行三次重试**:

```java
@Bean
public Step retryStep(
  ItemProcessor<Transaction, Transaction> processor,
  ItemWriter<Transaction> writer) throws ParseException {
    return stepBuilderFactory
      .get("retryStep")
      .<Transaction, Transaction>chunk(10)
      .reader(itemReader(inputCsv))
      .processor(processor)
      .writer(writer)
      .faultTolerant()
      .retryLimit(3)
      .retry(ConnectTimeoutException.class)
      .retry(DeadlockLoserDataAccessException.class)
      .build();
}
```

这里，我们调用了 `faultTolerant()`来启用重试功能。另外，**我们使用`retry`和`retryLimit`来分别定义符合重试条件的异常和项目的最大重试次数**。

## 4.测试重试次数

让我们来看一个测试场景，在这个场景中，返回`age`和`postCode`的 REST 端点关闭了一段时间。在这个测试场景中，我们将只获得前两个 API 调用的`ConnectTimeoutException`，第三个调用将会成功:

```java
@Test
public void whenEndpointFailsTwicePasses3rdTime_thenSuccess() throws Exception {
    FileSystemResource expectedResult = new FileSystemResource(EXPECTED_OUTPUT);
    FileSystemResource actualResult = new FileSystemResource(TEST_OUTPUT);

    when(httpResponse.getEntity())
      .thenReturn(new StringEntity("{ \"age\":10, \"postCode\":\"430222\" }"));

    //fails for first two calls and passes third time onwards
    when(httpClient.execute(any()))
      .thenThrow(new ConnectTimeoutException("Timeout count 1"))
      .thenThrow(new ConnectTimeoutException("Timeout count 2"))
      .thenReturn(httpResponse);

    JobExecution jobExecution = jobLauncherTestUtils
      .launchJob(defaultJobParameters());
    JobInstance actualJobInstance = jobExecution.getJobInstance();
    ExitStatus actualJobExitStatus = jobExecution.getExitStatus();

    assertThat(actualJobInstance.getJobName(), is("retryBatchJob"));
    assertThat(actualJobExitStatus.getExitCode(), is("COMPLETED"));
    AssertFile.assertFileEquals(expectedResult, actualResult);
}
```

在这里，我们的工作成功完成了。另外，从日志中可以明显看出，**与`id=1234`的第一条记录失败了两次，最后在第三次重试**时成功了:

```java
19:06:57.742 [main] INFO  o.s.batch.core.job.SimpleStepHandler - Executing step: [retryStep]
19:06:57.758 [main] INFO  o.b.batch.service.RetryItemProcessor - Attempting to process user with id=1234
19:06:57.758 [main] INFO  o.b.batch.service.RetryItemProcessor - Attempting to process user with id=1234
19:06:57.758 [main] INFO  o.b.batch.service.RetryItemProcessor - Attempting to process user with id=1234
19:06:57.758 [main] INFO  o.b.batch.service.RetryItemProcessor - Attempting to process user with id=9999
19:06:57.773 [main] INFO  o.s.batch.core.step.AbstractStep - Step: [retryStep] executed in 31ms
```

类似地，让我们用**另一个测试用例来看看当所有重试都用尽时会发生什么**:

```java
@Test
public void whenEndpointAlwaysFail_thenJobFails() throws Exception {
    when(httpClient.execute(any()))
      .thenThrow(new ConnectTimeoutException("Endpoint is down"));

    JobExecution jobExecution = jobLauncherTestUtils
      .launchJob(defaultJobParameters());
    JobInstance actualJobInstance = jobExecution.getJobInstance();
    ExitStatus actualJobExitStatus = jobExecution.getExitStatus();

    assertThat(actualJobInstance.getJobName(), is("retryBatchJob"));
    assertThat(actualJobExitStatus.getExitCode(), is("FAILED"));
    assertThat(actualJobExitStatus.getExitDescription(),
      containsString("org.apache.http.conn.ConnectTimeoutException"));
}
```

在这种情况下，**对第一条记录尝试了三次重试，最终由于`ConnectTimeoutException`** 导致作业失败。

## 5.使用 XML 配置重试

最后，让我们看一下上述配置的 XML 等价物:

```java
<batch:job id="retryBatchJob">
    <batch:step id="retryStep">
        <batch:tasklet>
            <batch:chunk reader="itemReader" writer="itemWriter"
              processor="retryItemProcessor" commit-interval="10"
              retry-limit="3">
                <batch:retryable-exception-classes>
                    <batch:include class="org.apache.http.conn.ConnectTimeoutException"/>
                    <batch:include class="org.springframework.dao.DeadlockLoserDataAccessException"/>
                </batch:retryable-exception-classes>
            </batch:chunk>
        </batch:tasklet>
    </batch:step>
</batch:job>
```

## 6.结论

在本文中，我们学习了如何在 Spring Batch 中配置重试逻辑。我们研究了 Java 和 XML 配置。

我们还使用了一个单元测试来观察重试在实践中是如何工作的。

和往常一样，本教程的示例代码可以在 GitHub 上找到。