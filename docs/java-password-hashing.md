# 用 Java 散列密码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-password-hashing>

## 1。概述

在本教程中，我们将讨论密码哈希的重要性。

我们将快速看一下它是什么，为什么它很重要，以及用 Java 实现它的一些安全和不安全的方法。

## 2。哈希是什么？

[散列](/web/20220625071421/https://www.baeldung.com/cs/hashing)是使用一个被称为`cryptographic hash function`的数学函数从一个给定的`message`生成一个字符串或`hash`的过程。

虽然有几种哈希函数，但那些专门用于哈希密码的函数需要具备四个主要属性才能确保安全:

1.  应该是`deterministic`:相同的哈希函数处理的相同消息应该`always `产生相同的`hash`
2.  不是`reversible`:从它的`hash`生成一个`message`是不切实际的
3.  它有很高的 [`entropy`](/web/20220625071421/https://www.baeldung.com/cs/cs-entropy-definition) :一个`message`的小变化应该产生一个非常不同的`hash`
4.  它抵制`collisions`:两个不同的`messages`不应该产生相同的`hash`

具有所有四个属性的散列函数是密码散列的强有力候选，因为它们一起极大地增加了从散列反向工程密码的难度。

此外，虽然，**密码散列函数应该很慢**。一种快速算法将有助于 [`brute force `攻击](/web/20220625071421/https://www.baeldung.com/cs/brute-force-cybersecurity-string-search)，在这种攻击中，黑客将试图通过散列和比较每秒数十亿([或数万亿](https://web.archive.org/web/20220625071421/https://www.wired.com/2014/10/snowdens-first-emails-to-poitras/))的潜在密码来猜测密码。

满足所有这些标准的一些很棒的散列函数是`PBKDF2, ` `BCrypt, `和`SCrypt. `，但是首先，让我们来看看一些较老的算法以及为什么不再推荐它们

## 3。不推荐:MD5

我们的第一个散列函数是 MD5 消息摘要算法，早在 1992 年就开发出来了。

Java 的`MessageDigest`使这一点易于计算，在其他情况下仍然有用。

然而，在过去的几年里， **MD5 被发现使[第四密码散列属性](https://web.archive.org/web/20220625071421/https://blog.avira.com/md5-the-broken-algorithm/)** 失效，因为它变得很容易在计算上产生冲突。最重要的是，MD5 是一种快速算法，因此无法抵御暴力攻击。

**因为这些，不推荐 MD5。**

## 4。不推荐:SHA-512

接下来，我们来看看 SHA-512，它是安全散列算法家族的一部分，该家族始于 1993 年的 SHA-0。

### 4.1。为什么是 SHA-512？

随着计算机功能的增强，以及我们发现新的漏洞，研究人员会衍生出新版本的 SHA。新版本的长度越来越长，或者有时研究人员会发布新版本的底层算法。

SHA-512 代表第三代算法中最长的密钥。

虽然现在有更安全的 SHA 版本，但是用 Java 实现的最强的版本。

### 4.2。用 Java 实现

现在，让我们看看用 Java 实现 SHA-512 散列算法。

首先，我们要理解`salt`的概念。简单地说，**这是为每个新散列**生成的随机序列。

通过引入这种随机性，我们增加了散列的`entropy`，并且我们保护我们的数据库免受被称为`rainbow tables`的预先编译的散列列表的影响。

我们的新散列函数大致变成:

```java
salt <- generate-salt;
hash <- salt + ':' + sha512(salt + password)
```

### 4.3。生成盐

为了介绍 salt，我们将使用来自`java.security`的`[SecureRandom](https://web.archive.org/web/20220625071421/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/SecureRandom.html) `类:

```java
SecureRandom random = new SecureRandom();
byte[] salt = new byte[16];
random.nextBytes(salt);
```

然后，我们将使用`[MessageDigest](https://web.archive.org/web/20220625071421/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/MessageDigest.html) `类用我们的 salt 配置`SHA-512 `散列函数:

```java
MessageDigest md = MessageDigest.getInstance("SHA-512");
md.update(salt);
```

添加后，我们现在可以使用`digest`方法来生成散列密码:

```java
byte[] hashedPassword = md.digest(passwordToHash.getBytes(StandardCharsets.UTF_8));
```

### 4.4。为什么不推荐？

当与盐一起使用时， [SHA-512 仍然是一个公平的选择，](https://web.archive.org/web/20220625071421/https://en.wikipedia.org/wiki/Secure_Hash_Algorithms)，**但是还有更强和更慢的选择。**

此外，我们将讨论的其余选项有一个重要的特性:可配置的强度。

## 5。PBKDF2、BCrypt 和 SCrypt

PBKDF2、BCrypt 和 SCrypt 是三种推荐的算法。

### 5.1。为什么推荐那些？

每一种都很慢，而且每一种都有一个可配置强度的出色特性。

这意味着随着计算机能力的增强，我们可以通过改变输入来降低算法的速度。

### 5.2。用 Java 实现 pbk df 2

现在， **salts 是密码散列的一个基本原理**，所以我们也需要一个用于 PBKDF2:

```java
SecureRandom random = new SecureRandom();
byte[] salt = new byte[16];
random.nextBytes(salt);
```

接下来，我们将创建一个 [`PBEKeySpec`](https://web.archive.org/web/20220625071421/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/spec/PBEKeySpec.html) 和一个`[SecretKeyFactory](https://web.archive.org/web/20220625071421/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/SecretKeyFactory.html)`，我们将使用`PBKDF2WithHmacSHA1 `算法实例化它们:

```java
KeySpec spec = new PBEKeySpec(password.toCharArray(), salt, 65536, 128);
SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");
```

第三个参数(`65536`)实际上是强度参数。它指示该算法运行了多少次迭代，从而增加了生成哈希所需的时间。

最后，我们可以使用我们的`SecretKeyFactory `来生成散列:

```java
byte[] hash = factory.generateSecret(spec).getEncoded();
```

### 5.3。用 Java 实现 BCrypt 和 SCrypt

因此，事实证明，虽然有些 Java 库支持 BCrypt 和 SCrypt，但 Java 还没有提供这些支持。

其中一个库是 Spring Security。

## 6。使用 Spring Security 的密码哈希

尽管 Java 本身支持 PBKDF2 和 SHA 散列算法，但它不支持 BCrypt 和 SCrypt 算法。

幸运的是，Spring Security 通过 [`PasswordEncoder`](https://web.archive.org/web/20220625071421/https://docs.spring.io/spring-security/site/docs/4.2.4.RELEASE/apidocs/org/springframework/security/crypto/password/PasswordEncoder.html) 接口支持所有这些推荐的算法:

*   `Pbkdf2PasswordEncoder`给了我们 PBKDF2
*   `BCryptPasswordEncoder `给我们 BCrypt，和
*   给了我们一个秘密

**pbk df 2、BCrypt 和 SCrypt 的密码编码器都支持配置所需的密码散列强度。**

即使没有基于 Spring 安全的应用程序，我们也可以直接使用这些编码器。或者，如果我们用 Spring Security 来保护我们的站点，那么我们可以通过它的 DSL 或者通过[依赖注入](/web/20220625071421/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)来配置我们想要的密码编码器。

而且，与我们上面的例子不同，**这些加密算法将在内部为我们生成 salt】。该算法将 salt 存储在输出哈希中，供以后验证密码时使用。**

## 7 .**。结论**

因此，我们深入研究了密码散列法；探索概念及其用途。

在用 Java 编写之前，我们已经看了一些历史上的散列函数和一些当前实现的散列函数。

最后，我们看到 Spring Security 附带了密码加密类，实现了一系列不同的散列函数。

和往常一样，代码可以在 GitHub 上获得。