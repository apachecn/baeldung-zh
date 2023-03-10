# 在 Spring 批处理中配置跳过逻辑

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-batch-skip-logic>

## 1.介绍

默认情况下， [Spring 批处理](https://web.archive.org/web/20221129004032/https://spring.io/projects/spring-batch)作业处理过程中遇到的任何错误都会导致相应的步骤失败。然而，在很多情况下，我们宁愿跳过当前处理的项目来处理某些异常。

在本教程中，**我们将探索在 Spring 批处理框架中配置跳过逻辑的两种方法。**

## 2.我们的使用案例

为了举例，**我们将重用一个简单的、面向块的作业，这个作业已经在我们的 [Spring 批处理介绍文章](/web/20221129004032/https://www.baeldung.com/introduction-to-spring-batch)中介绍过了。**

该作业将一些财务数据从 CSV 格式转换为 XML 格式。

### 2.1.输入数据

首先，让我们在原始 CSV 文件中添加几行:

```java
username, user_id, transaction_date, transaction_amount
devendra, 1234, 31/10/2015, 10000
john, 2134, 3/12/2015, 12321
robin, 2134, 2/02/2015, 23411
, 2536, 3/10/2019, 100
mike, 9876, 5/11/2018, -500
, 3425, 10/10/2017, 9999
```

如我们所见，最后三行包含一些无效数据——第 5 行和第 7 行缺少用户名字段，第 6 行的交易金额为负。

在后面的部分中，我们将配置我们的批处理作业来跳过这些损坏的记录。

## 3.配置跳过限制和可跳过例外

### 3.1.使用`skip`和`skipLimit`

现在让我们讨论两种配置作业的方法中的第一种，以便在出现故障时跳过项目——`skip`和`skipLimit`方法:

```java
@Bean
public Step skippingStep(
  ItemProcessor<Transaction, Transaction> processor,
  ItemWriter<Transaction> writer) throws ParseException {
    return stepBuilderFactory
      .get("skippingStep")
      .<Transaction, Transaction>chunk(10)
      .reader(itemReader(invalidInputCsv))
      .processor(processor)
      .writer(writer)
      .faultTolerant()
      .skipLimit(2)
      .skip(MissingUsernameException.class)
      .skip(NegativeAmountException.class)
      .build();
}
```

首先，**要启用跳过功能，我们需要在步骤构建过程中包含对`faultTolerant()`的调用。**

在`skip()` 和`skipLimit()`中，我们定义了我们想要跳过的异常和跳过项目的最大数量。

在上面的例子中，如果在读取、处理或写入阶段抛出了`MissingUsernameException`或`NegativeAmountException`，那么当前处理的项目将被忽略，并计入两个项目的总数。

因此，**如果第三次抛出异常，那么整个步骤将失败**。

### 3.1.使用`noSkip`

在前面的例子中，除了`MissingUsernameException`和`NegativeAmountException`之外的任何其他异常都会使我们的步骤失败。

然而，在某些情况下，**识别应该使我们的步骤失败并跳过任何其他步骤的异常可能更合适。**

让我们看看如何使用`skip`、`skipLimit`和`noSkip`进行配置:

```java
@Bean
public Step skippingStep(
  ItemProcessor<Transaction, Transaction> processor,
  ItemWriter<Transaction> writer) throws ParseException {
    return stepBuilderFactory
      .get("skippingStep")
      .<Transaction, Transaction>chunk(10)
      .reader(itemReader(invalidInputCsv))
      .processor(processor)
      .writer(writer)
      .faultTolerant()
      .skipLimit(2)
      .skip(Exception.class)
      .noSkip(SAXException.class)
      .build();
}
```

通过上面的配置，我们指示 Spring Batch framework 跳过除了`SAXException`之外的任何`Exception`(在配置的限制内)。**这意味着`SAXException`总是导致一个步骤失败。**

`skip()`和`noSkip()`调用的顺序无关紧要。

## 4.使用自定义`SkipPolicy`

有时我们可能需要一个更复杂的跳过检查机制。为此， **Spring 批处理框架提供了 [`SkipPolicy`](https://web.archive.org/web/20221129004032/https://docs.spring.io/spring-batch/4.1.x/api/org/springframework/batch/core/step/skip/SkipPolicy.html) 接口。**

然后，我们可以提供自己的跳过逻辑实现，并将其插入到我们的步骤定义中。

记住前面的例子，假设我们仍然想要定义两个项目的跳过限制，并且只让`MissingUsernameException`和`NegativeAmountException`可以跳过。

然而，**一个额外的约束是，我们可以跳过`NegativeAmountException,`，但前提是金额不超过一个定义的限制**。

让我们实现我们的定制`SkipPolicy`:

```java
public class CustomSkipPolicy implements SkipPolicy {

    private static final int MAX_SKIP_COUNT = 2;
    private static final int INVALID_TX_AMOUNT_LIMIT = -1000;

    @Override
    public boolean shouldSkip(Throwable throwable, int skipCount) 
      throws SkipLimitExceededException {

        if (throwable instanceof MissingUsernameException && skipCount < MAX_SKIP_COUNT) {
            return true;
        }

        if (throwable instanceof NegativeAmountException && skipCount < MAX_SKIP_COUNT ) {
            NegativeAmountException ex = (NegativeAmountException) throwable;
            if(ex.getAmount() < INVALID_TX_AMOUNT_LIMIT) {
                return false;
            } else {
                return true;
            }
        }

        return false;
    }
}
```

现在，我们可以在步骤定义中使用我们的自定义策略:

```java
 @Bean
    public Step skippingStep(
      ItemProcessor<Transaction, Transaction> processor,
      ItemWriter<Transaction> writer) throws ParseException {
        return stepBuilderFactory
          .get("skippingStep")
          .<Transaction, Transaction>chunk(10)
          .reader(itemReader(invalidInputCsv))
          .processor(processor)
          .writer(writer)
          .faultTolerant()
          .skipPolicy(new CustomSkipPolicy())
          .build();
    }
```

并且，类似于我们之前的例子，我们仍然需要使用`faultTolerant()`来启用跳过功能。

然而，这次我们不调用`skip()`或`noSkip()`。相反，**我们使用`skipPolicy()`** **方法来提供我们自己实现的`SkipPolicy`** **接口。**

正如我们所见，**这种方法给了我们更多的灵活性，所以在某些用例中它是一个很好的选择**。

## 5.结论

在本教程中，我们介绍了两种使 Spring 批处理作业容错的方法。

尽管将`skipLimit()`与`skip()`和`noSkip()`方法一起使用似乎更受欢迎，但我们可能会发现在某些情况下实现自定义的`SkipPolicy`更方便。

像往常一样，GitHub 上的所有代码示例[都可用。](https://web.archive.org/web/20221129004032/https://github.com/eugenp/tutorials/tree/master/spring-batch)