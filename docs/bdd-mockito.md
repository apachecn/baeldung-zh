# BDDMockito 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/bdd-mockito>

## 1。概述

**BDD 这个术语最早是由[丹·诺斯于 2006 年](https://web.archive.org/web/20220926190436/https://dannorth.net/introducing-bdd/)创造的。**

BDD 鼓励用自然的、人类可读的语言编写测试，关注应用程序的行为。

它定义了一种清晰的结构化方法来编写测试，遵循三个部分(安排、动作、断言):

*   `given`一些前提条件(安排)
*   一个动作发生了
*   `then`验证输出(断言)

**mock ITO 库附带了一个`BDDMockito`类，它引入了 BDD 友好的 API。**这个 API 允许我们采用一种更加 BDD 友好的方法，使用`given()`安排我们的测试，使用`then()`进行断言。

在本文中，我们将解释如何设置基于 BDD 的 Mockito 测试。我们还将讨论`Mockito` 和`BDDMockito`API 之间的差异，最终聚焦于`BDDMockito` API。

## 2。设置

### 2.1。Maven 依赖关系

**mock ITO 的 BDD 版本是`mockito-core`库**的一部分，为了开始，我们只需要包含工件:

```
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.21.0</version>
</dependency>
```

有关 Mockito 的最新版本，请查看 [Maven Central](https://web.archive.org/web/20220926190436/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.mockito%22%2C%20a%3A%22mockito-core%22) 。

### 2.2。进口

如果我们包含以下静态导入，我们的测试将变得更具可读性:

```
import static org.mockito.BDDMockito.*;
```

注意`BDDMockito`扩展了`Mockito`，所以我们不会错过传统`Mockito` API 提供的任何特性。

## 3\. Mockito vs. BDDMockito

Mockito 中的传统嘲讽是使用 *when(obj)* 来执行的。`then*()`在排列步骤中。

稍后，可以在 Assert 步骤中使用`verify()`来验证与我们的 mock 的交互。

**`BDDMockito`为各种`Mockito`方法提供了 BDD 别名，因此我们可以使用`given`(而不是`when`)编写我们的安排步骤，同样，我们可以使用`then`(而不是`verify`)编写我们的断言步骤。**

让我们看一个使用传统 Mockito 的测试体的例子:

```
when(phoneBookRepository.contains(momContactName))
  .thenReturn(false);

phoneBookService.register(momContactName, momPhoneNumber);

verify(phoneBookRepository)
  .insert(momContactName, momPhoneNumber);
```

让我们看看这与`BDDMockito`相比如何:

```
given(phoneBookRepository.contains(momContactName))
  .willReturn(false);

phoneBookService.register(momContactName, momPhoneNumber);

then(phoneBookRepository)
  .should()
  .insert(momContactName, momPhoneNumber);
```

## 4。`BDDMockito`嘲讽着

让我们试着测试一下`PhoneBookService`，我们需要模拟一下`PhoneBookRepository:`

```
public class PhoneBookService {
    private PhoneBookRepository phoneBookRepository;

    public void register(String name, String phone) {
        if(!name.isEmpty() && !phone.isEmpty()
          && !phoneBookRepository.contains(name)) {
            phoneBookRepository.insert(name, phone);
        }
    }

    public String search(String name) {
        if(!name.isEmpty() && phoneBookRepository.contains(name)) {
            return phoneBookRepository.getPhoneNumberByContactName(name);
        }
        return null;
    }
}
```

`BDDMockito` as `Mockito`允许我们返回一个固定或动态的值。它还允许我们抛出一个异常:

### 4.1。返回固定值

使用`BDDMockito,` ,我们可以很容易地将 Mockito 配置为每当调用我们的模拟对象目标方法时返回一个固定的结果:

```
given(phoneBookRepository.contains(momContactName))
  .willReturn(false);

phoneBookService.register(xContactName, "");

then(phoneBookRepository)
  .should(never())
  .insert(momContactName, momPhoneNumber);
```

### 4.2。返回一个动态值

`BDDMockito`允许我们提供一种更复杂的方式来返回值。我们可以根据输入返回一个动态结果:

```
given(phoneBookRepository.contains(momContactName))
  .willReturn(true);
given(phoneBookRepository.getPhoneNumberByContactName(momContactName))
  .will((InvocationOnMock invocation) ->
    invocation.getArgument(0).equals(momContactName) 
      ? momPhoneNumber 
      : null);
phoneBookService.search(momContactName);
then(phoneBookRepository)
  .should()
  .getPhoneNumberByContactName(momContactName); 
```

### 4.3。抛出异常

告诉 Mockito 抛出异常非常简单:

```
given(phoneBookRepository.contains(xContactName))
  .willReturn(false);
willThrow(new RuntimeException())
  .given(phoneBookRepository)
  .insert(any(String.class), eq(tooLongPhoneNumber));

try {
    phoneBookService.register(xContactName, tooLongPhoneNumber);
    fail("Should throw exception");
} catch (RuntimeException ex) { }

then(phoneBookRepository)
  .should(never())
  .insert(momContactName, tooLongPhoneNumber);
```

注意我们如何交换了`given`和`will*`的位置，这是强制性的，以防我们模仿一个没有返回值的方法。

还要注意，我们使用了类似(`any`， `eq`)的参数匹配器来提供一种基于标准而不是依赖于固定值的更通用的模拟方式。

## 5。结论

在这个快速教程中，我们讨论了 BDDMockito 如何试图将 BDD 相似性引入到我们的 Mockito 测试中，我们还讨论了`Mockito` 和`BDDMockito`之间的一些差异。

和往常一样，源代码可以在 GitHub 的[中找到——在测试包`com.baeldung.bddmockito`中。](https://web.archive.org/web/20220926190436/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito)