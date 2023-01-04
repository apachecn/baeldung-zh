# 在 Java 中将十六进制转换为 ASCII

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-hex-to-ascii>

## 1。概述

在这篇简短的文章中，我们将在十六进制和 ASCII 格式之间做一些简单的转换。

在一个典型的用例中，十六进制格式可以用来以紧凑的形式写下非常大的整数值。例如，AD45 比它的十进制等效值 44357 短，随着值的增加，长度的差异变得更加明显。

## 2.ASCII 到十六进制

现在，让我们看看将 ASCII 值转换为十六进制的选项:

1.  将字符串转换为字符数组
2.  将每个`char`转换为一个`int`
3.  使用`Integer.toHexString()`将其转换为十六进制

这里有一个我们如何实现上述步骤的快速示例:

```
private static String asciiToHex(String asciiStr) {
    char[] chars = asciiStr.toCharArray();
    StringBuilder hex = new StringBuilder();
    for (char ch : chars) {
        hex.append(Integer.toHexString((int) ch));
    }

    return hex.toString();
}
```

## 3.十六进制到 ASCII 格式

同样，让我们分三步进行十六进制到 ASCII 格式的转换:

1.  在 2 个`char`组中剪切十六进制值
2.  使用`Integer.parseInt(hex, 16)`将其转换为基数为 16 的整数，并转换为`char`
3.  追加所有字符到一个`StringBuilder`

让我们看一个如何实现上述步骤的示例:

```
private static String hexToAscii(String hexStr) {
    StringBuilder output = new StringBuilder("");

    for (int i = 0; i < hexStr.length(); i += 2) {
        String str = hexStr.substring(i, i + 2);
        output.append((char) Integer.parseInt(str, 16));
    }

    return output.toString();
}
```

## 4.试验

最后，使用这些方法，让我们做一个快速测试:

```
@Test
public static void whenHexToAscii() {
    String asciiString = "www.baeldung.com";
    String hexEquivalent = 
      "7777772e6261656c64756e672e636f6d";

    assertEquals(asciiString, hexToAscii(hexEquivalent));
}

@Test
public static void whenAsciiToHex() {
    String asciiString = "www.baeldung.com";
    String hexEquivalent = 
      "7777772e6261656c64756e672e636f6d";

    assertEquals(hexEquivalent, asciiToHex(asciiString));
}
```

## 5.结论

最后，我们看了使用 Java 在 ASCII 和 Hex 之间进行转换的最简单的方法。

所有这些示例和代码片段的实现都可以在 github 项目中找到[——只需导入项目并按原样运行。](https://web.archive.org/web/20220120143609/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-2)