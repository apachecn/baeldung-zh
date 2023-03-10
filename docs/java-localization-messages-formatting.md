# Java 本地化–格式化消息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-localization-messages-formatting>

## 1.介绍

在本教程中，我们将考虑如何基于`Locale`本地化和格式化消息。

我们将同时使用 Java 的`MessageFormat` 和第三方库`ICU.`

## 2.本地化用例

当我们的应用程序获得了来自世界各地的大量用户时，我们自然会希望根据用户的偏好显示不同的消息。

第一个也是最重要的方面是用户说的语言。其他可能包括货币、数字和日期格式。最后但并非最不重要的是文化偏好:对一个国家的用户来说可以接受的，对其他国家来说可能是无法忍受的。

假设我们有一个电子邮件客户端，我们希望在新邮件到达时显示通知。

这种消息的一个简单示例可能是:

```java
Alice has sent you a message.
```

这对于说英语的用户来说很好，但是那些不说英语的用户可能就不那么高兴了。例如，说法语的用户更喜欢看到以下消息:

```java
Alice vous a envoyé un message. 
```

虽然波兰人看到这个会很高兴:

```java
Alice wysłała ci wiadomość. 
```

如果我们想要一个格式正确的通知，即使 Alice 不只发送一条消息，而是发送几条消息，该怎么办？

我们可能会尝试通过在单个字符串中连接各个部分来解决这个问题，如下所示:

```java
String message = "Alice has sent " + quantity + " messages"; 
```

当我们需要通知时，情况很容易失控，因为不仅是 Alice，Bob 也可能发送消息:

```java
Bob has sent two messages.
Bob a envoyé deux messages.
Bob wysłał dwie wiadomości.
```

注意，在波兰语(`wysłała` vs `wysłał`)的情况下，动词是如何变化的。它说明了一个事实，即**平庸的字符串连接很少被本地化消息**所接受。

如我们所见，我们遇到两种类型的问题:**一种与翻译有关，另一种与格式**有关。让我们在接下来的部分中解决它们。

## 3.消息本地化

我们可以将应用程序的**本地化，或`l10n`定义为使应用程序适应用户舒适度**的过程。有时，也使用术语`internalization,` 或`i18n`。

为了本地化应用程序，首先，让我们通过将所有硬编码的消息移动到我们的`resources`文件夹中来消除它们:

