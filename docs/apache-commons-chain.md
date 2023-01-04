# 阿帕奇公共链

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-chain>

## 1。简介

Apache Commons Chain 是一个使用责任链[模式](https://web.archive.org/web/20221112032904/https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)的库，通常用于组织复杂的处理流程，其中多个接收者可以处理一个请求。

在这篇简短的文章中，我们将看一个从 ATM 机取款的例子。

## 2。Maven 依赖关系

首先，我们将使用 Maven 导入这个库的最新版本:

```java
<dependency>
    <groupId>commons-chain</groupId>
    <artifactId>commons-chain</artifactId>
    <version>1.2</version>
</dependency> 
```

要查看本库的最新版本，请点击这里。

## 3。示例链

自动柜员机将一个数字作为输入，并将其传递给负责执行不同操作的处理程序。这些包括计算要分发的钞票数量，并向银行和客户发送交易通知。

## 4。链上下文

上下文表示应用程序的当前状态，存储有关事务的信息。

对于我们的 ATM 取款请求，我们需要的信息是:

*   要提取的总金额
*   100 面额纸币的数量
*   50 面值纸币的数量
*   10 面额纸币的数量
*   有待提取的金额

这种状态在类中定义:

```java
public class AtmRequestContext extends ContextBase {
    int totalAmountToBeWithdrawn;
    int noOfHundredsDispensed;
    int noOfFiftiesDispensed;
    int noOfTensDispensed;
    int amountLeftToBeWithdrawn;

    // standard setters & getters
}
```

## 5。命令

`Command` 将`C` `ontext`作为输入并进行处理。

我们将把上面提到的每个步骤都实现为一个`Command:`

```java
public class HundredDenominationDispenser implements Command {

    @Override
    public boolean execute(Context context) throws Exception {
        intamountLeftToBeWithdrawn = (int) context.get("amountLeftToBeWithdrawn);
        if (amountLeftToBeWithdrawn >= 100) {
            context.put("noOfHundredsDispensed", amountLeftToBeWithdrawn / 100);
            context.put("amountLeftToBeWithdrawn", amountLeftToBeWithdrawn % 100);
        }
        return false;
    }
} 
```

`FiftyDenominationDispenser` & `TenDenominationDispenser` 的`Command`年代也差不多。

## 6。链条

一个`Chain` 是以指定顺序执行的命令集合。我们的`Chain` 将由上面的`Command`和结尾的`AuditFilter` 组成:

```java
public class AtmWithdrawalChain extends ChainBase {

    public AtmWithdrawalChain() {
        super();
        addCommand(new HundredDenominationDispenser());
        addCommand(new FiftyDenominationDispenser());
        addCommand(new TenDenominationDispenser());
        addCommand(new AuditFilter());
    }
}
```

当`Chain` 中的任意一个`Command` 返回 true 时，强制`Chain` 结束。

## 7。过滤器

过滤器也是一个`Command` ，但是有一个在`Chain.` 执行后被调用的`postProcess` 方法

我们的`Filter`将向客户&银行发送通知:

```java
public class AuditFilter implements Filter {

    @Override
    public boolean postprocess(Context context, Exception exception) {
        // send notification to bank and user
        return false;
    }

    @Override
    public boolean execute(Context context) throws Exception {
        return false;
    }
}
```

## 8。连锁目录

它是带有逻辑名称的`Chains`和`Commands`的集合。

在我们的例子中，我们的`Catalog` 将包含`AtmWithdrawalChain.`

```java
public class AtmCatalog extends CatalogBase {

    public AtmCatalog() {
        super();
        addCommand("atmWithdrawalChain", new AtmWithdrawalChain());
    }
}
```

## 9。使用链条

让我们看看如何使用上面的`Chain` 来处理取款请求。我们将首先创建一个`Context` ，然后将它传递给`Chain.`,`Chain` 将处理`Context.`

我们将编写一个测试用例来演示我们的`AtmWithdrawalChain:`

```java
public class AtmChainTest {

    @Test
    public void givenInputsToContext_whenAppliedChain_thenExpectedContext() throws Exception {
        Context context = new AtmRequestContext();
        context.put("totalAmountToBeWithdrawn", 460);
        context.put("amountLeftToBeWithdrawn", 460);

        Catalog catalog = new AtmCatalog();
        Command atmWithdrawalChain = catalog.getCommand("atmWithdrawalChain");

        atmWithdrawalChain.execute(context);

        assertEquals(460, (int) context.get("totalAmountToBeWithdrawn"));
        assertEquals(0, (int) context.get("amountLeftToBeWithdrawn"));
        assertEquals(4, (int) context.get("noOfHundredsDispensed"));
        assertEquals(1, (int) context.get("noOfFiftiesDispensed"));
        assertEquals(1, (int) context.get("noOfTensDispensed"));
    }
}
```

## 10。结论

在本教程中，我们探索了一个使用 Apache 的 Apache Commons 链库的实际场景——你可以在这里阅读更多关于[的内容。](https://web.archive.org/web/20221112032904/https://commons.apache.org/proper/commons-chain/cookbook.html)

和往常一样，这篇文章的代码可以在 Github 的[上找到。](https://web.archive.org/web/20221112032904/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons)