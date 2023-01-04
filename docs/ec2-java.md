# 在 Java 中管理 EC2 实例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ec2-java>

## 1。概述

在本文中，我们将学习使用 Java SDK 来**控制 EC2 资源。如果你不熟悉 [EC2(弹性云计算)](https://web.archive.org/web/20220703143257/https://aws.amazon.com/ec2/)——这是一个在亚马逊的云中提供计算能力的平台。**

## 2。先决条件

使用 Amazon AWS SDK for EC2 所需的 Maven 依赖项、AWS 帐户设置和客户端连接与本文中的[相同。](/web/20220703143257/https://www.baeldung.com/aws-s3-java)

假设我们已经创建了前一篇文章中描述的`AWSCredentials,`的实例，我们可以继续创建我们的 EC2 客户端:

```
AmazonEC2 ec2Client = AmazonEC2ClientBuilder
  .standard()
  .withCredentials(new AWSStaticCredentialsProvider(credentials))
  .withRegion(Regions.US_EAST_1)
  .build();
```

## 3。创建 EC2 实例

使用 SDK，我们可以快速**设置启动我们的第一个 EC2 实例所需的东西。**

### 3.1。创建安全组

安全组控制我们 EC2 实例的网络流量。我们能够为几个 EC2 实例使用一个安全组。

让我们创建一个安全组:

```
CreateSecurityGroupRequest createSecurityGroupRequest = new CreateSecurityGroupRequest()
  .withGroupName("BaeldungSecurityGroup")
  .withDescription("Baeldung Security Group");
CreateSecurityGroupResult createSecurityGroupResult = ec2Client.createSecurityGroup(
  createSecurityGroupRequest);
```

由于安全组默认不允许任何网络流量，我们必须**配置我们的安全组来允许流量。**

让我们允许来自任何 IP 地址的 HTTP 流量:

```
IpRange ipRange = new IpRange().withCidrIp("0.0.0.0/0");
IpPermission ipPermission = new IpPermission()
  .withIpv4Ranges(Arrays.asList(new IpRange[] { ipRange }))
  .withIpProtocol("tcp")
  .withFromPort(80)
  .withToPort(80);
```

最后，我们必须**将 `ipRange` 实例附加到`AuthorizeSecurityGroupIngressRequest`** 上，并使用我们的 EC2 客户端发出请求:

```
AuthorizeSecurityGroupIngressRequest authorizeSecurityGroupIngressRequest 
  = new AuthorizeSecurityGroupIngressRequest()
  .withGroupName("BaeldungSecurityGroup")
  .withIpPermissions(ipPermission);
ec2Client.authorizeSecurityGroupIngress(authorizeSecurityGroupIngressRequest);
```

### 3.2。创建密钥对

当启动 EC2 实例时，我们需要指定一个[密钥对。](https://web.archive.org/web/20220703143257/https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) **我们可以使用 SDK 创建一个密钥对:**

```
CreateKeyPairRequest createKeyPairRequest = new CreateKeyPairRequest()
  .withKeyName("baeldung-key-pair");
CreateKeyPairResult createKeyPairResult = ec2Client.createKeyPair(createKeyPairRequest);
```

让我们得到私钥:

```
createKeyPairResult.getKeyPair().getKeyMaterial(); 
```

我们必须确保把这把钥匙放在安全的地方。如果我们丢失了它，我们将无法找回它(亚马逊不会保留它)。这是我们连接到 EC2 实例的唯一方式。

### 3.3。创建 EC2 实例

为了创建 EC2，我们将使用一个`RunInstancesRequest:`

```
RunInstancesRequest runInstancesRequest = new RunInstancesRequest()
  .withImageId("ami-97785bed")
  .withInstanceType("t2.micro") 
  .withKeyName("baeldung-key-pair") 
  .withMinCount(1)
  .withMaxCount(1)
  .withSecurityGroups("BaeldungSecurityGroup"); 
```

映像 Id 是这个实例将使用的 **[AMI 映像](https://web.archive.org/web/20220703143257/https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)。**

[实例类型](https://web.archive.org/web/20220703143257/https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html)定义了实例的**规格。**

**键名是可选的；**如果没有指定，那么我们不能连接到我们的实例。如果我们确信已经正确地设置了实例，并且不需要连接，这就很好。

最小和最大计数给出了将创建多少实例的界限。这取决于可用性区域:如果 AWS 不能在该区域中创建最少数量的实例，它将不会创建任何实例。

相反，如果 AWS 不能创建最大数量的实例，它将尝试创建更少的实例，前提是这个数量大于我们指定的最小实例数量。

现在，我们可以使用 runInstances()方法执行请求，并检索创建的实例的 id:

```
String yourInstanceId = ec2Client.runInstances(runInstancesRequest)
  .getReservation().getInstances().get(0).getInstanceId();
```

## 4。管理 EC2 实例

使用 SDK，我们可以**启动、停止、重启、描述和配置 EC2 实例的监控**。

### 4.1。启动、停止和重启 EC2 实例

启动、停止和重启一个实例相对简单。

启动实例:

```
StartInstancesRequest startInstancesRequest = new StartInstancesRequest()
  .withInstanceIds(yourInstanceId);

ec2Client.startInstances(request); 
```

停止实例: 

```
StopInstancesRequest stopInstancesRequest = new StopInstancesRequest()
  .withInstanceIds(yourInstanceId);

ec2Client.stopInstances(request); 
```

重新启动实例:

```
RebootInstancesRequest request = new RebootInstancesRequest()
  .withInstanceIds(yourInstanceId);

RebootInstancesResult rebootInstancesRequest = ec2Client.rebootInstances(request); 
```

从每个请求中，**可以询问实例的先前状态:**

```
ec2Client.stopInstances(stopInstancesRequest)
  .getStoppingInstances()
  .get(0)
  .getPreviousState()
  .getName()
```

### 4.2。监控 EC2 实例

让我们看看如何**开始和停止监控我们的 EC2 实例:**

```
MonitorInstancesRequest monitorInstancesRequest = new MonitorInstancesRequest()
  .withInstanceIds(yourInstanceId);

ec2Client.monitorInstances(monitorInstancesRequest);

UnmonitorInstancesRequest unmonitorInstancesRequest = new UnmonitorInstancesRequest()
  .withInstanceIds(yourInstanceId);

ec2Client.unmonitorInstances(unmonitorInstancesRequest); 
```

### 4.3。描述一个 EC2 实例

最后，我们可以**描述我们的 EC2 实例:**

```
DescribeInstancesRequest describeInstancesRequest
 = new DescribeInstancesRequest();
DescribeInstancesResult response = ec2Client
  .describeInstances(describeInstancesRequest); 
```

EC2 实例被分组到**预订**中。预订是用来创建一个或多个 EC2 实例的`StartInstancesRequest`调用:

```
response.getReservations()
```

从这里我们可以得到实际的实例。让我们来看看第一个预订中的第一个实例:

```
response.getReservations().get(0).getInstances().get(0)
```

现在，我们可以描述我们的实例:

```
// ...
.getImageId()
.getSubnetId()
.getInstanceId()
.getImageId()
.getInstanceType()
.getState().getName()
.getMonitoring().getState()
.getKernelId()
.getKeyName() 
```

## 5。结论

在这个快速教程中，我们展示了如何使用 Java SDK 管理 Amazon EC2 实例。

像往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220703143257/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-miscellaneous)