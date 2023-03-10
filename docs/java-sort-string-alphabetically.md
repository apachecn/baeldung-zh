# 在 Java 中按字母顺序对字符串排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sort-string-alphabetically>

## 1.概观

在本教程中，我们将展示如何按字母顺序对`String`进行排序。

我们想这么做可能有很多原因——其中之一可能是验证两个单词是否由相同的字符集组成。这样，我们就能验证它们是否是变位词。

## 2.对字符串排序

在内部，`String`使用一个字符数组来操作。因此，我们可以利用 **`toCharArray() : char[]`方法，对数组进行排序，并根据结果创建一个新的`String`:**

```java
@Test
void givenString_whenSort_thenSorted() {
    String abcd = "bdca";
    char[] chars = abcd.toCharArray();

    Arrays.sort(chars);
    String sorted = new String(chars);

    assertThat(sorted).isEqualTo("abcd");
}
```

在 Java 8 中，我们可以利用`Stream` API 为我们排序`String`:

```java
@Test
void givenString_whenSortJava8_thenSorted() {
    String sorted = "bdca".chars()
      .sorted()
      .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
      .toString();

    assertThat(sorted).isEqualTo("abcd");
}
```

这里，我们使用与第一个例子相同的算法，但是使用`Stream sorted()`方法对 char 数组进行排序。

注意，字符**是按照其 ASCII 码**排序的，因此，大写字母总是出现在开头。因此，如果我们想对“abC”进行排序，排序的结果将是“Cab”。

为了解决这个问题，我们需要用的`toLowerCase()`方法对字符串进行**转换。我们将在我们的变位验证器示例中这样做。**

## 3.测试

为了测试排序，我们将构建变位词验证器。如上所述，当两个不同的单词或句子是同一组字符的复合时，就会出现变位词。

让我们来看看我们的`AnagramValidator`课:

```java
public class AnagramValidator {

    public static boolean isValid(String text, String anagram) {
        text = prepare(text);
        anagram = prepare(anagram);

        String sortedText = sort(text);
        String sortedAnagram = sort(anagram);

        return sortedText.equals(sortedAnagram);
    }

    private static String sort(String text) {
        char[] chars = prepare(text).toCharArray();

        Arrays.sort(chars);
        return new String(chars);
    }

    private static String prepare(String text) {
        return text.toLowerCase()
          .trim()
          .replaceAll("\\s+", "");
    }
}
```

现在，我们将利用我们的排序方法，验证变位词是否有效:

```java
@Test
void givenValidAnagrams_whenSorted_thenEqual() {
    boolean isValidAnagram = AnagramValidator.isValid("Avida Dollars", "Salvador Dali");

    assertTrue(isValidAnagram);
}
```

## 4.结论

在这篇简短的文章中，我们展示了如何以两种方式按字母顺序对`String`进行排序。此外，我们还实现了变位词验证器，它利用了字符串排序方法。

像往常一样，完整的代码可以在 [GitHub 项目](https://web.archive.org/web/20221208143926/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-sorting-2)上获得。