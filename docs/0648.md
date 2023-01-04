# 使用 Web3j 的轻量级以太坊客户端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/web3j>

 ![announcement-icon.png](img/120278fd77a75de419c2360fa607c671.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220525004302/https://www.baeldung.com/lightrun-n-security)

## 1。简介

本教程介绍 Web3j，它是流行的 Web3 抽象库的 Java 实现。

通过使用 JSON-RPC 或 HTTP、WebSockets、IPC 等熟悉的标准连接到以太坊节点，Web3j 用于与以太坊网络交互。

以太坊本身就是一个完整的主题，所以让我们先快速看一下它是什么！

## 2。以太坊

以太坊是用`Solidity.`写成的(1) `cryptocurrency`(令牌符号 [ETH](https://web.archive.org/web/20220525004302/https://coinmarketcap.com/currencies/ethereum/) )，(2)分布式超级计算机，(3)区块链，(4)智能合约网络

换句话说，以太坊(`network`)是由一堆名为`nodes`的连接服务器运行的，这些服务器以一种网状拓扑结构进行通信(从技术上来说，这并不完全正确，但足够接近，可以更好地理解它是如何工作的)。

`Web3j`，以及它的父库`Web3`，**允许`web applications`连接到其中一个`nodes`，从而提交以太坊** `**transactions**,`，它们实际上是先前已经部署到以太坊`network`的编译过的实体`smart contract` `functions`。关于智能合约的更多信息，请点击这里查看我们关于使用 Solidity [创建和部署智能合约的文章。](/web/20220525004302/https://www.baeldung.com/smart-contracts-ethereum-solidity)

每个节点将它的变化广播给所有其他节点`node`,以便达成共识和验证。因此，**每个`node`同时包含`Ethereum blockchain`的全部历史**，从而以防篡改的方式，并通过`network`中所有其他`node`的一致同意和验证，创建所有数据的冗余备份。\

有关以太坊的更多详细信息，请查看[官方页面](https://web.archive.org/web/20220525004302/https://www.ethereum.org/)。

## 3。设置

为了使用 Web3j 提供的全套特性，我们必须比平常做更多的设置。首先，Web3j 在几个独立的模块中提供，每个模块都可以有选择地添加到核心`pom.xml`依赖关系中:

```
<dependency>
    <groupId>org.web3j</groupId>
    <artifactId>core</artifactId>
    <version>3.3.1</version>
</dependency> 
```

请注意**Web3j 的团队提供了一个预建的 Spring Boot 启动器，内置了一些配置和有限的功能！**

在本文中，我们将把重点限制在核心功能上(包括如何将 Web3j 添加到 Spring MVC 应用程序中，从而获得与更广泛的 Spring webapps 的兼容性)。

这些模块的完整列表可以在 [Maven Central](https://web.archive.org/web/20220525004302/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22core%22%20AND%20g%3A%22org.web3j%22) 上找到。

### 3.1。汇编合同:松露还是土壤

编译和部署以太坊智能合约主要有两种方式。`solc`文件):

1.  官方 [Solidity](https://web.archive.org/web/20220525004302/https://solidity.readthedocs.io/en/v0.4.21/installing-solidity.html) 编译器。
2.  Truffle (用于测试、部署和管理智能合同的抽象套件)。

在本文中，我们将坚持使用块菌。 **Truffle 简化并抽象了编译智能合同**，迁移它们，并将它们部署到网络的过程。它还包装了`Solc`编译器，让我们在这两方面都获得了一些经验。

要设置 Truffle:

```
$ npm install truffle -g
$ truffle version
```

我们将分别使用四个关键命令来初始化我们的项目，编译我们的应用程序，将我们的应用程序部署到区块链，并分别进行测试:

```
$ truffle init
$ truffle compile
$ truffle migrate
$ truffle test
```

现在，让我们看一个简单的例子:

```
pragma solidity ^0.4.17;

contract Example {
  function Example() {
    // constructor
  }
} 
```

编译时应该会产生以下 ABI JSON:

```
{
  "contractName": "Example",
  "abi": [
    {
      "inputs": [],
      "payable": false,
      "stateMutability": "nonpayable",
      "type": "constructor"
    }
  ],
  "bytecode": "0x60606040523415600e57600080fd5b603580601b6...,
  "deployedBytecode": "0x6060604052600080fd00a165627a7a72305...,
  //...
}
```

然后，我们可以在应用程序中使用提供的字节码和 ABI 与部署的契约进行交互！

### 3.2。测试合同:加纳切

使用以太坊测试网最简单的方法之一是启动自己的 [Ganache](https://web.archive.org/web/20220525004302/https://github.com/trufflesuite/ganache) 服务器。我们将使用预构建的现成解决方案，因为它最容易设置和配置。它还为 Ganache CLI 提供了一个接口和服务器外壳，在底层驱动 Ganache。

我们可以通过默认提供的 URL 地址连接到我们的 Ganache 服务器:`http://localhost:8545 or http://localhost:7545.`

还有一些其他流行的方法来建立测试网络，包括使用[元掩码](https://web.archive.org/web/20220525004302/https://metamask.io/)、 [Infura](https://web.archive.org/web/20220525004302/https://infura.io/) 或 [Go-Lang 和 Geth](https://web.archive.org/web/20220525004302/https://github.com/ethereum/go-ethereum) 。

在本文中，我们将坚持使用 Ganache，因为设置自己的 GoLang 实例(并将其配置为自定义 testnet)可能会非常棘手，而且 Chrome 上的元掩码的状态目前还不确定。

我们可以将 Ganache 用于手动测试场景(当调试或完成我们的集成测试时)，或者将它们用于自动化测试场景(我们必须围绕这些场景构建我们的测试，因为在这种情况下，我们可能没有可用的端点)。

## 4。Web3 和 RPC

Web3 提供了与以太坊区块链和以太坊服务器节点轻松交互的门面和接口。换句话说， **Web3 通过 JSON-RPC 的方式促进了客户端和以太坊区块链**之间的互通。 **Web3J 是 [Web3](https://web.archive.org/web/20220525004302/https://github.com/ethereum/web3.js) 的官方 Java 端口。**

我们可以通过传入一个提供者(例如第三方或本地以太坊节点的端点)来初始化 Web3j，以便在我们的应用程序中使用:

```
Web3j web3a = Web3j.build(new HttpService());
Web3j web3b = Web3j.build(new HttpService("YOUR_PROVIDER_HERE"));
Web3j myEtherWallet = Web3j.build(
  new HttpService("https://api.myetherapi.com/eth"));
```

第三个选项显示了如何添加第三方提供者(从而连接到他们的以太坊节点)。但是我们也可以选择让我们的提供者选项为空。在这种情况下，将在`localhost` 上使用默认端口(`8545`，而不是`.`

## 5。基本 Web3 方法

现在我们知道了如何初始化我们的应用程序来与以太坊区块链进行通信，让我们来看看一些与以太坊区块链进行交互的核心方法。

用一个`CompleteableFuture`包装您的 Web3 方法是一个好策略，以处理对您配置的以太坊节点的 JSON-RPC 请求的异步性质。

### 5.1。当前块号

我们可以，例如，**返回当前块号**:

```
public EthBlockNumber getBlockNumber() {
    EthBlockNumber result = new EthBlockNumber();
    result = this.web3j.ethBlockNumber()
      .sendAsync()
      .get();
    return result;
}
```

### 5.2。账户

获取指定地址的**账号:**

```
public EthAccounts getEthAccounts() {
    EthAccounts result = new EthAccounts();
    result = this.web3j.ethAccounts()
        .sendAsync() 
        .get();
    return result;
}
```

### 5.3。账户交易笔数

要获得给定地址的**笔交易:**

```
public EthGetTransactionCount getTransactionCount() {
    EthGetTransactionCount result = new EthGetTransactionCount();
    result = this.web3j.ethGetTransactionCount(DEFAULT_ADDRESS, 
      DefaultBlockParameter.valueOf("latest"))
        .sendAsync() 
        .get();
    return result;
}
```

### 5.4。账户余额

最后，获取地址或钱包的当前余额:

```
public EthGetBalance getEthBalance() {
    EthGetBalance result = new EthGetBalance();
    this.web3j.ethGetBalance(DEFAULT_ADDRESS, 
      DefaultBlockParameter.valueOf("latest"))
        .sendAsync() 
        .get();
    return result;
}
```

## 6。在 Web3j 中使用合同

一旦我们使用 Truffle 编译了我们的 Solidity 契约，我们就可以使用独立的 Web3j 命令行工具来处理我们编译的 `Application Binary Interfaces` ( `ABI`),可以在这里[使用](https://web.archive.org/web/20220525004302/https://github.com/web3j/web3j/#command-line-tools)或者在这里作为独立的 zip 文件[。](https://web.archive.org/web/20220525004302/https://github.com/web3j/web3j/releases/download/v3.3.1/web3j-3.3.1.zip)

### 6.1.CLI 魔术

然后，我们可以使用以下命令自动生成 Java 智能契约包装器(本质上是一个公开智能契约 ABI 的 POJO ):

```
$ web3j truffle generate [--javaTypes|--solidityTypes] 
  /path/to/<truffle-smart-contract-output>.json 
  -o /path/to/src/main/java -p com.your.organisation.name
```

在项目的根目录下运行以下命令:

```
web3j truffle generate dev_truffle/build/contracts/Example.json 
  -o src/main/java/com/baeldung/web3/contract -p com.baeldung
```

生成了我们的`Example `类:

```
public class Example extends Contract {
    private static final String BINARY = "0x60606040523415600e576...";
    //...
}
```

### 6.2.Java POJO 的

现在我们有了智能契约包装器，**我们可以通过编程创建一个钱包，然后将我们的契约部署到地址**:

```
WalletUtils.generateNewWalletFile("PASSWORD", new File("/path/to/destination"), true);
```

```
Credentials credentials = WalletUtils.loadCredentials("PASSWORD", "/path/to/walletfile");
```

### 6.3.部署合同

我们可以像这样部署我们的合同:

```
Example contract = Example.deploy(this.web3j,
  credentials,
  ManagedTransaction.GAS_PRICE,
  Contract.GAS_LIMIT).send(); 
```

然后得到地址:

```
contractAddress = contract.getContractAddress();
```

### 6.4.发送交易

要使用我们的`Contract`的`Functions`发送一个`Transaction`，我们可以用输入值的`List`和输出参数的`List`初始化一个 Web3j `Function`:

```
List inputParams = new ArrayList();
List outputParams = new ArrayList();
Function function = new Function("fuctionName", inputParams, outputParams);
String encodedFunction = FunctionEncoder.encode(function); 
```

然后，我们可以用必要的`gas`(用于执行`Transaction`)和 nonce 参数初始化我们的`Transaction`:

```
BigInteger nonce = BigInteger.valueOf(100);
BigInteger gasprice = BigInteger.valueOf(100);
BigInteger gaslimit = BigInteger.valueOf(100);

Transaction transaction = Transaction
  .createFunctionCallTransaction("FROM_ADDRESS", 
    nonce, gasprice, gaslimit, "TO_ADDRESS", encodedFunction);

EthSendTransaction transactionResponse = web3j.ethSendTransaction(transaction).sendAsync().get();
transactionHash = transactionResponse.getTransactionHash(); 
```

有关智能合同功能的完整列表，请参见[官方文档](https://web.archive.org/web/20220525004302/https://docs.web3j.io/)。

## 7。结论

就是这样！**我们已经用 Web3j** 设置了一个 Java Spring MVC app 区块链时代到了！

和往常一样，本文中使用的代码示例可以在 [GitHub](https://web.archive.org/web/20220525004302/https://github.com/eugenp/tutorials/tree/master/ethereum/src/main/java/com/baeldung/web3j) 上找到。