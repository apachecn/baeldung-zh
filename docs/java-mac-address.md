# 用 Java 获取 MAC 地址

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mac-address>

## 1.介绍

在本教程中，我们将使用 Java 来获取本地机器的 MAC 地址。

MAC 地址是物理网络接口卡的唯一标识符。

我们将只讨论 MAC 地址，但是对于网络接口的更一般的概述，请参考[在 Java 中使用网络接口](/web/20220701013512/https://www.baeldung.com/java-network-interfaces)。

## 2.例子

在下面的例子中，我们将利用`java.net.NetworkInterface`和 `java.net.InetAddress`API。

### 2.1.机器本地主机

首先，让我们获得我们机器的本地主机的 MAC 地址:

```java
InetAddress localHost = InetAddress.getLocalHost();
NetworkInterface ni = NetworkInterface.getByInetAddress(localHost);
byte[] hardwareAddress = ni.getHardwareAddress(); 
```

由于 **`NetworkInterface` # `getHardwareAddress`返回一个字节数组，**我们可以将结果格式化:

```java
String[] hexadecimal = new String[hardwareAddress.length];
for (int i = 0; i < hardwareAddress.length; i++) {
    hexadecimal[i] = String.format("%02X", hardwareAddress[i]);
}
String macAddress = String.join("-", hexadecimal);
```

注意我们如何使用 `**String#format**` **将数组中的每个字节格式化为十六进制数。**

之后，我们可以用“-”(破折号)连接所有格式化的元素。

### 2.2.本地 IP

其次，让我们获取给定本地 IP 地址的 MAC 地址:

```java
InetAddress localIP = InetAddress.getByName("192.168.1.108");
NetworkInterface ni = NetworkInterface.getByInetAddress(localIP);
byte[] macAddress = ni.getHardwareAddress();
```

同样，请注意我们是如何获得 MAC 地址的字节数组的。

### 2.3.所有网络接口

最后，让我们获取机器上所有网络接口的 MAC 地址:

```java
Enumeration<NetworkInterface> networkInterfaces = NetworkInterface.getNetworkInterfaces();
while (networkInterfaces.hasMoreElements()) {
    NetworkInterface ni = networkInterfaces.nextElement();
    byte[] hardwareAddress = ni.getHardwareAddress();
    if (hardwareAddress != null) {
        String[] hexadecimalFormat = new String[hardwareAddress.length];
        for (int i = 0; i < hardwareAddress.length; i++) {
            hexadecimalFormat[i] = String.format("%02X", hardwareAddress[i]);
        }
        System.out.println(String.join("-", hexadecimalFormat));
    }
}
```

由于 **`getNetworkInterfaces`同时返回物理接口和虚拟接口，我们需要过滤掉虚拟接口。**

例如，我们可以通过对`getHardwareAddress`进行空检查来做到这一点。

## 3.结论

在这个快速教程中，我们探索了获取本地机器 MAC 地址的不同方法。

像往常一样，本教程中所有示例的源代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220701013512/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-2)