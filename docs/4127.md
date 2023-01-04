# 以太网简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ethereumj>

## 1。简介

在本文中，我们来看一下[以太坊](https://web.archive.org/web/20221208143919/https://github.com/ethereum/ethereumj)库，它允许我们使用 Java 与[以太坊](https://web.archive.org/web/20221208143919/https://www.ethereum.org/)区块链进行交互。

首先，让我们简单地了解一下这项技术是怎么回事。

## 2。关于以太坊

[以太坊](https://web.archive.org/web/20221208143919/https://www.ethereum.org/)是一个`cryptocurrency`以可编程`blockchain`的形式利用分布式对等数据库，以太坊虚拟机(EVM)。它是同步的，并通过不同但相连的`nodes`来操作。

截止 2017 年， `Nodes`通过共识同步`blockchain`，通过挖矿造币(`proof of work`，验证交易，执行[固化](https://web.archive.org/web/20221208143919/https://solidity.readthedocs.io/en/develop/)中写入的`smart contracts` ，运行 EVM。

`blockchain`分为`blocks`，包含`account states`(包括`accounts`和`proof of work`之间的交易)。

## 3。 `Ethereum` 门面

类将许多 EthereumJ 包抽象并统一到一个易于使用的接口中。

可以连接到一个节点来与整个网络同步，一旦连接，我们就可以使用区块链。

创建外观对象非常简单:

```
Ethereum ethereum = EthereumFactory.createEthereum();
```

## 4。连接以太坊网络

为了连接到网络，我们必须首先连接到一个节点*，即*一个运行官方客户端的服务器。节点由`org.ethereum.net.rlpx.Node`类表示。

在成功建立到节点的连接后，`org.ethereum.listener.EthereumListenerAdapter`处理我们的客户端检测到的区块链事件。

### 4.1。连接以太坊网络

让我们连接到网络上的一个节点。这可以手动完成:

```
String ip = "http://localhost";
int port = 8345;
String nodeId = "a4de274d3a159e10c2c9a68c326511236381b84c9ec...";

ethereum.connect(ip, port, nodeId);
```

也可以使用 bean 自动连接到网络:

```
public class EthBean {
    private Ethereum ethereum;

    public void start() {
        ethereum = EthereumFactory.createEthereum();
        ethereum.addListener(new EthListener(ethereum));
    }

    public Block getBestBlock() {
        return ethereum.getBlockchain().getBestBlock();
    }

    public BigInteger getTotalDifficulty() {
        return ethereum.getBlockchain().getTotalDifficulty();
    }
}
```

然后我们可以将我们的`EthBean`注入到我们的应用程序配置中。然后，它会自动连接到以太坊网络，并开始下载区块链。

事实上，大多数连接处理都被方便地包装和抽象，只需向我们创建的`org.ethereum.facade.Ethereum`实例添加一个`org.ethereum.listener.EthereumListenerAdapter` 实例，就像我们在上面的 `start()` 方法中所做的那样:

```
EthBean eBean = new EthBean();
Executors.newSingleThreadExecutor().submit(eBean::start); 
```

### 4.2。使用监听器处理区块链

我们还可以子类化`EthereumListenerAdapter` 来处理客户端检测到的区块链事件。

为了完成这一步，我们需要创建我们的子类监听器:

```
public class EthListener extends EthereumListenerAdapter {

    private void out(String t) {
        l.info(t);
    }

    //...

    @Override
    public void onBlock(Block block, List receipts) {
        if (syncDone) {
            out("Net hash rate: " + calcNetHashRate(block));
            out("Block difficulty: " + block.getDifficultyBI().toString());
            out("Block transactions: " + block.getTransactionsList().toString());
            out("Best block (last block): " + ethereum
              .getBlockchain()
              .getBestBlock().toString());
            out("Total difficulty: " + ethereum
              .getBlockchain()
              .getTotalDifficulty().toString());
        }
    }

    @Override
    public void onSyncDone(SyncState state) {
        out("onSyncDone " + state);
        if (!syncDone) {
            out(" ** SYNC DONE ** ");
            syncDone = true;
        }
    }
} 
```

在接收到任何新的块(无论是旧的还是当前的)时，触发`onBlock()`方法。EthereumJ 使用``org.ethereum.core.Block`` 类来表示和处理块。

一旦同步完成，就会触发 `onSyncDone()`方法，更新本地以太坊数据。

## 5。与区块链合作

现在，我们可以连接到以太坊网络并直接使用区块链，我们将深入到我们经常使用的几个基本但非常重要的操作。

### 5.1。提交交易

现在，我们已经连接到区块链，我们可以提交交易。提交一个`Transaction`相对容易，但是创建一个实际的`Transaction`本身就是一个冗长的话题:

```
ethereum.submitTransaction(new Transaction(new byte[]));
```

### 5.2。访问`Blockchain`对象

`getBlockchain()`方法返回一个`Blockchain` facade 对象，带有获取当前网络困难的 getters 和特定的`Blocks`。

由于我们在第 4.3 节设置了`EthereumListener` ，我们可以使用上述方法访问区块链:

```
ethereum.getBlockchain(); 
```

### 5.3。返回以太坊账户地址

我们还可以返回一个以太坊`Address.`

要获得以太坊，我们首先需要在区块链上认证一对公钥和私钥。

让我们用新的随机密钥对创建一个新的密钥:

```
org.ethereum.crypto.ECKey key = new ECKey(); 
```

让我们从给定的私钥创建一个密钥:

```
org.ethereum.crypto.ECKey key = ECKey.fromPivate(privKey);
```

然后，我们可以使用我们的密钥来初始化一个`Account`。通过调用`.init()`，我们在`Account`对象上设置了一个`ECKey`和相关的`Address`:

```
org.ethereum.core.Account account = new Account();
account.init(key);
```

## 6。其他功能

该框架还提供了另外两个主要功能，我们不会在这里讨论，但是值得一提。

首先，我们有能力编制和执行 Solidity smart 合同。然而，在实体中创建契约，并随后编译和执行它们本身就是一个广泛的主题。

第二，尽管框架支持使用 CPU 进行有限的挖掘，但考虑到前者缺乏盈利能力，建议使用 GPU miner。

更多关于以太坊本身的高级话题可以在官方[文档](https://web.archive.org/web/20221208143919/https://www.ethereum.org/developers/)中找到。

## 7。结论

在这个快速教程中，我们展示了如何连接到以太坊网络以及使用区块链的几种重要方法。

和往常一样，本例中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143919/https://github.com/eugenp/tutorials/tree/master/ethereum)