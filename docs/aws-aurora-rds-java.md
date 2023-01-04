# 使用 Java 的 AWS Aurora RDS 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aws-aurora-rds-java>

## 1。简介

**Amazon Aurora 是一款兼容 MySQL 和 PostgreSQL 的[关系数据库](https://web.archive.org/web/20220628145323/https://aws.amazon.com/relational-database/)，专为云**打造，将高端商业数据库的性能和可用性与开源数据库的简单性和成本效益相结合。

在本教程中，我们将介绍如何用 Java 创建 Amazon RDS 实例并与之交互，我们还将在 Amazon RDS 上连接和执行 SQL 测试。

让我们从设置项目开始。

## 2。Maven 依赖关系

让我们创建一个 Java Maven 项目并将 AWS SDK 添加到我们的项目中:

```java
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk</artifactId>
    <version>1.11.377</version>
</dependency>
```

要查看最新版本，请查看 [Maven Central](https://web.archive.org/web/20220628145323/https://search.maven.org/search?q=g:com.amazonaws%20AND%20a:aws-java-sdk&core=gav) 。

## 3。先决条件

要使用 AWS SDK，我们需要进行一些设置:

*   AWS 帐户
*   AWS 安全凭据
*   选择 AWS 区域

我们需要一个亚马逊网络服务账户。如果你还没有，那就去[创建一个账户](https://web.archive.org/web/20220628145323/https://portal.aws.amazon.com/gp/aws/developer/registration/index.html)

**AWS 安全凭证** **是允许我们对 AWS API 动作**进行编程调用的访问键。我们可以通过两种方式获得这些凭证，或者从[安全凭证](https://web.archive.org/web/20220628145323/https://console.aws.amazon.com/iam/home#security_credential)页面的访问密钥部分使用 AWS root 帐户凭证，或者从 [IAM 控制台](https://web.archive.org/web/20220628145323/https://console.aws.amazon.com/iam/home)使用 IAM 用户凭证

**我们必须选择一个 AWS 区域**来存储我们的亚马逊 RDS。请记住，RDS 价格因地区而异。更多细节，请查看[官方文件](https://web.archive.org/web/20220628145323/https://aws.amazon.com/rds/aurora/pricing/)。

对于本教程，我们将使用亚太地区(悉尼)(地区`ap-southeast-2`)。

## 4.连接到 AWS RDS 服务

首先，我们需要创建一个客户端连接来访问 Amazon RDS web 服务。

为此，我们将使用`AmazonRDS `接口:

```java
AWSCredentials credentials = new BasicAWSCredentials(
  "<AWS accesskey>", 
  "<AWS secretkey>"
); 
```

然后用适当的`region`和`credentials`配置`RDS Builder `:

```java
AmazonRDSClientBuilder.standard().withCredentials(credentials)
  .withRegion(Regions.AP_SOUTHEAST_2)
  .build(); 
```

## 5.亚马逊 Aurora 实例

现在让我们创建 Amazon Aurora RDS 实例。

### 5.1.创建`RDS`实例

要创建 RDS 实例，我们需要用以下属性实例化一个`CreateDBInstanceRequest `:

*   在 Amazon RDS 中所有现有实例名称中唯一的数据库实例标识符
*   DB 实例类指定 CPU、ECU、内存等的配置。，来自[实例类表](https://web.archive.org/web/20220628145323/https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.DBInstanceClass.html)
*   数据库引擎。PostgreSQL 或者 MySQL，我们就用 PostgreSQL
*   数据库主/超级用户名
*   数据库主用户口令
*   DB name 使用指定的名称创建初始数据库
*   对于存储类型，指定一个`Amazon EBS volume`类型。名单可在[这里](https://web.archive.org/web/20220628145323/https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)获得
*   GiB 中的存储分配

```java
CreateDBInstanceRequest request = new CreateDBInstanceRequest();
request.setDBInstanceIdentifier("baeldung");   
request.setDBInstanceClass("db.t2.micro");
request.setEngine("postgres");
request.setMultiAZ(false);
request.setMasterUsername("username");
request.setMasterUserPassword("password");
request.setDBName("mydb");       
request.setStorageType("gp2");   
request.setAllocatedStorage(10); 
```

现在让我们通过调用`createDBInstance()` : 来创建我们的第一个实例

```java
amazonRDS.createDBInstance(request); 
```

几分钟后将创建 RDS 实例。

我们不会在响应中获得端点 URL，因为这个调用是异步的。

### 5.2.列出数据库实例

在这一节中，我们将看到如何列出创建的数据库实例。

**要列出 RDS 实例，我们需要使用`AmazonRDS` 接口的`describeDBInstances`:**

```java
DescribeDBInstancesResult result = amazonRDS.describeDBInstances();
List<DBInstance> instances = result.getDBInstances();
for (DBInstance instance : instances) {
    // Information about each RDS instance
    String identifier = instance.getDBInstanceIdentifier();
    String engine = instance.getEngine();
    String status = instance.getDBInstanceStatus();
    Endpoint endpoint = instance.getEndpoint();
}
```

**端点** **URL 是我们新的 DB 实例**的连接 URL。连接到数据库时，此 URL 将作为主机提供。

### 5.3.运行 JDBC 测试

现在让我们连接 RDS 实例并创建第一个表。

让我们创建一个 db.properties 文件并添加数据库信息:

```java
db_hostname=<Endpoint URL>
db_username=username
db_password=password
db_database=mydb 
```

创建文件后，让我们连接到 RDS 实例并创建名为`jdbc_test`的表:

```java
Properties prop = new Properties();
InputStream input = AwsRdsDemo.class.getClassLoader().getResourceAsStream("db.properties");
prop.load(input);
String db_hostname = prop.getProperty("db_hostname");
String db_username = prop.getProperty("db_username");
String db_password = prop.getProperty("db_password");
String db_database = prop.getProperty("db_database"); 
```

```java
Connection conn = DriverManager.getConnection(jdbc_url, db_username, db_password);
Statement statement = conn.createStatement();
String sql = "CREATE TABLE IF NOT EXISTS jdbc_test (id SERIAL PRIMARY KEY, content VARCHAR(80))";
statement.executeUpdate(sql); 
```

**之后，我们将从表中插入和检索数据:**

```java
PreparedStatement preparedStatement = conn.prepareStatement("INSERT INTO jdbc_test (content) VALUES (?)");
String content = "" + UUID.randomUUID();
preparedStatement.setString(1, content);
preparedStatement.executeUpdate(); 
```

```java
String sql = "SELECT  count(*) as count FROM jdbc_test";
ResultSet resultSet = statement.executeQuery(sql);
while (resultSet.next()) {
    String count = resultSet.getString("count");
    Logger.log("Total Records: " + count);
} 
```

## 5.4.删除实例

要删除 DB 实例，我们需要生成`DeleteDBInstanceRequest.` **它需要 DB 实例标识符和`skipFinalSnapshot parameter.`**

`skipFinalSanpshot `用于指定我们是否要在删除实例之前拍摄快照:

```java
DeleteDBInstanceRequest request = new DeleteDBInstanceRequest();
request.setDBInstanceIdentifier(identifier);
request.setSkipFinalSnapshot(true);
DBInstance instance = amazonRDS.deleteDBInstance(request);
```

## 6.结论

在本文中，我们主要关注通过 Amazon SDK 与 Amazon Aurora (PostgreSQL) RDS 交互的基础知识。本教程重点介绍 PostgreSQL，也有其他选项，包括 MySQL。

尽管交互方法在 RDS 中保持不变。Aurora 是许多客户的首选，因为它比标准 MySQL 数据库快五倍，比标准 PostgreSQL 数据库快三倍。

欲了解更多信息，请访问亚马逊极光。

和往常一样，代码可以在 Github 上找到[。](https://web.archive.org/web/20220628145323/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-miscellaneous)