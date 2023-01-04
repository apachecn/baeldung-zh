# Java 中罗马数字和阿拉伯数字之间的转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-roman-arabic>

## 1。简介

古罗马人发展了他们自己的数字系统，称为罗马数字。系统使用具有不同值的字母来表示数字。罗马数字今天仍然在一些次要的应用中使用。

在本教程中，我们将实现简单的转换器，将数字从一个系统转换到另一个系统。

## 2。罗马数字

在罗马体系中，我们有 7 个代表数字的符号:

*   `I`代表 1
*   `V`代表 5
*   `X`代表 10
*   `L`代表 50
*   `C`代表 100
*   `D`代表 500
*   `M`代表 1000

最初，人们用 IIII 代表 4，用 XXXX 代表 40。这读起来会很不舒服。也很容易把旁边的四个符号错当成三个符号。

**罗马数字使用减法记数法**来避免这样的错误。不要说`four times one` (IIII)，可以说是`one less than five` (IV)。

从我们的角度来看，这有多重要？这很重要，因为我们不是简单地一个符号一个符号地相加，我们可能需要检查下一个符号来确定这个数字是应该加还是应该减。

## 3。型号

让我们定义一个枚举来表示罗马数字:

```
enum RomanNumeral {
    I(1), IV(4), V(5), IX(9), X(10), 
    XL(40), L(50), XC(90), C(100), 
    CD(400), D(500), CM(900), M(1000);

    private int value;

    RomanNumeral(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    public static List<RomanNumeral> getReverseSortedValues() {
        return Arrays.stream(values())
          .sorted(Comparator.comparing((RomanNumeral e) -> e.value).reversed())
          .collect(Collectors.toList());
    }
}
```

注意，我们已经定义了额外的符号来帮助减法符号。我们还定义了一个名为`getReverseSortedValues()`的附加方法。

这个方法将允许我们显式地以降序检索定义的罗马数字。

## 4。罗马文到阿拉伯文

**罗马数字只能代表 1 到 4000 之间的整数**。我们可以使用下面的算法将罗马数字转换成阿拉伯数字(从`M`到`I`以相反的顺序遍历符号):

```
LET numeral be the input String representing an Roman Numeral
LET symbol be initialy set to RomanNumeral.values()[0]
WHILE numeral.length > 0:
    IF numeral starts with symbol's name:
        add symbol's value to the result
        remove the symbol's name from the numeral's beginning
    ELSE:
        set symbol to the next symbol
```

### 4.1。实施

接下来，我们可以用 Java 实现该算法:

```
public static int romanToArabic(String input) {
    String romanNumeral = input.toUpperCase();
    int result = 0;

    List<RomanNumeral> romanNumerals = RomanNumeral.getReverseSortedValues();

    int i = 0;

    while ((romanNumeral.length() > 0) && (i < romanNumerals.size())) {
        RomanNumeral symbol = romanNumerals.get(i);
        if (romanNumeral.startsWith(symbol.name())) {
            result += symbol.getValue();
            romanNumeral = romanNumeral.substring(symbol.name().length());
        } else {
            i++;
        }
    }

    if (romanNumeral.length() > 0) {
        throw new IllegalArgumentException(input + " cannot be converted to a Roman Numeral");
    }

    return result;
}
```

### 4.2。测试

最后，我们可以测试实现:

```
@Test
public void given2018Roman_WhenConvertingToArabic_ThenReturn2018() {
    String roman2018 = "MMXVIII";

    int result = RomanArabicConverter.romanToArabic(roman2018);

    assertThat(result).isEqualTo(2018);
}
```

## 5。阿拉伯语到罗马语

我们可以使用下面的算法将阿拉伯数字转换成罗马数字(从`M`到`I`以相反的顺序遍历符号):

```
LET number be an integer between 1 and 4000
LET symbol be RomanNumeral.values()[0]
LET result be an empty String
WHILE number > 0:
    IF symbol's value <= number:
        append the result with the symbol's name
        subtract symbol's value from number
    ELSE:
        pick the next symbol
```

### 5.1。实施

接下来，我们现在可以实现算法:

```
public static String arabicToRoman(int number) {
    if ((number <= 0) || (number > 4000)) {
        throw new IllegalArgumentException(number + " is not in range (0,4000]");
    }

    List<RomanNumeral> romanNumerals = RomanNumeral.getReverseSortedValues();

    int i = 0;
    StringBuilder sb = new StringBuilder();

    while ((number > 0) && (i < romanNumerals.size())) {
        RomanNumeral currentSymbol = romanNumerals.get(i);
        if (currentSymbol.getValue() <= number) {
            sb.append(currentSymbol.name());
            number -= currentSymbol.getValue();
        } else {
            i++;
        }
    }

    return sb.toString();
}
```

### 5.2。测试

最后，我们可以测试实现:

```
@Test
public void given1999Arabic_WhenConvertingToRoman_ThenReturnMCMXCIX() {
    int arabic1999 = 1999;

    String result = RomanArabicConverter.arabicToRoman(arabic1999);

    assertThat(result).isEqualTo("MCMXCIX");
}
```

## 6。结论

在这篇简短的文章中，我们展示了如何在罗马数字和阿拉伯数字之间进行转换。

我们使用了一个`enum` 来表示一组罗马数字，并且创建了一个实用程序类来执行转换。

完整的实现和所有测试可以在 GitHub 上找到[。](https://web.archive.org/web/20220626195347/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-3)