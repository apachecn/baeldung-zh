# 在 Java 中查找 IP 地址是否在指定的范围内

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-ip-address-range>

## 1.概观

在本教程中，我们将讨论如何使用 Java 来发现一个 [IP 地址](/web/20220625070344/https://www.baeldung.com/cs/ipv4-vs-ipv6)是否在给定的范围内。对于这个问题，在整篇文章中，我们将认为所有给定的 **IP 地址都是有效的 IPv4(互联网协议版本 4)和 IPv6(互联网协议版本 6)地址。**

## 2.问题简介

给定一个输入 IP 地址和另外两个 IP 地址作为范围(从和到)。我们应该能够确定输入的 [IP 地址是否在给定的范围内](/web/20220625070344/https://www.baeldung.com/cs/get-ip-range-from-subnet-mask)。

例如:

*   输入= 192.220.3.0，范围在 192.210.0.0 和 192.255.0.0 之间
    输出=真
*   输入= 192.200.0.0，范围在 192.210.0.0 和 192.255.0.0 之间
    输出=假

现在，让我们看看使用各种 Java 库检查给定 IP 地址是否在范围内的不同方法。

## 3.IP 地址库

Sean C Foley 编写的 [IPAddress](https://web.archive.org/web/20220625070344/https://seancfoley.github.io/IPAddress/) 库支持处理 IPv4 和 IPv6 地址，适用于多种[用例](https://web.archive.org/web/20220625070344/https://github.com/seancfoley/IPAddress/wiki/Code-Examples)。**需要注意的是，这个库至少需要 Java 8 才能运行。**

建立这个库很简单。我们需要将 [ipaddress](https://web.archive.org/web/20220625070344/https://mvnrepository.com/artifact/com.github.seancfoley/ipaddress) 依赖项添加到 pom.xml:

```java
<dependency>
    <groupId>com.github.seancfoley</groupId>
    <artifactId>ipaddress</artifactId>
    <version>5.3.3</version>
</dependency>
```

它提供了解决我们的问题所需的以下 Java 类:

*   `IPAddress,`将 IP 地址保存为 Java 实例
*   `IPAddressString`，从给定的 IP 以字符串形式构造 IPAddress 实例
*   `IPAddressSeqRange,` 代表任意范围的 IP 地址

现在，让我们看看使用上述类查找 IP 地址是否在给定范围内的代码:

```java
public static boolean checkIPIsInGivenRange (String inputIP, String rangeStartIP, String rangeEndIP) 
  throws AddressStringException {
    IPAddress startIPAddress = new IPAddressString(rangeStartIP).getAddress();
    IPAddress endIPAddress = new IPAddressString(rangeEndIP).getAddress();
    IPAddressSeqRange ipRange = startIPAddress.toSequentialRange(endIPAddress);
    IPAddress inputIPAddress = new IPAddressString(inputIP).toAddress();

    return ipRange.contains(inputIPAddress);
}
```

以上代码适用于 IPv4 和 IPv6 地址。`IPAddressString`参数化构造器以 IP 为字符串构造`IPAddress`实例。`IPAddressString`实例可以通过以下两种方法转换成`IPAddress`:

*   `toAddress()`
*   `getAddress()`

`getAddress() `方法假设给定的 IP 是有效的，但是`toAddress()`方法验证一次输入，如果无效就抛出`AddressStringException`。`IPAddress `类提供了一个`toSequentialRange `方法，该方法使用开始和结束 IP 范围来构造`IPAddressSeqRange`实例。

让我们考虑几个使用 IPv4 和 IPv6 地址调用`checkIPIsInGivenRange`的单元案例:

```java
@Test
void givenIPv4Addresses_whenIsInRange_thenReturnsTrue() throws Exception {
    assertTrue(IPWithGivenRangeCheck.checkIPIsInGivenRange("192.220.3.0", "192.210.0.0", "192.255.0.0"));
}

@Test
void givenIPv4Addresses_whenIsNotInRange_thenReturnsFalse() throws Exception {
    assertFalse(IPWithGivenRangeCheck.checkIPIsInGivenRange("192.200.0.0", "192.210.0.0", "192.255.0.0"));
}

@Test
void givenIPv6Addresses_whenIsInRange_thenReturnsTrue() throws Exception {
    assertTrue(IPWithGivenRangeCheck.checkIPIsInGivenRange(
      "2001:db8:85a3::8a03:a:b", "2001:db8:85a3::8a00:ff:ffff", "2001:db8:85a3::8a2e:370:7334"));
}

@Test
void givenIPv6Addresses_whenIsNotInRange_thenReturnsFalse() throws Exception {
    assertFalse(IPWithGivenRangeCheck.checkIPIsInGivenRange(
      "2002:db8:85a3::8a03:a:b", "2001:db8:85a3::8a00:ff:ffff", "2001:db8:85a3::8a2e:370:7334"));
}
```

## 4.公共知识产权数学

**[Commons IP Math](https://web.archive.org/web/20220625070344/https://github.com/jgonian/commons-ip-math) 库提供了表示 IPv4 和 IPv6 地址和范围的类。**它提供了用于处理最常见操作的 API，此外，它还提供了用于处理 IP 范围的比较器和其他实用程序。

我们需要将 [commons-ip-math](https://web.archive.org/web/20220625070344/https://mvnrepository.com/artifact/com.github.jgonian/commons-ip-math) 依赖项添加到 pom.xml 中:

```java
<dependency>
    <groupId>com.github.jgonian</groupId>
    <artifactId>commons-ip-math</artifactId>
    <version>1.32</version>
</dependency> 
```

### 4.1.对于 IPv4

该库提供了`Ipv4`和`Ipv4Range `类，分别用于保存单个 IP 地址和一系列地址作为实例。现在，让我们看一下利用上述类的代码示例:

```java
public static boolean checkIPv4IsInRange (String inputIP, String rangeStartIP, String rangeEndIP) {
    Ipv4 startIPAddress = Ipv4.of(rangeStartIP);
    Ipv4 endIPAddress = Ipv4.of(rangeEndIP);
    Ipv4Range ipRange = Ipv4Range.from(startIPAddress).to(endIPAddress);
    Ipv4 inputIPAddress = Ipv4.of(inputIP);
    return ipRange.contains(inputIPAddress);
}
```

`Ipv4`类提供了一个静态方法`of() `，它接受 IP 字符串来构造一个`Ipv4`实例。`Ipv4Range `类使用[生成器设计模式](/web/20220625070344/https://www.baeldung.com/creational-design-patterns#builder)通过使用`from()`和`to()`方法指定范围来创建其实例。此外，它还提供了`contains the ()` 函数来检查 IP 地址是否在指定的范围内。

现在让我们对我们的函数进行一些测试:

```java
@Test
void givenIPv4Addresses_whenIsInRange_thenReturnsTrue() throws Exception {
    assertTrue(IPWithGivenRangeCheck.checkIPv4IsInRange("192.220.3.0", "192.210.0.0", "192.255.0.0"));
}

@Test
void givenIPv4Addresses_whenIsNotInRange_thenReturnsFalse() throws Exception {
    assertFalse(IPWithGivenRangeCheck.checkIPv4IsInRange("192.200.0.0", "192.210.0.0", "192.255.0.0"));
}
```

### 4.2.对于 IPv6

对于 IP 版本 6，库提供了相同的类和函数，只是版本号从 4 变到了 6。版本 6 的类是`Ipv6`和`Ipv6Range. `

让我们利用前面提到的类来看看 IP 版本 6 的代码示例:

```java
public static boolean checkIPv6IsInRange (String inputIP, String rangeStartIP, String rangeEndIP) {
    Ipv6 startIPAddress = Ipv6.of(rangeStartIP);
    Ipv6 endIPAddress = Ipv6.of(rangeEndIP);
    Ipv6Range ipRange = Ipv6Range.from(startIPAddress).to(endIPAddress);
    Ipv6 inputIPAddress = Ipv6.of(inputIP);
    return ipRange.contains(inputIPAddress);
}
```

现在让我们运行单元测试来检查我们的代码:

```java
@Test
void givenIPv6Addresses_whenIsInRange_thenReturnsTrue() throws Exception {
    assertTrue(IPWithGivenRangeCheck.checkIPv6IsInRange(
      "2001:db8:85a3::8a03:a:b", "2001:db8:85a3::8a00:ff:ffff", "2001:db8:85a3::8a2e:370:7334"));
}

@Test
void givenIPv6Addresses_whenIsNotInRange_thenReturnsFalse() throws Exception {
    assertFalse(IPWithGivenRangeCheck.checkIPv6IsInRange(
      "2002:db8:85a3::8a03:a:b", "2001:db8:85a3::8a00:ff:ffff", "2001:db8:85a3::8a2e:370:7334"));
}
```

## 5.将 Java 的`InetAddress`类用于 IPv4

**IPv4 地址是四个 1 字节值的序列。因此，它可以转换为 32 位整数。**我们可以检查它是否落在给定的范围内。

Java 的`InetAddress `类表示一个 IP 地址，并提供获取任何给定主机名的 IP 的方法。InetAddress 的一个实例表示 IP 地址及其对应的主机名。

下面是将 IPv4 地址转换成长整数的 Java 代码:

```java
long ipToLongInt (InetAddress ipAddress) {
    long resultIP = 0;
    byte[] ipAddressOctets = ipAddress.getAddress();

    for (byte octet : ipAddressOctets) {
        resultIP <<= 8;
        resultIP |= octet & 0xFF;
    }
    return resultIP;
}
```

通过使用上述方法，让我们检查 IP 是否在范围内:

```java
public static boolean checkIPv4IsInRangeByConvertingToInt (String inputIP, String rangeStartIP, String rangeEndIP) 
  throws UnknownHostException {
    long startIPAddress = ipToLongInt(InetAddress.getByName(rangeStartIP));
    long endIPAddress = ipToLongInt(InetAddress.getByName(rangeEndIP));
    long inputIPAddress = ipToLongInt(InetAddress.getByName(inputIP));

    return (inputIPAddress >= startIPAddress && inputIPAddress <= endIPAddress);
}
```

`InetAddress`类中的`getByName()`方法接受域名或 IP 地址作为输入，如果无效就抛出`UnknownHostException`。让我们通过运行单元测试来检查我们的代码:

```java
@Test
void givenIPv4Addresses_whenIsInRange_thenReturnsTrue() throws Exception {
    assertTrue(IPWithGivenRangeCheck.checkIPv4IsInRangeByConvertingToInt("192.220.3.0", "192.210.0.0", "192.255.0.0"));
}

@Test
void givenIPv4Addresses_whenIsNotInRange_thenReturnsFalse() throws Exception {
    assertFalse(IPWithGivenRangeCheck.checkIPv4IsInRangeByConvertingToInt("192.200.0.0", "192.210.0.0", "192.255.0.0"));
}
```

上述将 IP 地址转换成整数的逻辑也适用于 IPv6，但它是一个 128 位的整数。Java 语言在原始数据类型中最多支持 64 位(长整数)。如果我们必须在版本 6 中应用上述逻辑，我们需要使用两个长整数或 [BigInteger](/web/20220625070344/https://www.baeldung.com/java-bigdecimal-biginteger) 类进行计算。但这将是一个繁琐的过程，还涉及复杂的计算。

## 6.Java IPv6 库

[Java IPv6 库](https://web.archive.org/web/20220625070344/https://github.com/janvanbesien/java-ipv6/)是专门为 Java 中的 IPv6 支持而编写的，并在其上执行相关操作。**该库内部使用两个长整数来存储 IPv6 地址。而且至少需要 Java 6 才能工作。**

我们需要将 [java-ipv6](https://web.archive.org/web/20220625070344/https://mvnrepository.com/artifact/com.googlecode.java-ipv6/java-ipv6) 依赖项添加到 pom.xml 中:

```java
<dependency>
    <groupId>com.googlecode.java-ipv6</groupId>
    <artifactId>java-ipv6</artifactId>
    <version>0.17</version>
</dependency> 
```

该库提供了各种类来操作 IPv6 地址。这里有两个帮助我们解决问题的例子:

*   `IPv6Address`，用于将 IPv6 表示为 Java 实例
*   `IPv6AddressRange`，用于表示连续 IPv6 地址的连续范围

让我们看看使用上述类来检查 IP 是否在给定范围内的代码片段:

```java
public static boolean checkIPv6IsInRangeByIPv6library (String inputIP, String rangeStartIP, String rangeEndIP) {
    IPv6Address startIPAddress = IPv6Address.fromString(rangeStartIP);
    IPv6Address endIPAddress = IPv6Address.fromString(rangeEndIP);
    IPv6AddressRange ipRange = IPv6AddressRange.fromFirstAndLast(startIPAddress, endIPAddress);
    IPv6Address inputIPAddress = IPv6Address.fromString(inputIP);
    return ipRange.contains(inputIPAddress);
}
```

`IPv6Address`类给了我们各种静态函数来构造它的实例:

*   `fromString`
*   `fromInetAddress`
*   `fromBigInteger`
*   `fromByteArray`
*   `fromLongs`

以上所有方法都是不言自明的，这有助于我们创建一个`IPv6Address`实例。`IPv6AddressRange `有一个名为`fromFirstAndLast()`的方法，它接受两个 IP 地址作为输入。此外，它还提供了一个`contains()`方法，该方法将一个`IPv6Address`作为参数，并确定它是否出现在指定的范围内。

通过调用我们定义的上述方法，让我们在测试中传递几个样本输入:

```java
@Test
void givenIPv6Addresses_whenIsInRange_thenReturnsTrue() throws Exception {
    assertTrue(IPWithGivenRangeCheck.checkIPv6IsInRangeByIPv6library(
      "fe80::226:2dff:fefa:dcba",
      "fe80::226:2dff:fefa:cd1f",
      "fe80::226:2dff:fefa:ffff"
    ));
}

@Test
void givenIPv6Addresses_whenIsNotInRange_thenReturnsFalse() throws Exception {
    assertFalse(IPWithGivenRangeCheck.checkIPv6IsInRangeByIPv6library(
      "2002:db8:85a3::8a03:a:b",
      "2001:db8:85a3::8a00:ff:ffff",
      "2001:db8:85a3::8a2e:370:7334"
    ));
}
```

## 7.结论

在本文中，我们研究了如何确定给定的 IP 地址(v4 和 v6)是否在指定的范围内。在各种库的帮助下，我们分析了检查 IP 地址的存在，而没有任何复杂的逻辑和计算。

和往常一样，这篇文章的代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220625070344/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-3)