# 跳马入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vault>

## 1.概观

在本教程中，我们将探索 Hashicorp 的 Vault–**这是一个流行的工具，用于在现代应用架构中安全地管理敏感信息**。

我们将讨论的主要话题包括:

*   Vault 试图解决什么问题
*   金库的架构和主要概念
*   简单测试环境的设置
*   使用命令行工具与 Vault 交互

## 2.敏感信息的问题是

在深入研究 Vault 之前，让我们试着理解它试图解决的问题:敏感信息管理。

**大多数应用程序需要访问敏感数据才能正常工作**。例如，电子商务应用程序可能在某处配置了用户名/密码，以便连接到其数据库。它可能还需要 API 密钥来与其他服务提供商集成，例如支付网关、物流和其他业务合作伙伴。

数据库凭证和 API 密钥是一些敏感信息的例子，我们需要以一种安全的方式存储这些敏感信息，并使它们对我们的应用程序可用。

一个简单的解决方案是将这些凭证存储在配置文件中，并在启动时读取它们。然而，这种方法的问题是显而易见的。任何可以访问这个文件的人都可以共享我们的应用程序所拥有的相同的数据库特权。

我们可以通过加密这些文件来让事情变得更难一些。然而，这种方法在整体安全性方面不会增加太多。主要是因为我们的应用程序必须能够访问主密钥。当以这种方式使用加密时，只会获得一种“虚假的”安全感。

现代应用程序和云环境往往会增加一些额外的复杂性:分布式服务、多个数据库、消息传递系统等等，所有这些都将敏感信息散布在各处，从而增加了安全漏洞的风险。

那么，我们能做什么呢？我们跳马吧！

## 3.什么是跳马？

Hashicorp Vault 解决了管理敏感信息的问题——用 Vault 的说法就是`secret`。**“管理”在本文中是指 Vault 控制敏感信息**的所有方面:它的生成、存储、使用，以及最后但同样重要的撤销。

Hashicorp 提供两种版本的 Vault。本文中使用的开源版本可以免费使用，甚至可以在商业环境中使用。还提供付费版本，包括不同 SLA 的技术支持和附加功能，如 HSM(硬件安全模块)支持。

### 3.1.架构和主要特性

金库的建筑简单得令人难以置信。它的主要组成部分是:

*   持久性后端——存储所有秘密
*   处理客户端请求并对机密执行操作的 API 服务器
*   数量`secret engines, `每种支持的保密类型一个

通过将所有秘密处理委托给 Vault，我们可以缓解一些安全问题:

*   我们的应用程序不再需要存储它们，只需在需要时询问 Vault 并将其丢弃即可
*   我们可以使用短期秘密，从而限制攻击者使用窃取的秘密的“机会之窗”

在将数据写入存储之前，Vault 会使用加密密钥对所有数据进行加密。这个加密密钥由另一个密钥加密，即仅在启动时使用的主密钥。

Vault 实现中的一个关键点是，它不在服务器中存储主密钥。**这意味着启动后，即使是 Vault 也无法访问其保存的数据。**此时，一个存储库实例被称为处于“密封”状态。

稍后，我们将介绍生成主密钥和解封一个 Vault 实例所需的步骤。

一旦解封，Vault 将准备好接受 API 请求。当然，这些请求需要身份验证，这就为我们介绍了 Vault 如何对客户端进行身份验证，并决定它们能做什么或不能做什么。

### 3.2.证明

**要访问 Vault 中的机密，客户端需要使用一种受支持的方法进行身份验证**。最简单的方法是使用令牌，令牌是在每个 API 请求上使用特殊的 HTTP 头发送的字符串。

最初安装时，Vault 会自动生成一个“根令牌”。这个令牌相当于 Linux 系统中的 root 超级用户，所以应该尽量少用。作为一种最佳做法，我们应该使用这个根令牌来创建具有更少权限的其他令牌，然后撤销它。不过，这不是问题，因为我们稍后可以使用解封密钥生成另一个根令牌。

