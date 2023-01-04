# Java 中的基本计算器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-basic-calculator>

## 1.概观

在本教程中，我们将用 Java 实现一个支持加、减、乘、除运算的基本计算器。

我们还将操作符和操作数作为输入，并基于它们处理计算。

## 2.基本设置

首先，让我们展示一些关于计算器的信息:

```
System.out.println("---------------------------------- \n" +
  "Welcome to Basic Calculator \n" +
  "----------------------------------");
System.out.println("Following operations are supported : \n" +
  "1\. Addition (+) \n" +
  "2\. Subtraction (-) \n" +
  "3\. Multiplication (*) \n" +
  "4\. Division (/) \n");
```

现在，让我们用 [`java.util.Scanner`](/web/20220627180756/https://www.baeldung.com/java-scanner) 来取用户输入:

```
Scanner scanner = new Scanner(System.in);

System.out.println("Enter an operator: (+ OR - OR * OR /) ");
char operation = scanner.next().charAt(0);

System.out.println("Enter the first number: ");
double num1 = scanner.nextDouble();

System.out.println("Enter the second number: ");
double num2 = scanner.nextDouble();
```

当我们向系统中输入信息时，我们需要验证它们。例如，如果操作符不是+、-、*或/，那么我们的计算器应该会调出错误的输入。同样，如果我们输入第二个数为 0 进行除法运算，结果也不会好。

所以，让我们来实现这些验证。

首先，让我们集中讨论操作员无效时的情况:

```
if (!(operation == '+' || operation == '-' || operation == '*' || operation == '/')) {
    System.err.println("Invalid Operator. Please use only + or - or * or /");
}
```

然后我们可以显示无效操作的错误:

```
if (operation == '/' && num2 == 0.0) {
    System.err.println("The second number cannot be zero for division operation.");
}
```

首先验证用户输入。之后，计算结果将显示为:

```
<number1><operation><number2>=
```

## 3.处理计算

首先，我们可以使用一个`[if-else](/web/20220627180756/https://www.baeldung.com/java-if-else)`构造来处理计算

```
if (operation == '+') {
    System.out.println(num1 + " + " + num2 + " = " + (num1 + num2));
} else if (operation == '-') {
    System.out.println(num1 + " - " + num2 + " = " + (num1 - num2));
} else if (operation == '*') {
    System.out.println(num1 + " x " + num2 + " = " + (num1 * num2));
} else if (operation == '/') {
    System.out.println(num1 + " / " + num2 + " = " + (num1 / num2));
} else {
    System.err.println("Invalid Operator Specified.");
}
```

同样，我们可以用一个 Java [`switch`](/web/20220627180756/https://www.baeldung.com/java-switch) 语句:

```
switch (operation) {
    case '+':
        System.out.println(num1 + " + " + num2 + " = " + (num1 + num2));
        break;
    case '-':
        System.out.println(num1 + " - " + num2 + " = " + (num1 - num2));
        break;
    case '*':
        System.out.println(num1 + " x " + num2 + " = " + (num1 * num2));
        break;
    case '/':
        System.out.println(num1 + " / " + num2 + " = " + (num1 / num2));
        break;
    default:
        System.err.println("Invalid Operator Specified.");
        break;
}
```

我们可以使用一个变量来存储计算结果。这样一来，最后就可以打印出来了。在这种情况下，`System.out.println`只会被使用一次。

此外，计算的最大范围是 2147483647。因此，如果我们超过它，我们将从一个`int`数据类型溢出。因此，它应该存储在一个[更大的数据类型](/web/20220627180756/https://www.baeldung.com/java-primitives)的变量中，例如一个`double`数据类型。

## 4.结论

在本教程中，我们用 Java 实现了一个基本的计算器，使用了两种不同的结构。我们还确保在进一步处理输入之前对其进行验证。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220627180756/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math)