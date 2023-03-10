# 在 Java 中启用无限强度加密

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jce-enable-unlimited-strength>

## 1.概观

在本教程中，我们将了解为什么 Java 加密扩展(JCE)无限强度策略文件并不总是默认启用。此外，我们将解释如何检查加密强度。之后，我们将展示如何在不同版本的 Java 中启用无限加密。

## 2.JCE 无限力量政策文件

我们来理解一下[密码强度](/web/20220627085358/https://www.baeldung.com/cs/cryptographic-algorithm-complexity)是什么意思。它是由发现密钥的难度定义的，这取决于使用的[密码](/web/20220627085358/https://www.baeldung.com/java-list-cipher-algorithms)和密钥的长度。一般来说，密钥越长，加密越强。有限的加密强度使用最大 128 位密钥。另一方面，无限制密钥使用最大长度为 2147483647 位的密钥。

正如我们所知，JRE 本身包含加密功能。**JCE 使用管辖策略文件来控制加密强度**。**策略文件由两个罐子组成:`local_policy.jar`和`US_export_policy.jar`T5。由于这一点，Java 平台已经内置了对加密强度的控制。**

## 3.为什么默认情况下不包括 JCE 无限强度策略文件

首先，只有旧版本的 JRE 不包含无限强度策略文件。JRE 版本 8u151 和更早版本仅捆绑了有限的策略文件。相反，从 Java 版本 8u151 开始，JRE 提供了无限和有限的策略文件。**原因很简单，一些国家要求受限的加密强度。**如果某个国家的法律允许无限制的加密强度，则可以根据 Java 版本捆绑或启用它。

## 4.如何检查加密强度

让我们来看看如何检查加密强度。我们可以通过检查允许的最大密钥长度来做到这一点:

```java
int maxKeySize = javax.crypto.Cipher.getMaxAllowedKeyLength("AES");
```

在策略文件有限的情况下，它返回 128。另一方面，如果它返回 2147483647，那么 JCE 使用无限的策略文件。

## 5。策略文件位于何处

**Java 版本 8u151 及更早版本包含`JAVA_HOME/jre/lib/security directory.`** 中的策略文件

**从版本 8u151 开始，JRE 提供不同的策略文件集。**因此，在 JRE 目录下`JAVA_HOME/jre/lib/security/policy`有两个子目录:`limited`和`unlimited`。第一个包含有限强度的策略文件。第二个包含无限个。

## 6.如何启用无限强度加密

现在让我们来看看如何实现最大的加密强度。根据我们使用的 Java 版本的不同，有不同的方法来实现。

### 6.1.Java 版本 8u151 之前的处理

在 8u151 版本之前，JRE 仅包含有限的强度策略文件。我们必须用 Oracle 网站上的无限制版本来替换它。

首先，我们下载 Java 8 的文件，可以在[这里找到。](https://web.archive.org/web/20220627085358/https://www.oracle.com/java/technologies/javase-jce8-downloads.html)接下来，我们对下载的包进行解包，其中包含`local_policy.jar`和`US_export_policy.jar`。

最后，我们将这些文件复制到`JAVA_HOME/jre/lib/security.`

### 6.2。Java 版本 8u151 之后的处理

在 Java 版本 8u151 和更高版本中，默认情况下，JCE 框架使用无限强度策略文件。此外，如果我们想定义使用哪个版本，有一个安全属性`crypto.policy:`

```java
Security.setProperty("crypto.policy", "unlimited");
```

**我们必须在 JCE 框架初始化之前设置属性。**它在`JAVA_HOME/jre/lib/security/policy`下为策略文件定义了一个目录。

首先，当安全属性未设置时，框架检查策略文件的遗留位置`JAVA_HOME/jre/lib/security` 。尽管默认情况下，在新版本的 Java 中，遗留位置没有策略文件。JCE 检查它作为第一个兼容旧版本。

其次，如果 jar 文件不在遗留位置，并且没有定义属性，那么 JRE 默认使用无限制的策略文件。

## 7.结论

在这篇短文中，我们了解了 JCE 无限强度政策文件。首先，我们看了为什么在旧版本的 Java 中没有默认启用无限加密强度。接下来，我们学习了如何通过检查最大密钥长度来确定加密强度。最后，我们看到了如何在不同版本的 Java 中启用它。

和往常一样，这个例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627085358/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-3)