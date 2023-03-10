# Java 中的单一责任原则

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-single-responsibility-principle>

## 1.概观

在本教程中，我们将讨论单一责任原则，作为面向对象编程的坚实原则之一。

总的来说，我们将深入探讨这个原则是什么，以及在设计我们的软件时如何实现它。此外，我们将解释这一原则何时会产生误导。

`*SRP = Single Responsibility Principle`

## 2.单一责任原则

顾名思义，这个原则规定**每个类应该有** **一个职责，一个单一的目的**。这意味着一个类将只做一项工作，这使我们得出结论，它应该只有**一个理由来改变**。

我们不希望对象知道的太多，行为不相关。这些类更难维护。例如，如果我们有一个因为不同原因而经常改变的类，那么这个类应该被分成更多的类，每个类处理一个单独的问题。当然，如果出现错误，会更容易发现。

让我们考虑一个包含以某种方式改变文本的代码的类。这个类唯一的工作应该是**操纵文本**。

```java
public class TextManipulator {
    private String text;

    public TextManipulator(String text) {
        this.text = text;
    }

    public String getText() {
        return text;
    }

    public void appendText(String newText) {
        text = text.concat(newText);
    }

    public String findWordAndReplace(String word, String replacementWord) {
        if (text.contains(word)) {
            text = text.replace(word, replacementWord);
        }
        return text;
    }

    public String findWordAndDelete(String word) {
        if (text.contains(word)) {
            text = text.replace(word, "");
        }
        return text;
    }

    public void printText() {
        System.out.println(textManipulator.getText());
    }
}
```

虽然这看起来不错，但它不是 SRP 的一个好例子。这里我们有**两个** **职责** : **操纵和打印文本**。

在这个类中使用打印文本的方法违反了单一责任原则。为此，我们应该创建另一个类，它只处理打印文本:

```java
public class TextPrinter {
    TextManipulator textManipulator;

    public TextPrinter(TextManipulator textManipulator) {
        this.textManipulator = textManipulator;
    }

    public void printText() {
        System.out.println(textManipulator.getText());
    }

    public void printOutEachWordOfText() {
        System.out.println(Arrays.toString(textManipulator.getText().split(" ")));
    }

    public void printRangeOfCharacters(int startingIndex, int endIndex) {
        System.out.println(textManipulator.getText().substring(startingIndex, endIndex));
    }
}
```

现在，在这个类中，我们可以为我们想要的打印文本的各种变化创建方法，因为这是它的工作。

## 3.这个原则怎么会误导人呢？

在我们的软件中实现 SRP 的诀窍是知道每个类的职责。

然而，**每个开发者都有他们对类目的**的看法，这使得事情变得棘手。由于我们没有关于如何实施这一原则的严格指示，我们只能解释责任是什么。

这意味着，有时只有我们，作为应用程序的设计者，才能决定某个东西是否在类的范围内。

当根据 SRP 原则编写一个类时，我们必须考虑问题域、业务需求和应用程序架构。这是非常主观的，这使得实施这一原则比看起来更难。它不会像本教程中的例子那么简单。

这就引出了下一点。

## 4.内聚力

遵循 SRP 原则，我们的类将遵循一个功能。他们的方法和数据都有一个明确的目的。这意味着 **[高内聚](/web/20220626103954/https://www.baeldung.com/cs/cohesion-vs-coupling)、**以及**健壮性，它们共同减少了错误**。

当基于 SRP 原则设计软件时，内聚性是必不可少的，因为它帮助我们找到类的单一职责。这个概念也帮助我们找到有不止一个责任的类。

让我们回到我们的`TextManipulator`类方法:

```java
...

public void appendText(String newText) {
    text = text.concat(newText);
}

public String findWordAndReplace(String word, String replacementWord) {
    if (text.contains(word)) {
        text = text.replace(word, replacementWord);
    }
    return text;
}

public String findWordAndDelete(String word) {
    if (text.contains(word)) {
        text = text.replace(word, "");
    }
    return text;
}

...
```

这里，我们对这个类的功能有一个清晰的描述:文本操作。

但是，如果我们不考虑内聚性，我们对这个类的责任没有一个清晰的定义，我们可以说写作和更新文本是两个不同的和独立的工作。在这种思想的引导下，我们可以得出结论，这应该是两个独立的类:`WriteText`和`UpdateText`。

实际上，我们会得到**两个紧密耦合而松散结合的类**，它们应该总是一起使用。这三种方法可能**执行不同的操作，但是它们本质上服务于一个目的**:文本操作。关键是不要想太多。

有助于实现方法高内聚的工具之一是 LCOM。本质上， **LCOM 测量类组件之间的连接以及它们彼此之间的关系。**

马丁·伊茨和贝扎德·蒙塔泽里推出了 lco M4 T1，这款产品曾一度被 T2 的 sonarqueb T3 计量，但后来被弃用。

## 5.结论

尽管该原则的名称是不言自明的，但我们可以看到错误地实现它是多么容易。在开发项目时，一定要区分每个类的责任，并特别注意内聚性。

和往常一样，代码可以在 GitHub 上获得[。](https://web.archive.org/web/20220626103954/https://github.com/eugenp/tutorials/tree/master/patterns/solid/)