[![](img/909c19bee2f3b2682b2458358f795ef8.png)](/web/20220524002247/https://www.baeldung.com/wp-content/uploads/2019/05/messages-localization.png)

每个文件都应该包含具有相应语言的消息的键值对。例如，文件`messages_en.properties`应该包含以下一对:

```java
label=Alice has sent you a message.
```

`messages_pl.properties`应该包含以下一对:

```java
label=Alice wysłała ci wiadomość.
```

类似地，其他文件给键`label`分配适当的值。现在，为了获取英语版本的通知，我们可以使用`ResourceBundle`:

```java
ResourceBundle bundle = ResourceBundle.getBundle("messages", Locale.UK);
String message = bundle.getString("label");
```

变量`message`的值将是`“Alice has sent you a message.”`

Java 的`Locale`类包含了常用语言和国家的快捷方式。

在波兰语的情况下，我们可以这样写:

```java
ResourceBundle bundle
  = ResourceBundle.getBundle("messages", Locale.forLanguageTag("pl-PL"));
String message = bundle.getString("label");
```

让我们提一下，如果我们不提供地区，那么系统将使用一个默认的。我们可能会在我们的文章“[Java 8 中的国际化和本地化](/web/20220524002247/https://www.baeldung.com/java-8-localization)”中详细介绍这个问题。然后，在可用的翻译中，系统将选择与当前活动语言环境最相似的翻译。

将消息放在资源文件中是使应用程序更加用户友好的一个好步骤。这使得翻译整个应用程序变得更加容易，原因如下:

1.  翻译人员不必在应用程序中寻找消息
2.  译者可以看到整个短语，这有助于把握上下文，从而促进更好的翻译
3.  当一种新语言的翻译就绪时，我们不必重新编译整个应用程序

## 4.消息格式

即使我们已经将消息从代码中移到一个单独的位置，它们仍然包含一些硬编码的信息。如果能够定制消息**中的名称和数字，使它们保持语法正确，那就太好了。**

我们可以将格式化定义为通过用占位符的值替换占位符来呈现字符串模板的过程。

在下面几节中，我们将考虑两种允许我们格式化消息的解决方案。

### 4.1.Java 的`MessageFormat`

为了格式化字符串，Java 在`java.lang.String` 中定义了[众多的格式化方法。但是，我们可以通过](/web/20220524002247/https://www.baeldung.com/string/format)`[java.text.format.MessageFormat](https://web.archive.org/web/20220524002247/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/MessageFormat.html)`获得更多支持。

为了说明这一点，让我们创建一个模式并将其提供给一个`MessageFormat`实例:

```java
String pattern = "On {0, date}, {1} sent you "
  + "{2, choice, 0#no messages|1#a message|2#two messages|2<{2, number, integer} messages}.";
MessageFormat formatter = new MessageFormat(pattern, Locale.UK); 
```

模式字符串有三个占位符的位置。

如果我们提供每个值:

```java
String message = formatter.format(new Object[] {date, "Alice", 2});
```

然后`MessageFormat `将填写模板并呈现我们的消息:

```java
On 27-Apr-2019, Alice sent you two messages.
```

### 4.2.`MessageFormat`语法

从上面的例子中，我们看到消息模式:

```java
pattern = "On {...}, {..} sent you {...}.";
```

包含占位符，这些占位符是带必需参数`index`和两个可选参数`type`和`style`的花括号`{…}`:

```java
{index}
{index, type}
{index, type, style}
```

占位符的索引对应于我们要插入的对象数组中元素的位置。

当存在时，`type`和`style`可以取以下值:

| 类型 | 风格 |
| --- | --- |
| `number` | `integer, currency, percent, custom format` |
| `date` | `short, medium, long, full, custom format` |
| `time` | `short, medium, long, full, custom format` |
| `choice` | `custom format` |

类型和风格的名称很大程度上说明了问题，但是我们可以参考官方文档来了解更多细节。

让我们仔细看看，在`custom format` `. `

在上例中，我们使用了以下格式表达式:

```java
{2, choice, 0#no messages|1#a message|2#two messages|2<{2, number, integer} messages}
```

通常，选择样式具有由竖线(或竖线)分隔的选项形式:

[![](img/1aea7dc9c11884e292650b2d603d700e.png)](/web/20220524002247/https://www.baeldung.com/wp-content/uploads/2019/05/message-format-syntax.png)

在选项内部，除了最后一个选项，匹配值`k[i]`和字符串`v[i]`用#分隔。注意，我们可以将其他模式嵌套到字符串`v[i]`中，就像我们对最后一个选项所做的那样:

```java
{2, choice, ...|2<{2, number, integer} messages}
```

**选择类型是一个基于数字的类型**，因此匹配值`k[i ]`有一个自然的排序，它将一个数字行分割成多个区间:

[![](img/a661f29e8827ce21b68ed272e2f9b50a.png)](/web/20220524002247/https://www.baeldung.com/wp-content/uploads/2019/05/choice-style-ordering.png)

如果我们给一个属于区间`[k[i], k[i+1])`的值`k`(包括左端，不包括右端)，那么选择值`v[i]`。

让我们更详细地考虑一下所选风格的范围。为此，我们采取这种模式:

```java
pattern = "You''ve got "
  + "{0, choice, 0#no messages|1#a message|2#two messages|2<{0, number, integer} messages}.";
```

并为其唯一的占位符传递不同的值:

| n | 消息 |
| --- | --- |
| `-1, 0, 0.5` | `You've got no messages.` |
| `1, 1.5` | `You've got a message.` |
| `2` | `You've got two messages.` |
| `2.5` | `You've got 2 messages.` |
| `5` | `You've got 5 messages.` |

### 4.3.让事情变得更好

所以，我们现在正在格式化我们的信息。但是，消息本身仍然是硬编码的。

从上一节中，我们知道应该将字符串模式提取到资源中。为了将我们的关注点分开，让我们创建另一组名为`formats`的资源文件:

[![](img/39d116bebfa3284af785a9e50ad3ac72.png)](/web/20220524002247/https://www.baeldung.com/wp-content/uploads/2019/05/messages-format.png)

在这些代码中，我们将创建一个名为`label`的键，其中包含特定于语言的内容。

例如，在英文版本中，我们将输入以下字符串:

```java
label=On {0, date, full} {1} has sent you 
  + {2, choice, 0#nothing|1#a message|2#two messages|2<{2,number,integer} messages}.
```

由于零消息的情况，我们应该稍微修改一下法语版本:

```java
label={0, date, short}, {1}{2, choice, 0# ne|0<} vous a envoyé 
  + {2, choice, 0#aucun message|1#un message|2#deux messages|2<{2,number,integer} messages}.
```

在波兰语和意大利语版本中，我们也需要做类似的修改。

事实上，波兰版本展示了另一个问题。根据波兰语(以及其他许多语言)的语法，动词必须与主语的性别一致。我们可以通过使用 choice 类型来解决这个问题，但是让我们考虑另一个解决方案。

### 4.4.重症监护室`MessageFormat`

让我们使用`International Components for Unicode` (ICU)库。我们已经在我们的[转换一个字符串为标题的案例](/web/20220524002247/https://www.baeldung.com/java-string-title-case#icu4j)教程中提到过。这是一个成熟且广泛使用的解决方案，允许我们为各种语言定制应用程序。

在这里，我们不打算探索它的全部细节。我们将把自己限制在玩具应用程序需要的范围内。要获得最全面和最新的信息，我们应该查看 [ICU 的官方网站](https://web.archive.org/web/20220524002247/http://site.icu-project.org/)。

在撰写本文时， [ICU for Java ( `ICU4J` )](https://web.archive.org/web/20220524002247/https://search.maven.org/search?q=g:com.ibm.icu%20AND%20a:icu4j) 的最新版本是 64.2。通常，为了开始使用它，我们应该将它作为一个依赖项添加到我们的项目中:

```java
<dependency>
    <groupId>com.ibm.icu</groupId>
    <artifactId>icu4j</artifactId>
    <version>64.2</version>
</dependency>
```

假设我们希望以各种语言并针对不同数量的消息获得格式正确的通知:

| 普通 | 英语 | 抛光剂 |
| --- | --- | --- |
| `0` | `Alice has sent you no messages.
Bob has sent you no messages.` | `Alice nie wysłała ci żadnej wiadomości.
Bob nie wysłał ci żadnej wiadomości.`  |
| `1` | `Alice has sent you a message.
Bob has sent you a message.` | `Alice wysłała ci wiadomość.
Bob wysłał ci wiadomość.`  |
| `> 1` | `Alice has sent you N messages.
Bob has sent you N messages.` | `Alice wysłała ci N wiadomości.
Bob wysłał ci N wiadomości.`  |

首先，我们应该在特定于地区的资源文件中创建一个模式。

让我们重新使用文件`formats.properties`并在那里添加一个键`label-icu`，其内容如下:

```java
label-icu={0} has sent you
  + {2, plural, =0 {no messages} =1 {a message}
  + other {{2, number, integer} messages}}.
```

它包含三个占位符，我们通过传递一个三元素数组来填充它们:

```java
Object[] data = new Object[] { "Alice", "female", 0 }
```

我们看到，在英文版本中，性别值占位符没有用，而在波兰语版本中:

```java
label-icu={0} {2, plural, =0 {nie} other {}}
+  {1, select, male {wysłał} female {wysłała} other {wysłało}} 
+  ci {2, plural, =0 {żadnych wiadomości} =1 {wiadomość}
+  other {{2, number, integer} wiadomości}}.
```

我们用它来区分`wysłał/wysłała/wysłało`。

## 5.结论

在本教程中，我们考虑了如何本地化和格式化向应用程序用户展示的消息。

和往常一样，本教程的代码片段在我们的 [GitHub 库](https://web.archive.org/web/20220524002247/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-strings)上。