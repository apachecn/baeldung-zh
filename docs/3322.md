# 用 Java 加密和解密文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cipher-input-output-stream>

## 1.概观

在本教程中，我们将看看如何使用现有的 JDK API 加密和解密文件。

## 2.首先编写一个测试

我们将从编写我们的测试开始，TDD 风格。由于我们将在这里处理文件，集成测试似乎是合适的。

因为我们只是使用现有的 JDK 功能，所以不需要外部依赖。

首先，**我们将使用新生成的密钥**加密内容(我们使用 AES，[高级加密标准](https://web.archive.org/web/20221013193919/https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)，作为本例中的对称加密算法)。

还要注意，我们在构造函数(`AES/CBC/PKCS5Padding`)中定义了完整的转换字符串，它是使用的加密、块密码模式和填充(`algorithm/mode/padding`)的串联。默认情况下，JDK 实现支持许多不同的转换，但是请注意，按照今天的标准，并不是每个组合都可以被认为是密码安全的。

我们假设**我们的`FileEncrypterDecrypter`类将把输出写到一个名为`baz.enc`** 的文件中。之后，**我们使用相同的密钥**解密该文件，并检查解密的内容是否等于原始内容:

```
@Test
public void whenEncryptingIntoFile_andDecryptingFileAgain_thenOriginalStringIsReturned() {
    String originalContent = "foobar";
    SecretKey secretKey = KeyGenerator.getInstance("AES").generateKey();

    FileEncrypterDecrypter fileEncrypterDecrypter
      = new FileEncrypterDecrypter(secretKey, "AES/CBC/PKCS5Padding");
    fileEncrypterDecrypter.encrypt(originalContent, "baz.enc");

    String decryptedContent = fileEncrypterDecrypter.decrypt("baz.enc");
    assertThat(decryptedContent, is(originalContent));

    new File("baz.enc").delete(); // cleanup
}
```

## 3.加密

我们将使用指定的转换`String.`在我们的`FileEncrypterDecrypter`类的构造函数中初始化密码

这允许我们在指定错误转换的情况下尽早失败:

```
FileEncrypterDecrypter(SecretKey secretKey, String transformation) {
    this.secretKey = secretKey;
    this.cipher = Cipher.getInstance(transformation);
}
```

然后我们可以**使用实例化的密码和提供的密钥来执行加密:**

```
void encrypt(String content, String fileName) {
    cipher.init(Cipher.ENCRYPT_MODE, secretKey);
    byte[] iv = cipher.getIV();

    try (FileOutputStream fileOut = new FileOutputStream(fileName);
      CipherOutputStream cipherOut = new CipherOutputStream(fileOut, cipher)) {
        fileOut.write(iv);
        cipherOut.write(content.getBytes());
    }
}
```

Java 允许我们**利用方便的`CipherOutputStream`类将加密的内容写入另一个`OutputStream`** 。

请注意，我们将 IV ( [初始化向量](https://web.archive.org/web/20221013193919/https://en.wikipedia.org/wiki/Initialization_vector))写到输出文件的开头。在本例中，初始化`Cipher`时会自动生成 IV。

使用 CBC 模式时，必须使用 IV，以便随机化加密输出。但是 IV 不被认为是秘密，所以把它写在文件的开头是没问题的。

## 4.[通信]解密

为了解密，我们同样必须先读取 IV。之后，我们可以初始化我们的密码并解密内容。

同样，我们可以使用一个特殊的 Java 类 **`CipherInputStream`，它透明地负责实际的解密**:

```
String decrypt(String fileName) {
    String content;

    try (FileInputStream fileIn = new FileInputStream(fileName)) {
        byte[] fileIv = new byte[16];
        fileIn.read(fileIv);
        cipher.init(Cipher.DECRYPT_MODE, secretKey, new IvParameterSpec(fileIv));

        try (
                CipherInputStream cipherIn = new CipherInputStream(fileIn, cipher);
                InputStreamReader inputReader = new InputStreamReader(cipherIn);
                BufferedReader reader = new BufferedReader(inputReader)
            ) {

            StringBuilder sb = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                sb.append(line);
            }
            content = sb.toString();
        }

    }
    return content;
}
```

## 5.结论

我们已经看到，我们可以使用标准的 JDK 类来执行基本的加密和解密，比如`Cipher`、`CipherOutputStream`和`CipherInputStream`。

像往常一样，本文的完整代码可以在我们的 [GitHub 库](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security)中找到。

此外，您可以在这里找到 JDK [中可用的密码列表。](https://web.archive.org/web/20221013193919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/Cipher.html)

最后，请注意，这里的代码示例并不意味着是产品级代码，在使用它们时，需要彻底考虑系统的具体情况。