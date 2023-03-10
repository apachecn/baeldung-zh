# 阿帕奇 Geode 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-geode>

## 1。概述

[Apache Geode](https://web.archive.org/web/20220703150239/https://geode.apache.org/) 是一个支持缓存和数据计算的分布式内存数据网格。

在本教程中，我们将介绍 Geode 的关键概念，并使用其 Java 客户端浏览一些代码示例。

## 2。设置

首先，我们需要下载并安装 Apache Geode，并设置`gfsh `环境。为此，我们可以遵循 [Geode 官方指南](https://web.archive.org/web/20220703150239/https://geode.apache.org/docs/guide/16/getting_started/15_minute_quickstart_gfsh.html)中的说明。

其次，本教程将创建一些文件系统工件。因此，我们可以通过创建一个临时目录并从那里启动来隔离它们。

### 2.1。安装和配置

从我们的临时目录中，我们需要启动一个`Locator`实例:

```java
gfsh> start locator --name=locator --bind-address=localhost
```

**`Locators`负责一个 Geode `Cluster`、**不同成员之间的协调，我们可以在 JMX 上空进一步管理。

接下来，让我们启动一个`Server`实例来托管一个或多个数据`Region`:

```java
gfsh> start server --name=server1 --server-port=0
```

我们将`–server-port`选项设置为 0，这样 Geode 将选择任何可用的端口。但是如果我们忽略它，服务器将使用默认端口 40404。**服务器是作为长期进程运行的`Cluster` 的可配置成员，负责管理数据`Regions`** 。

最后，我们需要一个`Region`:

```java
gfsh> create region --name=baeldung --type=REPLICATE
```

**`Region` 是我们最终存储数据的地方。**

### 2.2。验证

在我们继续之前，让我们确保一切正常。

首先，让我们检查一下我们是否有我们的`Server `和`Locator`:

```java
gfsh> list members
 Name   | Id
------- | ----------------------------------------------------------
server1 | 192.168.0.105(server1:6119)<v1>:1024
locator | 127.0.0.1(locator:5996:locator)<ec><v0>:1024 [Coordinator]
```

接下来，我们有我们的`Region`:

```java
gfsh> describe region --name=baeldung
..........................................................
Name            : baeldung
Data Policy     : replicate
Hosting Members : server1

Non-Default Attributes Shared By Hosting Members  

 Type  |    Name     | Value
------ | ----------- | ---------------
Region | data-policy | REPLICATE
       | size        | 0
       | scope       | distributed-ack
```

此外，我们应该在文件系统的临时目录下有一些名为“locator”和“server1”的目录。

有了这个输出，我们知道我们已经准备好继续前进了。

## 3。Maven 依赖关系

现在我们有了一个正在运行的 Geode，让我们开始查看客户端代码。

为了在我们的 Java 代码中使用 Geode，我们需要将 [Apache Geode Java 客户端](https://web.archive.org/web/20220703150239/https://search.maven.org/search?q=a:geode-core)库添加到我们的`pom`:

```java
<dependency>
     <groupId>org.apache.geode</groupId>
     <artifactId>geode-core</artifactId>
     <version>1.6.0</version>
</dependency>
```

让我们从简单地在几个区域中存储和检索一些数据开始。

## 4。简单的存储和检索

让我们演示如何存储单个值、批量值以及自定义对象。

要开始在我们的“baeldung”区域中存储数据，让我们使用定位器连接到它:

```java
@Before
public void connect() {
    this.cache = new ClientCacheFactory()
      .addPoolLocator("localhost", 10334)
        .create();
    this.region = cache.<String, String> 
      createClientRegionFactory(ClientRegionShortcut.CACHING_PROXY)
        .create("baeldung");
}
```

### 4.1。保存单个值

现在，我们可以简单地存储和检索我们所在地区的数据:

```java
@Test
public void whenSendMessageToRegion_thenMessageSavedSuccessfully() {

    this.region.put("A", "Hello");
    this.region.put("B", "Baeldung");

    assertEquals("Hello", region.get("A"));
    assertEquals("Baeldung", region.get("B"));
}
```

### 4.2。一次保存多个值

我们还可以一次保存多个值，比如在尝试减少网络延迟时:

```java
@Test
public void whenPutMultipleValuesAtOnce_thenValuesSavedSuccessfully() {

    Supplier<Stream<String>> keys = () -> Stream.of("A", "B", "C", "D", "E");
    Map<String, String> values = keys.get()
        .collect(Collectors.toMap(Function.identity(), String::toLowerCase));

    this.region.putAll(values);

    keys.get()
        .forEach(k -> assertEquals(k.toLowerCase(), this.region.get(k)));
}
```

### 4.3。保存自定义对象

字符串是有用的，但是我们迟早需要存储自定义对象。

假设我们有一个客户记录，我们希望使用以下键类型来存储:

```java
public class CustomerKey implements Serializable {
    private long id;
    private String country;

    // getters and setters
    // equals and hashcode
}
```

和以下值类型:

```java
public class Customer implements Serializable {
    private CustomerKey key;
    private String firstName;
    private String lastName;
    private Integer age;

    // getters and setters 
}
```

要存储这些内容，需要几个额外的步骤:

第一，**他们应该实现`Serializable`。虽然这不是一个严格的要求，但是通过使它们成为`Serializable,` [Geode 可以更健壮地存储它们](https://web.archive.org/web/20220703150239/https://geode.apache.org/docs/guide/16/developing/data_serialization/data_serialization_options.html)。**

其次，**它们需要位于我们的应用程序的类路径以及我们的 Geode `Server`** 的类路径中。

为了将它们放到服务器的类路径中，让我们将它们打包，比如使用`mvn clean package`。

然后我们可以在新的`start server`命令中引用结果 jar:

```java
gfsh> stop server --name=server1
gfsh> start server --name=server1 --classpath=../lib/apache-geode-1.0-SNAPSHOT.jar --server-port=0
```

同样，我们必须从临时目录运行这些命令。

最后，让我们使用与创建“baeldung”区域相同的命令在`Server`上创建一个名为“baeldung-customers”的新`Region`:

```java
gfsh> create region --name=baeldung-customers --type=REPLICATE
```

在代码中，我们将像以前一样使用定位器，指定自定义类型:

```java
@Before
public void connect() {
    // ... connect through the locator
    this.customerRegion = this.cache.<CustomerKey, Customer> 
      createClientRegionFactory(ClientRegionShortcut.CACHING_PROXY)
        .create("baeldung-customers");
}
```

然后，我们可以像以前一样存储我们的客户:

```java
@Test
public void whenPutCustomKey_thenValuesSavedSuccessfully() {
    CustomerKey key = new CustomerKey(123);
    Customer customer = new Customer(key, "William", "Russell", 35);

    this.customerRegion.put(key, customer);

    Customer storedCustomer = this.customerRegion.get(key);
    assertEquals("William", storedCustomer.getFirstName());
    assertEquals("Russell", storedCustomer.getLastName());
}
```

## 5。区域类型

对于大多数环境，我们的区域会有多个副本或多个分区，这取决于我们的读写吞吐量需求。

到目前为止，我们已经使用了内存中的复制区域。让我们仔细看看。

### 5.1。复制区域

顾名思义， **a `Replicated Region` 在不止一个`Server`上维护其数据的副本。**我们来测试一下。

从工作目录中的`gfsh `控制台，我们再向集群添加一个名为`server2`的`Server`:

```java
gfsh> start server --name=server2 --classpath=../lib/apache-geode-1.0-SNAPSHOT.jar --server-port=0
```

还记得我们做“baeldung”的时候，用的是`–type=REPLICATE`。因此， **Geode 会自动将我们的数据复制到新服务器上。**

让我们通过停止`server1:`来验证这一点

```java
gfsh> stop server --name=server1
```

然后，让我们对“baeldung”区域执行一个快速查询。

如果数据复制成功，我们将得到结果:

```java
gfsh> query --query='select e.key from /baeldung.entries e'
Result : true
Limit  : 100
Rows   : 5

Result
------
C
B
A 
E
D
```

所以，看起来复制成功了！

向我们的区域添加一个副本可以提高数据可用性。而且，因为不止一个服务器可以响应查询，我们也将获得更高的读取吞吐量。

但是，**如果他们都坠毁了怎么办？由于这些是内存中的区域，数据将丢失`.`** 为此，我们可以使用`–type=REPLICATE_PERSISTENT`来代替，它在复制时也将数据存储在磁盘上。

### 5.2。分区区域

对于更大的数据集，我们可以通过配置 Geode 将一个区域分割成单独的分区或桶来更好地扩展系统。

让我们创建一个名为“baeldung-partitioned”的分区`Region` :

```java
gfsh> create region --name=baeldung-partitioned --type=PARTITION
```

添加一些数据:

```java
gfsh> put --region=baeldung-partitioned --key="1" --value="one"
gfsh> put --region=baeldung-partitioned --key="2" --value="two"
gfsh> put --region=baeldung-partitioned --key="3" --value="three"
```

并快速验证:

```java
gfsh> query --query='select e.key, e.value from /baeldung-partitioned.entries e'
Result : true
Limit  : 100
Rows   : 3

key | value
--- | -----
2   | two
1   | one
3   | three
```

然后，为了验证数据已经分区，让我们再次停止`server1`并重新查询:

```java
gfsh> stop server --name=server1
gfsh> query --query='select e.key, e.value from /baeldung-partitioned.entries e'
Result : true
Limit  : 100
Rows   : 1

key | value
--- | -----
2   | two
```

我们这次只取回了一些数据条目，因为该服务器只有一个数据分区，所以当`server1`被丢弃时，它的数据就丢失了。

但是如果我们既需要分区又需要冗余呢？ Geode 还支持[许多其他类型](https://web.archive.org/web/20220703150239/https://geode.apache.org/docs/guide/11/reference/topics/region_shortcuts_reference.html)。下面三个很方便:

*   `PARTITION_REDUNDANT` 分区`and`跨集群的不同成员复制我们的数据
*   `PARTITION_PERSISTENT`像`PARTITION`一样对数据进行分区，但是分区到磁盘，并且
*   给了我们三种行为。

## 6。对象查询语言

Geode 还支持对象查询语言或 OQL，这比简单的关键字查找功能更强大。有点像 SQL。

对于这个例子，让我们使用前面构建的“baeldung-customer”区域。

如果我们再增加几个客户:

```java
Map<CustomerKey, Customer> data = new HashMap<>();
data.put(new CustomerKey(1), new Customer("Gheorge", "Manuc", 36));
data.put(new CustomerKey(2), new Customer("Allan", "McDowell", 43));
this.customerRegion.putAll(data);
```

然后，我们可以使用`QueryService` 来查找名字为“Allan”的客户:

```java
QueryService queryService = this.cache.getQueryService();
String query = 
  "select * from /baeldung-customers c where c.firstName = 'Allan'";
SelectResults<Customer> results =
  (SelectResults<Customer>) queryService.newQuery(query).execute();
assertEquals(1, results.size());
```

## 7。功能

内存数据网格的一个更强大的概念是“将计算带到数据中”的想法。

简而言之，因为 Geode 是纯 Java，所以我们不仅可以轻松发送数据，还可以轻松发送对数据执行的逻辑。

这可能会让我们想起 SQL 扩展的概念，如 PL-SQL 或 Transact-SQL。

### 7.1。定义功能

为了定义 Geode 要做的工作单元，我们实现了 Geode 的`Function` 接口。

例如，假设我们需要将所有客户的名字都改为大写。

我们不需要查询数据并让我们的应用程序来完成这项工作，我们只需要实现`Function`:

```java
public class UpperCaseNames implements Function<Boolean> {
    @Override
    public void execute(FunctionContext<Boolean> context) {
        RegionFunctionContext regionContext = (RegionFunctionContext) context;
        Region<CustomerKey, Customer> region = regionContext.getDataSet();

        for ( Map.Entry<CustomerKey, Customer> entry : region.entrySet() ) {
            Customer customer = entry.getValue();
            customer.setFirstName(customer.getFirstName().toUpperCase());
        }
        context.getResultSender().lastResult(true);   
    }

    @Override
    public String getId() {
        return getClass().getName();
    }
}
```

**注意`getId `必须返回一个惟一的值，所以类名通常是一个很好的选择。**

`FunctionContext`包含我们所有的区域数据，因此我们可以对其进行更复杂的查询，或者像我们在这里所做的那样，对其进行变异。

而且`Function`还有很多比这更强的动力，所以去看看[的官方手册](https://web.archive.org/web/20220703150239/https://gemfire.docs.pivotal.io/95/geode/developing/function_exec/function_execution.html)，尤其是[的`getResultSender`法](https://web.archive.org/web/20220703150239/https://geode.apache.org/releases/latest/javadoc/org/apache/geode/cache/execute/FunctionContext.html#getResultSender--)。

### 7.2。展开功能

我们需要让 Geode 知道我们的功能，以便能够运行它。就像我们对自定义数据类型所做的那样，我们将对 jar 进行打包。

但是这一次，我们可以只使用`deploy`命令:

```java
gfsh> deploy --jar=./lib/apache-geode-1.0-SNAPSHOT.jar
```

### 7.3。执行功能

现在，我们可以使用`FunctionService:`从应用程序中执行`Function`

```java
@Test
public void whenExecuteUppercaseNames_thenCustomerNamesAreUppercased() {
    Execution execution = FunctionService.onRegion(this.customerRegion);
    execution.execute(UpperCaseNames.class.getName());
    Customer customer = this.customerRegion.get(new CustomerKey(1));
    assertEquals("GHEORGE", customer.getFirstName());
}
```

## 8。结论

在本文中，我们学习了`Apache Geode`生态系统的基本概念。我们研究了标准和定制类型、复制和分区区域以及 oql 和函数支持的简单 get 和 put。

和往常一样，所有这些样本都可以在 GitHub 上[找到。](https://web.archive.org/web/20220703150239/https://github.com/eugenp/tutorials/tree/master/apache-libraries)