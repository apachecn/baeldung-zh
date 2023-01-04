# 在 Java 中按包含的数字对字符串排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sort-strings-contained-numbers>

## 1。简介

在本教程中，我们将看看如何按照字母数字`String`所包含的数字对它们进行排序。在按照剩余的数字字符对多个`Strings `进行排序之前，我们将重点关注从`String `中移除所有非数字字符。

我们将看看常见的边缘情况，包括空的`String`和无效的数字。

最后，我们将对我们的解决方案进行单元测试，以确保它按预期工作。

## 2。概述问题

在我们开始之前，我们需要描述我们希望我们的代码实现什么。对于这个特殊的问题，我们将做如下假设:

1.  我们的字符串可能只包含数字、字母或两者的混合。
2.  我们的字符串中的数字可以是整数或双精度数。
3.  当字符串中的数字被字母分隔时，我们应该去掉字母，将数字压缩在一起。例如，`2d3 `变成了`23.`
4.  为简单起见，当一个无效或缺失的数字出现时，我们应该将它们视为 0。

确定了这一点，让我们开始解决问题吧。

## 3。正则表达式解决方案

因为我们的第一步是在我们的输入`String, `中搜索数字模式，所以我们可以使用正则表达式，通常称为 regex。

我们首先需要的是正则表达式。我们希望保留输入`String`中的所有整数和小数点。我们可以通过以下方式实现我们的目标:

```
String DIGIT_AND_DECIMAL_REGEX = "[^\\d.]"

String digitsOnly = input.replaceAll(DIGIT_AND_DECIMAL_REGEX, "");
```

让我们简单解释一下发生了什么:

1.  `‘[^ ]'`–表示取反的集合，因此目标是未被包含的正则表达式指定的任何字符
2.  `‘\d'`–匹配任何数字字符(0–9)
3.  `‘.'`–匹配任意"。"性格；角色；字母

然后，我们使用`String.replaceAll `方法来删除 regex 没有指定的任何字符。这样做，就能保证我们目标的前三点能够实现。

接下来，我们需要添加一些条件来保证空的和无效的`Strings`返回 0，而有效的`Strings`返回一个有效的`Double`:

```
if("".equals(digitsOnly)) return 0;

try {
    return Double.parseDouble(digitsOnly);
} catch (NumberFormatException nfe) {
    return 0;
}
```

这就完成了我们的逻辑。剩下要做的就是将它插入一个比较器，这样我们就可以方便地对输入`Strings. `的`Lists `进行排序

让我们创建一个有效的方法来从任何我们想要的地方返回我们的比较器:

```
public static Comparator<String> createNaturalOrderRegexComparator() {
    return Comparator.comparingDouble(NaturalOrderComparators::parseStringToNumber);
}
```

## 4。测试，测试，测试

没有测试来验证其功能的代码有什么用？让我们建立一个快速的单元测试，以确保一切按计划进行:

```
List<String> testStrings = 
  Arrays.asList("a1", "d2.2", "b3", "d2.3.3d", "c4", "d2.f4",); // 1, 2.2, 3, 0, 4, 2.4

testStrings.sort(NaturalOrderComparators.createNaturalOrderRegexComparator());

List<String> expected = Arrays.asList("d2.3.3d", "a1", "d2.2", "d2.f4", "b3", "c4");

assertEquals(expected, testStrings);
```

在这个单元测试中，我们已经包含了我们计划的所有场景。无效的数字、整数、小数和字母分隔的数字都包含在我们的`testStrings `变量中。

## 5。结论

在这篇短文中，我们展示了如何根据字母数字串中的数字对它们进行排序——利用正则表达式来完成这项艰巨的工作。

我们已经处理了在解析输入字符串时可能出现的标准异常，并使用单元测试测试了不同的场景。

和往常一样，代码[可以在 GitHub 上找到。](https://web.archive.org/web/20221206190959/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-sorting-2)