Vault 还支持其他身份验证机制，如 LDAP、JWT、TLS 证书等。所有这些机制都建立在基本令牌机制之上:一旦 Vault 验证了我们的客户端，它将提供一个令牌，然后我们可以使用它来访问其他 API。

令牌有一些相关的属性。主要属性有:

*   一组相关联的`Policies` (见下一节)
*   生存时间
*   是否可以续签
*   最大使用次数

除非另有说明，否则 Vault 创建的令牌将形成父子关系。子令牌最多可以拥有与其父令牌相同级别的权限。

反之则不然:我们可以——并且通常会——用限制性策略创建一个子令牌关于这种关系的另一个要点:**当我们使一个令牌无效时，所有子令牌及其后代也将无效**。

### 3.3.政策

**策略准确定义了客户端可以访问哪些机密以及可以对其执行哪些操作**。让我们看看一个简单的策略是什么样子的:

```
path "secret/accounting" {
    capabilities = [ "read" ]
}
```

这里我们使用了 HCL (Hashicorp 的配置语言)语法来定义我们的策略。为此，Vault 也支持 JSON，但是在我们的例子中我们将坚持使用 HCL，因为它更容易阅读。

**保险库中的策略是“默认拒绝”**。附加到这个示例策略的令牌将可以访问存储在`secret/accounting`下的秘密，除此之外别无其他。在创建时，一个令牌可以附加到多个策略。这非常有用，因为它允许我们创建和测试更小的策略，然后根据需要应用它们。

策略的另一个重要方面是它们利用了懒惰评估。**这意味着我们可以更新给定的策略，所有令牌将立即受到影响。**

