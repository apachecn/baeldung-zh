# 用 cassandra spring boot starter 构建一个交易机器人

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cassandre-spring-boot-trading-bot>

## 1.概观

交易机器人是一种计算机程序，可以在不需要人工干预的情况下自动向市场或交易所下单。

在本教程中，我们将使用 [Cassandre](https://web.archive.org/web/20221126233607/https://github.com/cassandre-tech/cassandre-trading-bot) 创建一个简单的加密交易机器人，它会在我们认为最佳时机时生成头寸。

## 2.Bot 概述

交易的意思是“用一种物品交换另一种物品”。

在金融市场，它购买股票、期货、期权、掉期、债券，或者像我们的情况一样，购买大量的加密货币。这里的思路是在特定的价格买入加密货币，在更高的价格卖出，从而获利(即使在价格下跌的情况下，我们仍然可以利用空仓获利)。

我们将使用沙盒交换；沙盒是一个虚拟系统，我们在其中拥有“假”资产，我们可以在其中下单并接收报价。

首先，让我们看看我们要做什么:

*   将 cassandra spring boot starter 添加到我们的项目
*   添加连接到交换机所需的配置
*   创建策略:
    *   从交易所接收报价
    *   选择何时购买
    *   到了买入的时候，检查我们是否有足够的资产并建立头寸
    *   显示日志以查看头寸何时开仓/平仓以及我们获得了多少收益
*   对照历史数据进行测试，看看我们能否盈利

## 3.Maven 依赖性

让我们从添加必要的依赖项到我们的`pom.xml`开始，首先是[Cassandre spring boot starter](https://web.archive.org/web/20221126233607/https://search.maven.org/artifact/tech.cassandre.trading.bot/cassandre-trading-bot-spring-boot-starter):

```
<dependency>
    <groupId>tech.cassandre.trading.bot</groupId>
    <artifactId>cassandre-trading-bot-spring-boot-starter</artifactId>
    <version>4.2.1</version>
</dependency>
```

Cassandre 依靠 [XChange](https://web.archive.org/web/20221126233607/https://github.com/knowm/XChange) 连接到加密交换。在本教程中，我们将使用 [Kucoin XChange 库](https://web.archive.org/web/20221126233607/https://search.maven.org/artifact/org.knowm.xchange/xchange-kucoin):

```
<dependency>
    <groupId>org.knowm.xchange</groupId>
    <artifactId>xchange-kucoin</artifactId>
    <version>5.0.8</version>
</dependency>
```

我们也使用`[hsqld](https://web.archive.org/web/20221126233607/https://search.maven.org/artifact/org.hsqldb/hsqldb)`来存储数据:

```
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <version>2.5.2</version>
</dependency>
```

为了根据历史数据测试我们的交易机器人，我们还添加了测试用的[Cassandre spring boot starter](https://web.archive.org/web/20221126233607/https://search.maven.org/artifact/tech.cassandre.trading.bot/cassandre-trading-bot-spring-boot-starter-test):

```
<dependency>
    <groupId>tech.cassandre.trading.bot</groupId>
    <artifactId>cassandre-trading-bot-spring-boot-starter-test</artifactId>
    <version>4.2.1</version>
    <scope>test</scope>
</dependency>
```

## 4.配置

让我们编辑 create `application.properties`来设置我们的配置:

```
# Exchange configuration
cassandre.trading.bot.exchange.name=kucoin
[[email protected]](/web/20221126233607/https://www.baeldung.com/cdn-cgi/l/email-protection)ail.com
cassandre.trading.bot.exchange.passphrase=cassandre
cassandre.trading.bot.exchange.key=6054ad25365ac6000689a998
cassandre.trading.bot.exchange.secret=af080d55-afe3-47c9-8ec1-4b479fbcc5e7

# Modes
cassandre.trading.bot.exchange.modes.sandbox=true
cassandre.trading.bot.exchange.modes.dry=false

# Exchange API calls rates (ms or standard ISO 8601 duration like 'PT5S')
cassandre.trading.bot.exchange.rates.account=2000
cassandre.trading.bot.exchange.rates.ticker=2000
cassandre.trading.bot.exchange.rates.trade=2000

# Database configuration
cassandre.trading.bot.database.datasource.driver-class-name=org.hsqldb.jdbc.JDBCDriver
cassandre.trading.bot.database.datasource.url=jdbc:hsqldb:mem:cassandre
cassandre.trading.bot.database.datasource.username=sa
cassandre.trading.bot.database.datasource.password= 
```

该配置有四个类别:

*   **交换配置**:我们为自己设置的交换凭证连接到 Kucoin 上的[现有沙盒](https://web.archive.org/web/20221126233607/https://trading-bot.cassandre.tech/ressources/how-tos/how-to-create-a-kucoin-account.html)账户
*   **模式**:我们想要使用的模式。在我们的例子中，我们要求 Cassandre 使用沙盒数据
*   **交易所 API 调用率**:表示我们希望从交易所检索数据(账户、订单、交易和报价器)的速度。要小心；所有的交易所都有我们可以调用的最大利率
*   **数据库配置** : Cassandre 使用数据库存储头寸、订单&交易。对于本教程，我们将使用一个简单的`hsqld`内存数据库。当然，在生产中，我们应该使用持久数据库

现在让我们在测试目录的`application.properties`中创建相同的文件，但是我们将`cassandre.trading.bot.exchange.modes.dry`改为`true`，因为在测试期间，我们不想向沙箱发送真正的命令。我们只想模拟它们。

## 5.战略

交易策略是为实现盈利回报而设计的固定计划；我们可以通过添加一个用 `@CassandreStrategy` 注释的 Java 类并扩展*basicassandrestragety*来创建自己的类。

让我们在 `MyFirstStrategy.java` : 中创建我们的策略类

```
@CassandreStrategy
public class MyFirstStrategy extends BasicCassandreStrategy {

    @Override
    public Set<CurrencyPairDTO> getRequestedCurrencyPairs() {
        return Set.of(new CurrencyPairDTO(BTC, USDT));
    }

    @Override
    public Optional<AccountDTO> getTradeAccount(Set<AccountDTO> accounts) {
        return accounts.stream()
          .filter(a -> "trade".equals(a.getName()))
          .findFirst();
    }
}
```

实现`BasicCassandreStrategy`迫使我们实现两个方法`getRequestedCurrencyPairs()` & `getTradeAccount()`:

在`getRequestedCurrencyPairs()`中，我们必须返回我们希望从交易所接收的货币对更新列表。货币对是两种不同货币的报价，其中一种货币的价值是对另一种货币的报价。在我们的例子中，我们希望与 BTC/USDT 合作。

为了更清楚起见，我们可以使用下面的`curl`命令手动检索一个 ticker:

```
curl -s https://api.kucoin.com/api/v1/market/orderbook/level1?symbol=BTC-USDT
```

我们会得到这样的结果:

```
{
  "time": 1620227845003,
  "sequence": "1615922903162",
  "price": "57263.3",
  "size": "0.00306338",
  "bestBid": "57259.4",
  "bestBidSize": "0.00250335",
  "bestAsk": "57260.4",
  "bestAskSize": "0.01"
}
```

价格值表明 1 BTC 值 57263.3 USDT。

我们必须实现的另一个方法是`getTradeAccount()`。在交易所，我们通常有几个账户，Cassandre 需要知道哪个账户是交易账户。为此，我们必须实现`getTradeAccount()`方法，该方法将我们拥有的账户列表作为参数提供给 usw，从该列表中，我们必须返回我们想要用于交易的账户。

在我们的例子中，我们在交易所的交易账户被命名为`“trade”`，所以我们简单地返回它。

## 6.创建职位

要获得新数据的通知，我们可以覆盖`BasicCassandreStrategy`的以下方法:

*   `onAccountUpdate()`接收账户更新
*   接收新的报价机
*   `onOrderUpdate()`接收订单更新
*   `onTradeUpdate()`)接收交易的最新信息
*   `onPositionUpdate()`接收有关职位的更新
*   `onPositionStatusUpdate()`接收有关职位状态变化的更新

对于本教程，我们将实现一个哑算法:**我们检查收到的每一个新的 ticker。如果 1 BTC 的价格跌破 56 000 USDT，我们认为是时候买**了。

为了使收益计算、订单、交易和平仓变得更容易，Cassandre 提供了一个自动管理头寸的类。

要使用它，第一步是借助于`PositionRulesDTO`类为职位创建规则，例如:

```
PositionRulesDTO rules = PositionRulesDTO.builder()
  .stopGainPercentage(4f)
  .stopLossPercentage(25f)
  .create();
```

然后，让我们使用该规则创建职位:

```
createLongPosition(new CurrencyPairDTO(BTC, USDT), new BigDecimal("0.01"), rules);
```

此时，卡珊德拉将创建一个 0.01 BTC 的买单。头寸状态将为“未平仓”,当所有相应的交易到达时，状态将变为“未平仓”。从现在开始，对于收到的每一个报价，Cassandre 将自动计算新的价格，如果在这个价格平仓将触发我们的两个规则之一(4%止盈或 25%止损)。

如果触发了一个规则，Cassandre 将自动创建一个 0.01 BTC 的卖单。头寸状态将移动到`CLOSING`，当所有对应的交易到达时，状态将移动到`CLOSED`。

这是我们将拥有的代码:

```
@Override
public void onTickerUpdate(TickerDTO ticker) {
    if (new BigDecimal("56000").compareTo(ticker.getLast()) == -1) {

        if (canBuy(new CurrencyPairDTO(BTC, USDT), new BigDecimal("0.01"))) {
            PositionRulesDTO rules = PositionRulesDTO.builder()
              .stopGainPercentage(4f)
              .stopLossPercentage(25f)
              .build();
            createLongPosition(new CurrencyPairDTO(BTC, USDT), new BigDecimal("0.01"), rules);
        }
    }
}
```

总而言之:

*   对于每一个新的股票，我们检查价格是否低于 56000。
*   如果我们的贸易账户上有足够的 USDT，我们就为 0.01 的 BTC 建仓。
*   从现在开始，对于每个股票:
    *   如果根据新价格计算出的收益超过 4%或 25%，卡珊德拉将平仓我们通过卖出 BTC 买入的 0.01 英镑建立的头寸。

## 7.跟踪日志中的位置演变

我们将最终实现`onPositionStatusUpdate()`来查看何时开仓/平仓:

```
@Override
public void onPositionStatusUpdate(PositionDTO position) {
    if (position.getStatus() == OPENED) {
        logger.info("> New position opened : {}", position.getPositionId());
    }
    if (position.getStatus() == CLOSED) {
        logger.info("> Position closed : {}", position.getDescription());
    }
}
```

## 8.回溯测试

简而言之，回溯测试策略就是测试前期交易策略的过程。Cassandre 交易机器人允许我们模拟机器人对历史数据的反应。

第一步是将我们的历史数据(CSV 或 TSV 文件)放在我们的`src/test/resources`文件夹中。

如果我们在 Linux 下，这里有一个简单的脚本来生成它们:

```
startDate=`date --date="3 months ago" +"%s"`
endDate=`date +"%s"`
curl -s "https://api.kucoin.com/api/v1/market/candles?type=1day&symbol;=BTC-USDT&startAt;=${startDate}&endAt;=${endDate}" \
| jq -r -c ".data[] | @tsv" \
| tac $1 > tickers-btc-usdt.tsv
```

它将创建一个名为`tickers-btc-usdt.tsv`的文件，包含从`startDate` (3 个月前)到`endDate`(现在)的 BTC-USDT 的历史汇率。

第二步是创建我们的虚拟账户余额，以模拟我们想要投资的资产的确切金额。

在这些文件中，我们为每个账户设置了每种加密货币的余额。例如，这是 user-trade.csv 的内容，它模拟了我们的贸易帐户资产:

该文件也必须在`src/test/resources`文件夹中。

```
BTC 1
USDT 10000
ETH 10
```

现在，我们可以添加一个测试:

```
@SpringBootTest
@Import(TickerFluxMock.class)
@DisplayName("Simple strategy test")
public class MyFirstStrategyLiveTest { 
```

```
 @Autowired
    private MyFirstStrategy strategy;

    private final Logger logger = LoggerFactory.getLogger(MyFirstStrategyLiveTest.class);

    @Autowired
    private TickerFluxMock tickerFluxMock;

    @Test
    @DisplayName("Check gains")
    public void whenTickersArrives_thenCheckGains() {
        await().forever().until(() -> tickerFluxMock.isFluxDone());

        HashMap<CurrencyDTO, GainDTO> gains = strategy.getGains();

        logger.info("Cumulated gains:");
        gains.forEach((currency, gain) -> logger.info(currency + " : " + gain.getAmount()));

        logger.info("Position still opened :");
        strategy.getPositions()
          .values()
          .stream()
          .filter(p -> p.getStatus().equals(OPENED))
          .forEach(p -> logger.info(" - {} " + p.getDescription()));

        assertTrue(gains.get(USDT).getPercentage() > 0);
    }

}
```

来自 `TickerFluxMock`的 `@Import` 将从我们的`src/test/resources`文件夹中加载历史数据，并将它们发送给我们的策略。然后，我们使用`await()`方法来确保所有从文件加载的 tickers 都已经发送到我们的策略。我们通过显示已平仓头寸、仍未平仓的头寸和整体收益来结束。

## 9.结论

本教程演示了如何创建一个与加密交换交互的策略，并根据历史数据对其进行测试。

当然，我们的算法很简单；在现实生活中，目标是找到一个有前途的技术，一个好的算法，好的数据来知道我们什么时候可以创建一个位置。例如，我们可以使用技术分析，因为 Cassandre 整合了`[ta4j.](https://web.archive.org/web/20221126233607/https://github.com/ta4j/ta4j)`

这篇文章的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221126233607/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-cassandre)