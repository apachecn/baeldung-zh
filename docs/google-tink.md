# 谷歌 Tink 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/google-tink>

## 1。简介

现在，许多开发人员使用加密技术来保护用户数据。

在密码学中，很小的实现错误就可能产生严重的后果，而理解如何正确地实现密码学是一项复杂而耗时的任务。

在本教程中，我们将描述[Tink](https://web.archive.org/web/20220630005809/https://github.com/google/tink)——一个多语言、跨平台的加密库，可以帮助我们实现安全的加密代码。

## 2。依赖性

我们可以使用 Maven 或 Gradle 来导入 Tink。

对于我们的教程，我们将只添加 [Tink 的 Maven 依赖关系](https://web.archive.org/web/20220630005809/https://search.maven.org/search?q=g:com.google.crypto.tink%20AND%20a:tink&core=gav):

```
<dependency>
    <groupId>com.google.crypto.tink</groupId>
    <artifactId>tink</artifactId>
    <version>1.2.2</version>
</dependency>
```

虽然我们可以用格雷尔来代替:

```
dependencies {
  compile 'com.google.crypto.tink:tink:latest'
}
```

## 3。初始化

在使用任何 Tink APIs 之前，我们需要初始化它们。

如果我们需要使用 Tink 中所有原语的所有实现，我们可以使用`TinkConfig.register()`方法:

```
TinkConfig.register();
```

而例如，如果我们只需要 AEAD 原语，我们可以使用`AeadConfig.register()`方法:

```
AeadConfig.register();
```

还为每个实现提供了可定制的初始化。

## 4。Tink 原语

本库使用的主要对象被称为原语，根据类型的不同，原语包含不同的加密功能。

一个原语可以有多种实现:

| 原始的 | 履行 |
| --- | --- |
| AEAD | AES-EAX，AES-GCM，AES-CTR-HMAC，KMS 信封，CHACHA20-POLY1305 |
| --- | --- |
| 流式 AEAD | AES-GCM-HKDF 流，AES-CTR-HMAC 流 |
| --- | --- |
| 确定性 AEAD | AEAD: AES-SIV |
| --- | --- |
| 测量与控制(Measurement and Control) | HMAC-SHA2 |
| --- | --- |
| 数字签名 | NIST 曲线上的 ECDSA，ED25519 |
| --- | --- |
| 混合加密 | 带有 AEAD 和 HKDF 的 ECIES，(NaCl 密码箱) |
| --- | --- |

我们可以通过调用相应工厂类的方法 `getPrimitive()`并传递给它一个`KeysetHandle`来获得一个原语:

```
Aead aead = AeadFactory.getPrimitive(keysetHandle); 
```

### 4.1.`KeysetHandle`

为了**提供加密功能，每个原语需要一个包含所有密钥材料和参数的密钥结构**。

Tink 提供了一个对象—`KeysetHandle – `,它用一些附加的参数和元数据包装了一个键集。

因此，在实例化一个原语之前，我们需要创建一个`KeysetHandle `对象:

```
KeysetHandle keysetHandle = KeysetHandle.generateNew(AeadKeyTemplates.AES256_GCM);
```

在生成一个密钥后，我们可能想要持久化它:

```
String keysetFilename = "keyset.json";
CleartextKeysetHandle.write(keysetHandle, JsonKeysetWriter.withFile(new File(keysetFilename)));
```

然后，我们可以随后加载它:

```
String keysetFilename = "keyset.json";
KeysetHandle keysetHandle = CleartextKeysetHandle.read(JsonKeysetReader.withFile(new File(keysetFilename)));
```

## 5。加密

Tink 提供了多种应用 AEAD 算法的方法。让我们来看看。

### 5.1。AEAD

AEAD 提供了对关联数据的认证加密，这意味着**我们可以加密明文，并且可以选择提供应该被认证但没有被加密的关联数据**。

注意，该算法确保相关数据的真实性和完整性，但不确保其保密性。

如前所述，要使用 AEAD 实现之一加密数据，我们需要初始化库并创建一个`keysetHandle:`

```
AeadConfig.register();
KeysetHandle keysetHandle = KeysetHandle.generateNew(
  AeadKeyTemplates.AES256_GCM);
```

一旦我们做到了这一点，我们就可以获得原语并加密所需的数据:

```
String plaintext = "baeldung";
String associatedData = "Tink";

Aead aead = AeadFactory.getPrimitive(keysetHandle); 
byte[] ciphertext = aead.encrypt(plaintext.getBytes(), associatedData.getBytes());
```

接下来，我们可以使用`decrypt()`方法解密`ciphertext`:

```
String decrypted = new String(aead.decrypt(ciphertext, associatedData.getBytes()));
```

### 5.2。流式 AEAD

类似地，**当要加密的数据太大而无法在一个步骤中处理时，我们可以使用流式 AEAD 原语**:

```
AeadConfig.register();
KeysetHandle keysetHandle = KeysetHandle.generateNew(
  StreamingAeadKeyTemplates.AES128_CTR_HMAC_SHA256_4KB);
StreamingAead streamingAead = StreamingAeadFactory.getPrimitive(keysetHandle);

FileChannel cipherTextDestination = new FileOutputStream("cipherTextFile").getChannel();
WritableByteChannel encryptingChannel =
  streamingAead.newEncryptingChannel(cipherTextDestination, associatedData.getBytes());

ByteBuffer buffer = ByteBuffer.allocate(CHUNK_SIZE);
InputStream in = new FileInputStream("plainTextFile");

while (in.available() > 0) {
    in.read(buffer.array());
    encryptingChannel.write(buffer);
}

encryptingChannel.close();
in.close();
```

基本上，我们需要`WriteableByteChannel` 来实现这一点。

因此，为了解密`cipherTextFile,` ，我们需要使用一个`ReadableByteChannel`:

```
FileChannel cipherTextSource = new FileInputStream("cipherTextFile").getChannel();
ReadableByteChannel decryptingChannel =
  streamingAead.newDecryptingChannel(cipherTextSource, associatedData.getBytes());

OutputStream out = new FileOutputStream("plainTextFile");
int cnt = 1;
do {
    buffer.clear();
    cnt = decryptingChannel.read(buffer);
    out.write(buffer.array());
} while (cnt>0);

decryptingChannel.close();
out.close();
```

## 6。混合加密

除了对称加密，Tink 还为混合加密实现了一些原语。

使用混合加密，我们可以获得对称密钥的效率和非对称密钥的便利。

简单地说，我们将使用一个对称密钥来加密明文**,使用一个公钥来加密对称密钥。**

请注意，它只提供保密性，而不提供发送者的身份真实性。

那么，让我们来看看如何使用`HybridEncrypt`和`HybridDecrypt:`

```
TinkConfig.register();

KeysetHandle privateKeysetHandle = KeysetHandle.generateNew(
  HybridKeyTemplates.ECIES_P256_HKDF_HMAC_SHA256_AES128_CTR_HMAC_SHA256);
KeysetHandle publicKeysetHandle = privateKeysetHandle.getPublicKeysetHandle();

String plaintext = "baeldung";
String contextInfo = "Tink";

HybridEncrypt hybridEncrypt = HybridEncryptFactory.getPrimitive(publicKeysetHandle);
HybridDecrypt hybridDecrypt = HybridDecryptFactory.getPrimitive(privateKeysetHandle);

byte[] ciphertext = hybridEncrypt.encrypt(plaintext.getBytes(), contextInfo.getBytes());
byte[] plaintextDecrypted = hybridDecrypt.decrypt(ciphertext, contextInfo.getBytes());
```

`contextInfo`是来自上下文的隐式公共数据，可以是`null`或空的，或用作 AEAD 加密的“关联数据”输入，或用作 HKDF 的“CtxInfo”输入。

`ciphertext` 允许检查`contextInfo` 的完整性，但不能检查其保密性或真实性。

## 7。消息认证码

Tink 也支持消息认证码或 MAC。

MAC 是几个字节的块，我们可以用它来验证消息。

让我们看看如何创建 MAC，然后验证其真实性:

```
TinkConfig.register();

KeysetHandle keysetHandle = KeysetHandle.generateNew(
  MacKeyTemplates.HMAC_SHA256_128BITTAG);

String data = "baeldung";

Mac mac = MacFactory.getPrimitive(keysetHandle);

byte[] tag = mac.computeMac(data.getBytes());
mac.verifyMac(tag, data.getBytes());
```

在数据不可信的情况下，方法`verifyMac()`抛出一个`GeneralSecurityException.`

## 8。数字签名

除了加密 API，Tink 还支持数字签名。

**为了实现数字签名，本库使用`PublicKeySign`原语进行数据签名，使用`PublickeyVerify`进行验证:**

```
TinkConfig.register();

KeysetHandle privateKeysetHandle = KeysetHandle.generateNew(SignatureKeyTemplates.ECDSA_P256);
KeysetHandle publicKeysetHandle = privateKeysetHandle.getPublicKeysetHandle();

String data = "baeldung";

PublicKeySign signer = PublicKeySignFactory.getPrimitive(privateKeysetHandle);
PublicKeyVerify verifier = PublicKeyVerifyFactory.getPrimitive(publicKeysetHandle);

byte[] signature = signer.sign(data.getBytes()); 
verifier.verify(signature, data.getBytes());
```

类似于前面的加密方法，当签名无效时，我们会得到一个`GeneralSecurityException.`

## 9。结论

在本文中，我们介绍了使用 Java 实现的 Google Tink 库。

我们已经了解了如何使用来加密和解密数据，以及如何保护其完整性和真实性。此外，我们已经看到了如何使用数字签名 API 对数据进行签名。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220630005809/https://github.com/eugenp/tutorials/tree/master/libraries-security)