到目前为止描述的策略也称为访问控制列表策略，或 ACL 策略。Vault 还支持另外两种策略类型:EGP 策略和 RGP 策略。这些仅在付费版本中可用，并在 [Sentinel](https://web.archive.org/web/20220626083749/https://www.vaultproject.io/docs/enterprise/sentinel/index.html) 支持下扩展了基本策略语法。

如果可行，这允许我们在策略中考虑额外的属性，如一天中的时间、多重身份验证因素、客户端网络来源等。例如，我们可以定义一个策略，只允许在工作时间访问给定的秘密。

我们可以在 [Vault 的文档](https://web.archive.org/web/20220626083749/https://www.vaultproject.io/docs/concepts/policies.html)中找到关于策略语法的更多细节。

## 4.秘密类型

Vault 支持一系列不同的密码类型，可满足不同的使用情况:

*   `Key-Value:`简单的静态键值对
*   `Dynamically generated credentials`:Vault 应客户要求生成
*   `Cryptographic keys`:用于对客户端数据执行加密功能

每种机密类型都由以下属性定义:

*   定义其 REST API 前缀的`mount` `point,`
*   通过相应的 API 公开的一组操作
*   一组配置参数

给定的 secret 实例可以通过`path`访问，很像文件系统中的目录树。**该路径的第一个组成部分对应于所有这类秘密所在的`mount point`**。

例如，字符串`secret/my-application `对应于我们可以找到`my-application`的键值对的路径。

### 4.1.键值机密

**顾名思义，键值秘密是给定路径下可用的简单对**。例如，我们可以将对`foo=bar`存储在路径`/secret/my-application. `下

稍后，我们使用相同的路径来检索相同的一对或多对——多个对可以存储在相同的路径下。

保管库支持三种键值机密:

*   `Non-versioned Key-Pairs`，其中更新替换现有值
*   保持可配置数量的旧版本
*   `Cubbyhole`，一种特殊类型的非版本化密钥对，其值的范围是给定的`access token`(稍后将详细介绍)。

键值机密本质上是静态的，因此没有与之相关联的相关过期的概念。这种秘密的主要用例是存储访问外部系统的凭证，比如 API 密钥。

在这种情况下，凭证更新是一个半手动的过程，通常需要有人获取新的凭证，并使用 Vault 的命令行或其用户界面输入新值。

### 4.2.动态生成的秘密

**当应用程序**请求时，Vault 会即时生成动态机密。保管库支持几种类型的动态机密，包括以下几种:

*   数据库凭据
*   SSH 密钥对
*   X.509 证书
*   AWS 凭据
*   谷歌云服务账户
*   Active Directory 帐户

所有这些都遵循相同的使用模式。首先，我们用连接到相关服务所需的细节来配置秘密引擎。然后，我们定义一个或多个描述实际秘密创建的`roles, `。

让我们以数据库秘密引擎为例。首先，我们必须使用所有用户数据库连接的详细信息来配置 Vault，包括来自具有创建新用户的管理员权限的预先存在的用户的凭证。

然后，我们创建一个或多个角色(Vault 角色，而不是数据库角色)，其中包含用于创建新用户的实际 SQL 语句。这些语句通常不仅包括用户创建语句，还包括访问模式对象(表、视图等)所需的所有必需的`grant` 语句。

**当客户端访问相应的 API 时，Vault 将使用提供的语句在数据库中创建一个新的临时用户，并返回其凭证**。然后，客户端可以使用这些凭据在由所请求角色的生存时间属性定义的时间段内访问数据库。

一旦凭证过期，Vault 将自动撤销与该用户相关联的任何权限。客户端也可以请求 Vault 续订这些凭据。只有在特定数据库驱动程序支持并且相关策略允许的情况下，才会发生续订过程。

### 4.3.加密密钥

类型的秘密引擎处理加密功能，如加密、解密、签名等。**所有这些操作都使用由保险库**在内部生成和存储的密钥。除非明确告知，否则 Vault 永远不会公开给定的加密密钥。

关联的 API 允许客户端发送 Vault 纯文本数据并接收其加密版本。反过来也是可以的:我们可以发送加密的数据，拿回原文。

目前，这种类型的发动机只有一种:`Transit`发动机。该引擎支持流行的密钥类型，如 RSA 和 ECDSA，并且还支持`Convergent Encryption.`当使用这种模式时，给定的明文值总是导致相同的密文结果，这一属性在一些应用中非常有用。

例如，我们可以使用这种模式来加密事务日志表中的信用卡号。有了聚合加密，每次我们插入新交易时，加密的信用卡值都是相同的，因此允许使用常规 SQL 查询进行报告、搜索等。

## 5.保险库设置

在本节中，我们将创建一个本地测试环境，以便测试 Vault 的功能。

Vault 的部署很简单:只需[下载对应于我们的操作系统的包](https://web.archive.org/web/20220626083749/https://www.vaultproject.io/downloads.html)，并将其可执行文件(Windows 上的`vault `或`vault.exe `)解压到我们路径上的某个目录中。**这个可执行文件包含服务器，也是标准客户端**。[还有一张官方 Docker 图片可用](https://web.archive.org/web/20220626083749/https://hub.docker.com/_/vault/)，但我们在此不做赘述。

Vault 支持一个`development`模式，这对于一些快速测试和习惯它的命令行工具来说很好，但是对于真实的用例来说太简单了:**所有的数据在重启时都会丢失，API 访问使用普通的 HTTP** 。

相反，我们将使用基于文件的持久存储和设置 HTTPS，这样我们就可以研究一些可能导致问题的实际配置细节。

### 5.1.正在启动 Vault 服务器

Vault 使用 HCL 或 JSON 格式的配置文件。以下文件定义了使用文件存储和自签名证书启动服务器所需的所有配置:

```
storage "file" {
  path = "./vault-data"
}
listener "tcp" {
  address = "127.0.0.1:8200"
  tls_cert_file = "./src/test/vault-config/localhost.cert"
  tls_key_file = "./src/test/vault-config/localhost.key"
}
```

现在，让我们跑跳马。打开命令 shell，转到包含我们的配置文件的目录并运行以下命令:

```
$ vault server -config ./vault-test.hcl
```

Vault 将启动并显示一些初始化消息。它们将包括它的版本、一些配置细节和 API 可用的地址。就这样，我们的 Vault 服务器已经启动并正在运行。

### 5.2.保险库初始化

我们的 Vault 服务器现在正在运行，但是因为这是它的第一次运行，我们需要初始化它。

让我们打开一个新的 shell 并执行以下命令来实现这一点:

```
$ export VAULT_ADDR=https://localhost:8200
$ export VAULT_CACERT=./src/test/vault-config/localhost.cert
$ vault operator init
```

这里我们已经定义了几个环境变量，所以我们不必每次都将它们作为参数传递给 Vault:

*   URI 基地，我们的 API 服务器将在这里为请求提供服务
*   `VAULT_CACERT`:我们服务器的证书公钥路径

在我们的例子中，我们使用了`VAULT_CACERT`，这样我们就可以使用 HTTPS 来访问 Vault 的 API。我们需要这个，因为我们正在使用自签名证书。这对于生产环境来说是不必要的，在生产环境中，我们通常可以访问 CA 签名的证书。

发出上述命令后，我们应该会看到类似这样的消息:

```
Unseal Key 1: <key share 1 value>
Unseal Key 2: <key share 2 value>
Unseal Key 3: <key share 3 value>
Unseal Key 4: <key share 4 value>
Unseal Key 5: <key share 5 value>

Initial Root Token: <root token value>

... more messages omitted
```

前五行是主密钥共享，我们稍后将使用它来解封 Vault 的存储。**请注意，Vault 仅在初始化期间显示主密钥共享，不会显示更多。** **记下并安全保存它们，否则服务器重启后我们将无法获取我们的秘密！**

另外，请注意`root token`，因为我们稍后会用到它。与解封密钥不同，**根令牌很容易在稍后生成**，所以一旦所有配置任务完成，销毁它是安全的。因为我们稍后将发出需要身份验证令牌的命令，所以现在让我们将根令牌保存在一个环境变量中:

```
$ export VAULT_TOKEN=<root token value> (Unix/Linux)
```

让我们看看我们的服务器状态，现在我们已经初始化了它，使用以下命令:

```
$ vault status
Key                Value
---                -----
Seal Type          shamir
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    0/3
Unseal Nonce       n/a
Version            0.10.4
HA Enabled         false
```

我们可以看到保险库仍然是密封的。我们还可以跟踪解封进度:“0/3”意味着 Vault 需要三个份额，但是到目前为止一个也没有。让我们向前迈进，向它提供我们的股份。

### 5.3.保险库解封

我们现在解封金库，这样我们就可以开始使用它的秘密服务。为了完成解封过程，我们需要提供五个密钥部分中的任意三个:

```
$ vault operator unseal <key share 1 value>
$ vault operator unseal <key share 2 value>
$ vault operator unseal <key share 3 value>
```

发出每个命令后，vault 将打印解封进度，包括需要多少份额。在发送最后一个密钥共享时，我们会看到如下消息:

```
Key             Value
---             -----
Seal Type       shamir
Sealed          false
... other properties omitted
```

在这种情况下,“Sealed”属性为“false ”,这意味着 Vault 已准备好接受命令。

## 6.测试保险库

在本节中，我们将使用两种受支持的机密类型来测试我们的 Vault 设置:Key/Value 和 Database。我们还将展示如何创建附加了特定策略的新令牌。

### 6.1.使用键/值机密

首先，让我们存储秘密的键-值对并读回它们。假设用于初始化 Vault 的命令 shell 仍然打开，我们使用以下命令将这些对存储在`secret/fakebank`路径下:

```
$ vault kv put secret/fakebank api_key=abc1234 api_secret=1a2b3c4d
```

现在，我们可以随时使用以下命令恢复这些对:

```
$ vault kv get secret/fakebank
======= Data =======
Key           Value
---           -----
api_key       abc1234
api_secret    1a2b3c4d 
```

这个简单的测试向我们展示了 Vault 的正常工作。我们现在可以测试一些额外的功能。

### 6.2.创建新令牌

到目前为止，我们已经使用了根令牌来认证我们的请求。由于根令牌`way`过于强大，因此使用特权较少、生存时间较短的令牌被认为是最佳实践。

让我们创建一个新令牌，我们可以像使用根令牌一样使用它，但它会在一分钟后过期:

```
$ vault token create -ttl 1m
Key                  Value
---                  -----
token                <token value>
token_accessor       <token accessor value>
token_duration       1m
token_renewable      true
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

让我们测试这个令牌，用它来读取我们之前创建的键/值对:

```
$ export VAULT_TOKEN=<token value>
$ vault kv get secret/fakebank
======= Data =======
Key           Value
---           -----
api_key       abc1234
api_secret    1a2b3c4d
```

如果我们等待一分钟并尝试重新发出该命令，我们会得到一条错误消息:

```
$ vault kv get secret/fakebank
Error making API request.

URL: GET https://localhost:8200/v1/sys/internal/ui/mounts/secret/fakebank
Code: 403\. Errors:

* permission denied
```

该消息表明我们的令牌不再有效，这是我们所期望的。

### 6.3.测试策略

我们在上一节中创建的示例令牌虽然寿命很短，但仍然非常强大。现在，让我们使用策略来创建更多受限令牌。

例如，让我们定义一个策略，只允许对我们之前使用的`secret/fakebank`路径进行读访问:

```
$ cat > sample-policy.hcl <<EOF
path "secret/fakebank" {
    capabilities = ["read"]
}
EOF
$ export VAULT_TOKEN=<root token>
$ vault policy write fakebank-ro ./sample-policy.hcl
Success! Uploaded policy: fakebank-ro
```

现在，我们使用以下命令创建一个包含此策略的令牌:

```
$ export VAULT_TOKEN=<root token>
$ vault token create -policy=fakebank-ro
Key                  Value
---                  -----
token                <token value>
token_accessor       <token accessor value>
token_duration       768h
token_renewable      true
token_policies       ["default" "fakebank-ro"]
identity_policies    []
policies             ["default" "fakebank-ro"]
```

正如我们之前所做的，让我们使用这个令牌来读取我们的秘密值:

```
$ export VAULT_TOKEN=<token value>
$ vault kv get secret/fakebank
======= Data =======
Key           Value
---           -----
api_key       abc1234
api_secret    1a2b3c4d
```

到目前为止，一切顺利。我们可以读取数据，正如所料。让我们看看当我们尝试更新这个秘密时会发生什么:

```
$ vault kv put secret/fakebank api_key=foo api_secret=bar
Error writing data to secret/fakebank: Error making API request.

URL: PUT https://127.0.0.1:8200/v1/secret/fakebank
Code: 403\. Errors:

* permission denied
```

由于我们的策略没有明确允许写入，Vault 返回 403–访问被拒绝状态代码。

### 6.4.使用动态数据库凭据

作为本文的最后一个例子，让我们使用 Vault 的数据库密码引擎来创建动态凭证。这里我们假设本地有一个可用的 MySQL 服务器，我们可以用“root”权限访问它。我们还将使用一个非常简单的模式，只包含一个表—`account`。

这里提供了用于创建该模式和特权用户的 SQL 脚本。

现在，让我们配置 Vault 以使用该数据库。默认情况下，数据库密码引擎是不启用的，因此我们必须先解决这个问题，然后才能继续:

```
$ vault secrets enable database
Success! Enabled the database secrets engine at: database/
```

我们现在创建一个数据库配置资源:

```
$ vault write database/config/mysql-fakebank \
  plugin_name=mysql-legacy-database-plugin \
  connection_url="{{username}}:{{password}}@tcp(127.0.0.1:3306)/fakebank" \
  allowed_roles="*" \
  username="fakebank-admin" \
  password="Sup&rSecre7;!"
```

路径前缀`database/config`是所有数据库配置必须存储的地方。我们选择名称`mysql-fakebank`,这样我们可以很容易地判断出这个配置指的是哪个数据库。至于配置密钥:

*   `plugin_name:`定义将使用哪个数据库插件。可用的插件名称在[库的文档](https://web.archive.org/web/20220626083749/https://www.vaultproject.io/docs/secrets/databases/index.html)中有描述
*   `connection_url`:这是插件连接数据库时使用的模板。请注意{ {用户名}}和{ {密码}}模板占位符。连接到数据库时，Vault 会用实际值替换这些占位符
*   `allowed_roles`:定义哪些保管库角色(接下来讨论)可以使用此配置。在我们的例子中，我们使用“*”，因此它适用于所有角色
*   `username & password:`这是 Vault 将用于执行数据库操作的帐户，例如创建新用户和撤销其权限

**金库数据库角色设置**

最后一项配置任务是创建 Vault 数据库角色资源，该资源包含创建用户所需的 SQL 命令。根据我们的安全需求，我们可以根据需要创建任意多的角色。

在这里，我们创建一个角色，授予对`fakebank`模式的所有表的只读访问权限:

```
$ vault write database/roles/fakebank-accounts-ro \
    db_name=mysql-fakebank \
    creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}';GRANT SELECT ON fakebank.* TO '{{name}}'@'%';" 
```

数据库引擎将路径前缀 `database/roles`定义为存储角色的位置。`fakebank-accounts-ro`是我们稍后在创建动态凭证时使用的角色名称。我们还提供以下密钥:

*   `db_name`:现有数据库配置的名称。对应于我们在创建配置资源时使用的路径的最后一部分
*   【Vault 将用于创建新用户的 SQL 语句模板列表

**创建动态凭证**

一旦我们准备好了数据库角色及其相应的配置，我们就可以使用以下命令生成新的动态凭证:

```
$ vault read database/creds/fakebank-accounts-ro
Key                Value
---                -----
lease_id           database/creds/fakebank-accounts-ro/0c0a8bef-761a-2ef2-2fed-4ee4a4a076e4
lease_duration     1h
lease_renewable    true
password           <password>
username           <username> 
```

前缀`database/creds`用于为可用角色生成凭证。由于我们已经使用了`fakebank-accounts-ro `角色，返回的用户名/密码将被限制在`select`操作中。

我们可以通过使用提供的凭据连接到数据库，然后执行一些 SQL 命令来验证这一点:

```
$ mysql -h 127.0.0.1 -u <username> -p fakebank
Enter password:
MySQL [fakebank]> select * from account;
... omitted for brevity
2 rows in set (0.00 sec)
MySQL [fakebank]> delete from account;
ERROR 1142 (42000): DELETE command denied to user 'v-fake-9xoSKPkj1'@'localhost' for table 'account' 
```

**我们可以看到第一个`select`成功完成，但是我们无法执行`delete`语句**。最后，如果我们等待一个小时，并尝试使用相同的凭据进行连接，我们将无法再连接到数据库。Vault 已自动撤销该用户的所有权限

## 7.结论

在本文中，我们探索了 Hashicorp 的 Vault 的基础知识，包括它试图解决的问题的一些背景知识、它的体系结构和基本用途。

在此过程中，我们创建了一个简单但功能强大的测试环境，将在后续文章中使用。

下一篇文章将介绍 Vault 的一个非常具体的用例:**在 Spring Boot 应用程序**的上下文中使用它。敬请期待！