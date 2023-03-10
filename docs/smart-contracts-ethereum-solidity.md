# 使用 Solidity 创建和部署智能合同

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/smart-contracts-ethereum-solidity>

## 1。概述

跑步的能力是以太坊区块链如此受欢迎和具有颠覆性的原因。

在我们解释什么是智能合同之前，让我们先来看一下`blockchain`的定义:

> 区块链是一个公共数据库，永久记录数字交易。它作为一个无信任的交易系统运行，在这个框架中，个人可以进行点对点交易，而不需要信任第三方或彼此。

让我们看看如何使用`solidity:`在以太坊上创建智能合约

## 2。以太坊

以太坊是一个平台，允许人们使用区块链技术高效地编写分散的应用程序。

分散式应用(`Dapp`)是一种工具，用于交互的不同方的人和组织在没有任何集中媒介的情况下走到一起。Dapps 的早期例子包括 BitTorrent(文件共享)和比特币(货币)。

我们可以将以太坊描述为内置编程语言的区块链。

### 2.1。以太坊虚拟机(EVM)

从实用的角度来看，EVM 可以被认为是一个包含数百万个对象的大型分散系统，称为`accounts`，它可以维护一个内部数据库，执行代码并相互对话。

第一种类型的帐户可能是使用网络的普通用户最熟悉的。其名称为`EOA`(外资账户)；它用于传递值(如[以太](https://web.archive.org/web/20220626080930/https://www.ethereum.org/))，由私钥控制。

另一方面，还有另一种类型的客户，即`contract.`让我们来看看这是怎么回事:

## 3。什么是智能合同？

一个`smart contract`是一个独立的脚本，通常**用 Solidity 编写，编译成`binary`或 JSON，部署到`blockchain`** 上的一个特定的`address`。同样，我们可以调用 RESTful API 的特定 URL 端点来通过`HttpRequest`执行一些逻辑，我们也可以通过提交正确的数据和必要的以太坊来调用部署和编译的 Solidity `function`，在特定的`address`执行部署的`smart contract`。

从商业的角度来看，**意味着`smart contract functions`可以内在地货币化**(类似于 AWS Lambda 函数，允许用户支付`per compute cycle`而不是`per instance`)。重要的是，`smart contract functions`不用花费以太坊就能运行。

简单地说，我们可以把一个`smart contract`看作存储在区块链网络中的代码集合，它定义了使用合同的所有各方都同意的条件。

这使得开发者能够创造出尚未被发明的东西。想一想——不需要中间人，也没有交易对手风险。我们可以创建新的市场，存储债务或承诺的注册信息，并确保我们拥有验证交易的网络共识。

任何人都可以将智能合约部署到分散的数据库，费用与包含代码的存储大小成比例。希望使用智能合同的节点必须以某种方式向网络的其余部分指示它们参与的结果。

### 3.1。坚实度

以太坊中使用的主要语言是[Solidity](https://web.archive.org/web/20220626080930/https://solidity.readthedocs.io/en/develop/)——这是一种专门为编写智能合同而开发的类似 Javascript 的语言。Solidity 是静态类型的，支持继承、库和复杂的用户定义类型等特性。

solidity 编译器将代码转换成 EVM 字节码，然后可以作为部署事务发送到以太网。这种部署比智能合约交互具有更大的交易费用，并且必须由合约所有者支付。

## 4。创建可靠的智能合同

solidity 契约中的第一行设置源代码版本。这是为了确保新的编译器版本不会突然改变契约的行为。

```java
pragma solidity ^0.4.0;
```

对于我们的示例，契约的名称是`Greeting`，正如我们所看到的，它的创建类似于 Java 或另一种面向对象编程语言中的类:

```java
contract Greeting {
    address creator;
    string message;

    // functions that interact with state variables
}
```

在这个例子中，我们声明了两个状态变量:`creator` 和`message`。在可靠性方面，我们使用名为`address` 的数据类型来存储账户地址。

接下来，我们需要在构造函数中初始化这两个变量。

### 4.1。构造器

我们通过使用关键字`function` 后跟契约名来声明一个构造函数(就像在 Java 中一样)。

构造函数是一个特殊的函数，当一个契约第一次被部署到以太坊区块链时，它只被调用一次。我们只能为一个协定声明一个构造函数:

```java
function Greeting(string _message) {
    message = _message;
    creator = msg.sender;
}
```

我们还将初始字符串`_message`作为参数注入到构造函数中，并将其设置为`message` 状态变量。

在构造函数的第二行中，我们将变量`creator`初始化为一个名为`msg.sender`的值。不需要在构造函数中注入`msg` 的原因是因为`msg` 是一个全局变量，它提供了关于消息的特定信息，比如发送消息的帐户地址。

我们可能会使用这些信息来实现对某些功能的访问控制。

### 4.2。Setter 和 Getter 方法

最后，我们实现了`message:`的 setter 和 getter 方法

```java
function greet() constant returns (string) {
    return message;
}

function setGreeting(string _message) {
    message = _message;
}
```

调用函数`greet` 将简单地返回当前保存的`message.` ,我们使用`constant`关键字来指定这个函数不修改契约状态，也不触发对区块链的任何写操作。

我们现在可以通过调用函数`setGreeting`来更改契约中状态的值。任何人都可以通过调用这个函数来改变这个值。这个方法没有返回类型，但是接受一个`String`类型作为参数。

现在我们已经创建了我们的第一个智能合约，下一步将把它部署到以太坊区块链，这样每个人都可以使用它。我们可以使用 [Remix](https://web.archive.org/web/20220626080930/https://remix.ethereum.org/) ，这是目前最好的在线 IDE，使用起来毫不费力。

## 5。与智能合同互动

为了与分散式网络(区块链)中的智能合约进行交互，我们需要访问其中一个客户端。

有两种方法可以做到这一点:

*   [自己运行客户端](https://web.archive.org/web/20220626080930/https://github.com/web3j/web3j#start-a-client)
*   使用类似于 [Infura](https://web.archive.org/web/20220626080930/https://infura.io/) 的服务连接到远程节点。

Infura 是最直接的选择，所以我们将请求一个[免费访问令牌](https://web.archive.org/web/20220626080930/https://infura.io/register)。一旦我们注册，我们需要选择 Rinkeby 测试网络的网址:`“https://rinkeby.infura.io/<token>”.`

为了能够从 Java 处理智能契约，我们需要使用一个名为 [Web3j](https://web.archive.org/web/20220626080930/https://web3j.io/) 的库。下面是 Maven 的依赖关系:

```java
<dependency>
    <groupId>org.web3j</groupId>
    <artifactId>core</artifactId>
    <version>3.3.1</version>
</dependency>
```

在格拉德:

```java
compile ('org.web3j:core:3.3.1')
```

在开始编写代码之前，我们需要先做一些事情。

### 5.1。创建钱包

Web3j 允许我们从命令行使用它的一些功能:

*   钱包创建
*   钱包密码管理
*   将资金从一个钱包转移到另一个钱包
*   生成可靠性智能合同函数包装

命令行工具可以以 zip 文件/tarball 的形式从项目资源库的 [releases](https://web.archive.org/web/20220626080930/https://github.com/web3j/web3j/releases/latest) 页面下载，或者对于 OS X 用户可以通过 homebrew:

```java
brew tap web3j/web3j
brew install web3j
```

要生成新的以太坊钱包，我们只需在命令行中键入以下内容:

```java
$ web3j wallet create
```

它会要求我们输入密码和存放钱包的位置。文件是 Json 格式的，需要记住的主要内容是以太坊地址。

我们将在下一步中使用它来请求乙醚。

### 5.2。在 Rinkeby 测试网络中请求乙醚

我们可以在这里申请免费的乙醚。为了防止恶意行为者耗尽所有可用资金，他们要求我们提供一个带有以太坊地址的社交媒体帖子的公共链接。

这是一个非常简单的步骤，他们几乎立刻就提供了乙醚，所以我们可以进行测试。

### 5.3。生成智能合同包装器

Web3j 可以自动生成智能合约包装器代码，以便在不离开 JVM 的情况下部署智能合约并与之交互。

为了生成包装器代码，我们需要编译我们的智能契约。我们可以在这里找到安装编译器[的指令。从那里，我们在命令行上键入以下内容:](https://web.archive.org/web/20220626080930/https://github.com/ethereum/go-ethereum/wiki/Contract-Tutorial#installing-a-compiler)

```java
$ solc Greeting.sol --bin --abi --optimize -o <output_dir>/
```

后者将创建两个文件:`Greeting.bin` 和`Greeting.abi.`现在，我们可以使用 web3j 的命令行工具生成包装器代码:

```java
$ web3j solidity generate /path/to/Greeting.bin 
  /path/to/Greeting.abi -o /path/to/src/main/java -p com.your.organisation.name
```

这样，我们现在就有了 Java 类来与主代码中的契约进行交互。

## 6。与智能合同互动

在我们的主类中，我们首先创建一个新的 web3j 实例来连接到网络上的远程节点:

```java
Web3j web3j = Web3j.build(
  new HttpService("https://rinkeby.infura.io/<your_token>"));
```

然后，我们需要加载以太坊钱包文件:

```java
Credentials credentials = WalletUtils.loadCredentials(
  "<password>",
 "/path/to/<walletfile>");
```

现在，让我们部署我们的智能合同:

```java
Greeting contract = Greeting.deploy(
  web3j, credentials,
  ManagedTransaction.GAS_PRICE, Contract.GAS_LIMIT,
  "Hello blockchain world!").send();
```

根据网络中的工作情况，部署合同可能需要一段时间。部署后，我们可能希望存储部署协定的地址。我们可以这样获得地址:

```java
String contractAddress = contract.getContractAddress();
```

该合同的所有交易都可以在 url 中看到:“`https://rinkeby.etherscan.io/address/<contract_address>”.`

另一方面，我们可以修改执行交易的智能合约的值:

```java
TransactionReceipt transactionReceipt = contract.setGreeting("Hello again").send();
```

最后，如果我们想查看存储的新值，我们可以简单地写:

```java
String newValue = contract.greet().send();
```

## 7。结论

在本教程中，我们看到 [Solidity](https://web.archive.org/web/20220626080930/https://solidity.readthedocs.io/en/develop/index.html) 是一种静态类型的编程语言，旨在开发运行在 EVM 上的智能合约。

我们还用这种语言创建了一个简单的契约，并发现它与其他编程语言非常相似。

`smart contract`只是一个用来描述可以促进价值交换的计算机代码的短语。在区块链上运行时，智能合约成为一个自我操作的计算机程序，在满足特定条件时自动执行。

我们在这篇文章中看到，在区块链中运行代码的能力是以太坊的主要区别，因为它允许开发人员构建一种新型的应用程序，这种应用程序远远超出了我们以前见过的任何应用程序。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220626080930/https://github.com/eugenp/tutorials/tree/master/ethereum/src/main/java/com/baeldung/web3j)