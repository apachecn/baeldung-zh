# 爪哇的凯撒密码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-caesar-cipher>

## 1.概观

在本教程中，我们将探索凯撒密码，这是一种加密方法，它将一条消息的字母转换成另一条可读性更差的消息。

首先，我们将通过加密方法，看看如何在 Java 中实现它。

然后，如果我们知道用于加密的偏移量，我们将看到如何解密加密的消息。

最后，我们将学习如何破解这样的密码，从而在不知道所用偏移量的情况下从加密的信息中恢复原始信息。

## 2.凯撒密码

### 2.1.说明

首先，我们来定义一下什么是密码。[密码是一种对消息](https://web.archive.org/web/20221205173933/http://www.cs.trincoll.edu/~crypto/historical/substitution.html)进行加密的方法，旨在降低其可读性。**至于凯撒密码，它是一种替代密码，通过将字母移动给定的偏移量来转换消息。**

假设我们想将字母表移动 3，那么字母`A`将被转换为字母`D`、`B`转换为`E`、`C`转换为`F`，依此类推。

以下是偏移量为 3 时原始字母和转换字母之间的完全匹配:

```
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
D E F G H I J K L M N O P Q R S T U V W X Y Z A B C
```

我们可以看到，**一旦变换超出了字母`Z`，我们就回到字母表的开始，这样`X`、`Y`和`Z`就分别变换成了`A`、`B`和`C`。**

因此，如果我们选择大于或等于 26 的偏移量，我们将在整个字母表上循环至少一次。假设我们将一条消息移动了 28，这实际上意味着我们将它移动了 2。事实上，在移位 26 之后，所有的字母都在自我匹配。

实际上，我们可以通过**对其执行模 26 运算**将任何偏移转换为更简单的偏移:

```
offset = offset % 26
```

### 2.2.Java 中的算法

现在，让我们看看如何用 Java 实现凯撒密码。

首先，让我们创建一个类`CaesarCipher`，它将保存一个将消息和偏移量作为参数的`cipher()`方法:

```
public class CaesarCipher {
    String cipher(String message, int offset) {}
}
```

该方法将使用凯撒密码加密信息。

这里我们假设偏移量是正的，消息只包含小写字母和空格。然后，我们想要的是将所有字母字符移动给定的偏移量:

```
StringBuilder result = new StringBuilder();
for (char character : message.toCharArray()) {
    if (character != ' ') {
        int originalAlphabetPosition = character - 'a';
        int newAlphabetPosition = (originalAlphabetPosition + offset) % 26;
        char newCharacter = (char) ('a' + newAlphabetPosition);
        result.append(newCharacter);
    } else {
        result.append(character);
    }
}
return result;
```

正如我们所见，**我们依靠字母表字母的 ASCII 码来实现我们的目标**。

首先，我们计算当前字母在字母表中的位置，为此，我们获取它的 ASCII 码并从中减去字母`a`的 ASCII 码。然后，我们将偏移量应用到这个位置，小心地使用模来保持在字母表范围内。最后，我们通过将新位置添加到字母`a`的 ASCII 代码中来检索新字符。

现在，让我们在消息“他告诉我我永远也不能教美洲驼开车”上尝试这个实现，偏移量为 3:

```
CaesarCipher cipher = new CaesarCipher();

String cipheredMessage = cipher.cipher("he told me i could never teach a llama to drive", 3);

assertThat(cipheredMessage)
  .isEqualTo("kh wrog ph l frxog qhyhu whdfk d oodpd wr gulyh");
```

正如我们所看到的，加密的消息遵循前面定义的偏移量为 3 的匹配。

现在，这个特殊的例子具有在转换期间不超过字母`z`的特殊性，因此不必回到字母表的开始。因此，让我们用 10 的偏移量再试一次，这样一些字母将被映射到字母表开头的字母，比如`t`将被映射到`d`:

```
String cipheredMessage = cipher.cipher("he told me i could never teach a llama to drive", 10);

assertThat(cipheredMessage)
  .isEqualTo("ro dyvn wo s myevn xofob dokmr k vvkwk dy nbsfo");
```

由于模运算，它像预期的那样工作。该操作还会处理较大的偏移量。假设我们想使用 36 作为偏移量，它相当于 10，模运算确保转换将给出相同的结果。

## 3.解释

### 3.1.说明

现在，让我们看看当我们知道用于加密的偏移量时，如何解密这样的消息。

事实上，**解密一条用凯撒密码加密的消息可以看作是用一个负偏移量对其加密，或者也可以用一个互补偏移量对其加密**。

假设我们有一条偏移量为 3 的加密消息。然后，我们可以用-3 的偏移量加密它，也可以用 23 的偏移量加密它。不管怎样，我们都要找回原始信息。

不幸的是，我们的算法不能开箱即用地处理负偏移。我们在转换循环回到字母表末尾的字母时会遇到问题(例如，将字母`a`转换成偏移量为-1 的字母`z`)。但是，我们可以计算正的互补失调，然后使用我们的算法。

那么，如何获得这种互补偏移呢？最简单的方法是用 26 减去原始偏移量。当然，这将适用于 0 到 26 之间的偏移量，否则会产生负的结果。

这就是**我们将再次使用模运算符的地方，在做减法**之前，直接在原始偏移上使用。这样，我们确保总是返回一个正的偏移量。

### 3.2.Java 中的算法

现在让我们用 Java 来实现它。首先，我们将在我们的类中添加一个`decipher()`方法:

```
String decipher(String message, int offset) {}
```

然后，让我们用计算出的互补偏移量调用`cipher()`方法:

```
return cipher(message, 26 - (offset % 26));
```

就这样，我们的破译算法设置好了。让我们在偏移量为 36 的示例中尝试一下:

```
String decipheredSentence = cipher.decipher("ro dyvn wo s myevn xofob dokmr k vvkwk dy nbsfo", 36);
assertThat(decipheredSentence)
  .isEqualTo("he told me i could never teach a llama to drive");
```

如我们所见，我们恢复了原始信息。

## 4.破解张敬利密码

### 4.1.说明

既然我们已经介绍了使用凯撒密码对信息进行加密和解密，那么我们可以深入研究如何破解它。**也就是说，在一开始不知道所用偏移量的情况下，解密一条加密的消息。**

为此，我们将利用[概率在文本](https://web.archive.org/web/20221205173933/http://www.cs.trincoll.edu/~crypto/historical/caesar.html)中找到英文字母。我们的想法是使用 0 到 25 的偏移量来解密消息，并检查什么样的偏移呈现类似于英语文本的字母分布。

**为了确定两个分布的相似性，我们将使用[卡方统计](https://web.archive.org/web/20221205173933/http://practicalcryptography.com/cryptanalysis/text-characterisation/chi-squared-statistic/)。**

卡方统计将提供一个数字，告诉我们两个分布是否相似。数字越小，越相似。

因此，我们将计算每个偏移的卡方，然后返回卡方最小的那个。这应该给我们用来加密信息的偏移量。

然而，我们必须记住**这种技术并不可靠**,如果消息太短或者使用了不代表标准英语文本的词语，它可能会返回错误的偏移量。

### 4.2.定义基本字母分布

现在让我们看看如何用 Java 实现 breaking 算法。

首先，让我们在我们的`CaesarCipher` 类中创建一个`breakCipher()`方法，它将返回用于加密消息的偏移量:

```
int breakCipher(String message) {}
```

然后，让我们定义一个包含在英文文本中找到某个字母的概率的数组:

```
double[] englishLettersProbabilities = {0.073, 0.009, 0.030, 0.044, 0.130, 0.028, 0.016, 0.035, 0.074,
  0.002, 0.003, 0.035, 0.025, 0.078, 0.074, 0.027, 0.003,
  0.077, 0.063, 0.093, 0.027, 0.013, 0.016, 0.005, 0.019, 0.001};
```

从这个数组中，我们将能够计算出给定消息中字母的预期频率，方法是将概率乘以消息的长度:

```
double[] expectedLettersFrequencies = Arrays.stream(englishLettersProbabilities)
  .map(probability -> probability * message.getLength())
  .toArray();
```

例如，在长度为 100 的消息中，我们应该预计字母`a`会出现 7.3 次，字母`e`会出现 13 次。

### 4.3.计算卡方值

现在，我们将计算破译的信息字母分布和标准英文字母分布的卡方。

为了实现这一点，我们需要导入包含计算卡方的实用程序类的 Apache Commons Math3 库:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>
```

我们现在需要做的是**创建一个数组，该数组包含为 0 到 25** 之间的每个偏移量计算的卡方。

因此，我们将使用每个偏移量解密加密的消息，然后计算消息中的字母数。

最后，我们将使用`ChiSquareTest#chiSquare`方法来计算预期和观察到的字母分布之间的卡方:

```
double[] chiSquares = new double[26];

for (int offset = 0; offset < chiSquares.length; offset++) {
    String decipheredMessage = decipher(message, offset);
    long[] lettersFrequencies = observedLettersFrequencies(decipheredMessage);
    double chiSquare = new ChiSquareTest().chiSquare(expectedLettersFrequencies, lettersFrequencies);
    chiSquares[offset] = chiSquare;
}

return chiSquares;
```

`observedLettersFrequencies()`方法简单地实现了在传递的消息中对字母`a`到`z`的计数:

```
long[] observedLettersFrequencies(String message) {
    return IntStream.rangeClosed('a', 'z')
      .mapToLong(letter -> countLetter((char) letter, message))
      .toArray();
}

long countLetter(char letter, String message) {
    return message.chars()
      .filter(character -> character == letter)
      .count();
}
```

### 4.4.找到最可能的偏移量

计算完所有卡方后，我们可以返回与最小卡方匹配的偏移量:

```
int probableOffset = 0;
for (int offset = 0; offset < chiSquares.length; offset++) {
    <span class="x x-first">log</span><span class="pl-k x">.</span><span class="x x-last">debug</span>(String.format("Chi-Square for offset %d: %.2f", offset, chiSquares[offset]));
    if (chiSquares[offset] < chiSquares[probableOffset]) {
        probableOffset = offset;
    }
}

return probableOffset;
```

虽然在开始循环之前，我们认为偏移量 0 是最小值，所以没有必要进入循环，但是我们这样做是为了打印它的卡方值。

让我们对使用偏移量 10 加密的消息尝试这种算法:

```
int offset = algorithm.breakCipher("ro dyvn wo s myevn xofob dokmr k vvkwk dy nbsfo");
assertThat(offset).isEqualTo(10);

assertThat(algorithm.decipher("ro dyvn wo s myevn xofob dokmr k vvkwk dy nbsfo", offset))
  .isEqualTo("he told me i could never teach a llama to drive");
```

正如我们所看到的，该方法检索正确的偏移量，然后可以用它来解密消息并检索原始消息。

以下是针对这一特定中断计算的不同卡方:

```
Chi-Square for offset 0: 210.69
Chi-Square for offset 1: 327.65
Chi-Square for offset 2: 255.22
Chi-Square for offset 3: 187.12
Chi-Square for offset 4: 734.16
Chi-Square for offset 5: 673.68
Chi-Square for offset 6: 223.35
Chi-Square for offset 7: 111.13
Chi-Square for offset 8: 270.11
Chi-Square for offset 9: 153.26
Chi-Square for offset 10: 23.74
Chi-Square for offset 11: 643.14
Chi-Square for offset 12: 328.83
Chi-Square for offset 13: 434.19
Chi-Square for offset 14: 384.80
Chi-Square for offset 15: 1206.47
Chi-Square for offset 16: 138.08
Chi-Square for offset 17: 262.66
Chi-Square for offset 18: 253.28
Chi-Square for offset 19: 280.83
Chi-Square for offset 20: 365.77
Chi-Square for offset 21: 107.08
Chi-Square for offset 22: 548.81
Chi-Square for offset 23: 255.12
Chi-Square for offset 24: 458.72
Chi-Square for offset 25: 325.45
```

我们可以看到，偏移 10 的那个明显比其他的小。

## 5.结论

在本文中，我们讨论了凯撒密码。我们学会了如何通过将字母移动给定的偏移量来加密和解密信息。我们还学会了如何破解密码。我们看到了所有允许我们这样做的 Java 实现。

这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221205173933/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-6)