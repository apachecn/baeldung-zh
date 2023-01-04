# JUnit 自定义显示名称生成器 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-custom-display-name-generator>

## 1.概观

JUnit 5 对定制测试类和测试方法名称有很好的支持。在这个快速教程中，我们将看到如何通过`@DisplayNameGeneration`注释使用 JUnit 5 定制显示名称生成器。

## 2.显示名称生成

我们可以通过`@DisplayNameGeneration`注释来**配置定制的显示名称生成器。然而，最好注意到`@DisplayName`注释总是优先于任何显示名称生成器。**

首先，JUnit 5 提供了一个`DisplayNameGenerator.ReplaceUnderscores`类，用空格替换名称中的任何下划线。让我们来看一个例子:

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class ReplaceUnderscoresGeneratorUnitTest {

    @Nested
    class when_doing_something {

        @Test
        void then_something_should_happen() {
        }

        @Test
        @DisplayName("@DisplayName takes precedence over generation")
        void override_generator() {
        }
    }
}
```

现在，当我们运行测试时，我们可以看到显示名称的生成使得测试输出更具可读性:

```java
└─ ReplaceUnderscoresGeneratorUnitTest ✓
   └─ when doing something ✓
      ├─ then something should happen() ✓
      └─ @DisplayName takes precedence over generation ✓
```

## 3.自定义显示名称生成器

为了编写一个定制的显示名称生成器，我们必须**编写一个类来实现`DisplayNameGenerator`接口**中的方法。该接口具有用于为类、嵌套类和方法生成名称的方法。

### 3.1.驼色外壳更换

让我们从一个简单的显示名称生成器开始，它用可读的句子替换 camel case 名称。首先，我们可以**扩展`DisplayNameGenerator.Standard`类**:

```java
 static class ReplaceCamelCase extends DisplayNameGenerator.Standard {
        @Override
        public String generateDisplayNameForClass(Class<?> testClass) {
            return replaceCamelCase(super.generateDisplayNameForClass(testClass));
        }

        @Override
        public String generateDisplayNameForNestedClass(Class<?> nestedClass) {
            return replaceCamelCase(super.generateDisplayNameForNestedClass(nestedClass));
        }

        @Override
        public String generateDisplayNameForMethod(Class<?> testClass, Method testMethod) {
            return this.replaceCamelCase(testMethod.getName()) + 
              DisplayNameGenerator.parameterTypesAsString(testMethod);
        }

        String replaceCamelCase(String camelCase) {
            StringBuilder result = new StringBuilder();
            result.append(camelCase.charAt(0));
            for (int i=1; i<camelCase.length(); i++) {
                if (Character.isUpperCase(camelCase.charAt(i))) {
                    result.append(' ');
                    result.append(Character.toLowerCase(camelCase.charAt(i)));
                } else {
                    result.append(camelCase.charAt(i));
                }
            }
            return result.toString();
        }
    }
```

在上面的例子中，我们可以看到生成显示名称不同部分的方法。

让我们为我们的生成器编写一个测试:

```java
@DisplayNameGeneration(DisplayNameGeneratorUnitTest.ReplaceCamelCase.class)
class DisplayNameGeneratorUnitTest {

    @Test
    void camelCaseName() {
    }
}
```

接下来，当运行测试时，我们可以看到**camel 案例名称已经被替换为可读的句子**:

```java
└─ Display name generator unit test ✓
   └─ camel case name() ✓
```

### 3.2.指示句

到目前为止，我们已经讨论了非常简单的用例。然而，我们可以变得更有创造力:

```java
 static class IndicativeSentences extends ReplaceCamelCase {
        @Override
        public String generateDisplayNameForNestedClass(Class<?> nestedClass) {
            return super.generateDisplayNameForNestedClass(nestedClass) + "...";
        }

        @Override
        public String generateDisplayNameForMethod(Class<?> testClass, Method testMethod) {
            return replaceCamelCase(testClass.getSimpleName() + " " + testMethod.getName()) + ".";
        }
    }
```

这里的想法是**从嵌套的类和测试方法**中创建指示语句。换句话说，嵌套的类名将被添加到测试方法名的前面:

```java
class DisplayNameGeneratorUnitTest {

    @Nested
    @DisplayNameGeneration(DisplayNameGeneratorUnitTest.IndicativeSentences.class)
    class ANumberIsFizz {
        @Test
        void ifItIsDivisibleByThree() {
        }

        @ParameterizedTest(name = "Number {0} is fizz.")
        @ValueSource(ints = { 3, 12, 18 })
        void ifItIsOneOfTheFollowingNumbers(int number) {
        }
    }

    @Nested
    @DisplayNameGeneration(DisplayNameGeneratorUnitTest.IndicativeSentences.class)
    class ANumberIsBuzz {
        @Test
        void ifItIsDivisibleByFive() {
        }

        @ParameterizedTest(name = "Number {0} is buzz.")
        @ValueSource(ints = { 5, 10, 20 })
        void ifItIsOneOfTheFollowingNumbers(int number) {
        }
    }
}
```

看看这个例子，我们使用嵌套类作为测试方法的上下文。为了更好地说明结果，让我们运行测试:

```java
└─ Display name generator unit test ✓
   ├─ A number is buzz... ✓
   │  ├─ A number is buzz if it is one of the following numbers. ✓
   │  │  ├─ Number 5 is buzz. ✓
   │  │  ├─ Number 10 is buzz. ✓
   │  │  └─ Number 20 is buzz. ✓
   │  └─ A number is buzz if it is divisible by five. ✓
   └─ A number is fizz... ✓
      ├─ A number is fizz if it is one of the following numbers. ✓
      │  ├─ Number 3 is fizz. ✓
      │  ├─ Number 12 is fizz. ✓
      │  └─ Number 18 is fizz. ✓
      └─ A number is fizz if it is divisible by three. ✓
```

正如我们所看到的，生成器将嵌套的类名和测试方法名结合起来，创建了指示性的句子。

## 4.结论

在本教程中，我们看到了如何使用@ `DisplayNameGeneration`注释来为我们的测试生成显示名称。此外，我们编写了自己的`DisplayNameGenerator`来定制显示名称的生成。

和往常一样，本文中使用的例子可以在 [GitHub 项目](https://web.archive.org/web/20220525142147/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-advanced)中找到。