# 帕萨伊指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-passay>

## 1.介绍

如今，大多数 web 应用程序都有自己的密码策略——简单地说，就是强制用户创建难以破解的密码。

为了生成这样的密码或者验证它们，我们可以使用 [Passay 库](https://web.archive.org/web/20221129003433/http://www.passay.org/)。

## 2.Maven 依赖性

如果我们想在我们的项目中使用 Passay 库，有必要向我们的`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>org.passay</groupId>
    <artifactId>passay</artifactId>
    <version>1.3.1</version>
</dependency>
```

我们可以在这里找到它[。](https://web.archive.org/web/20221129003433/https://search.maven.org/search?q=g:org.passay%20AND%20a:passay&core=gav)

## 3.密码验证

密码验证是 Passay 库提供的两个主要功能之一。不费吹灰之力，很直观。让我们来发现它。

### 3.1.`PasswordData`

为了验证我们的密码，我们应该使用`PasswordData.` **它是一个容器，用于存放验证所需的信息。**它可以存储如下数据:

*   密码
*   用户名
*   密码参考列表
*   起源

密码和用户名属性可以自行解释。Passay 库给了我们`HistoricalReference`和`SourceReference`，我们可以将它们添加到密码引用列表中。

我们可以使用 origin 字段来保存关于密码是由用户生成还是定义的信息。

### 3.2.`PasswordValidator`

**我们应该知道我们需要`PasswordData`和`PasswordValidator`对象来开始验证密码。我们已经讨论过`PasswordData`。现在就来创造`PasswordValidator`吧。**

首先，我们应该定义一组密码验证规则。我们必须在创建一个`PasswordValidator`对象时将它们传递给构造函数:

```java
PasswordValidator passwordValidator = new PasswordValidator(new LengthRule(5));
```

有两种方法可以将我们的密码传递给一个`PasswordData`对象。我们将它传递给构造函数或 setter 方法:

```java
PasswordData passwordData = new PasswordData("1234");

PasswordData passwordData2 = new PasswordData();
passwordData.setPassword("1234");
```

我们可以通过调用`PasswordValidator`上的`validate()`方法来验证我们的密码:

```java
RuleResult validate = passwordValidator.validate(passwordData);
```

结果，我们将得到一个`RuleResult`对象。

### 3.3.`RuleResult`

`RuleResult`保存关于验证过程的有趣信息。这是`validate()`方法的结果。

首先，它可以告诉我们密码是否有效:

```java
Assert.assertEquals(false, validate.isValid());
```

**此外，我们还可以了解密码无效时会返回什么错误。**错误代码和验证描述保存在`RuleResultDetail`:

```java
RuleResultDetail ruleResultDetail = validate.getDetails().get(0);
Assert.assertEquals("TOO_SHORT", ruleResultDetail.getErrorCode());
Assert.assertEquals(5, ruleResultDetail.getParameters().get("minimumLength"));
Assert.assertEquals(5, ruleResultDetail.getParameters().get("maximumLength"));
```

最后，我们可以使用`RuleResultMetadata`探索密码验证的元数据:

```java
Integer lengthCount = validate
  .getMetadata()
  .getCounts()
  .get(RuleResultMetadata.CountCategory.Length);
Assert.assertEquals(Integer.valueOf(4), lengthCount);
```

## 4.密码生成

除了验证之外， [`Passay`](https://web.archive.org/web/20221129003433/https://search.maven.org/search?q=g:org.passay%20AND%20a:passay&core=gav) 库使我们能够生成密码。**我们可以提供发电机应该使用的规则。**

要生成密码，我们需要有一个`PasswordGenerator`对象。一旦我们有了它，我们就调用`generatePassword()`方法和`CharacterRules`的传递列表。下面是一个示例代码:

```java
CharacterRule digits = new CharacterRule(EnglishCharacterData.Digit);

PasswordGenerator passwordGenerator = new PasswordGenerator();
String password = passwordGenerator.generatePassword(10, digits);

Assert.assertTrue(password.length() == 10);
Assert.assertTrue(containsOnlyCharactersFromSet(password, "0123456789"));
```

要知道我们需要一个`CharacterData`的对象来创建`CharacterRule`。**另一个有趣的事实是，本库为我们提供了`EnglishCharacterData.`** 这是一个由五组字符组成的枚举:

*   数字
*   小写英文字母
*   大写英文字母
*   小写和大写集合的组合
*   特殊字符

然而，没有什么能阻止我们定义我们的字符集。这和实现`CharacterData`接口一样简单。让我们来看看我们该怎么做:

```java
CharacterRule specialCharacterRule = new CharacterRule(new CharacterData() {
    @Override
    public String getErrorCode() {
        return "SAMPLE_ERROR_CODE";
    }

    @Override
    public String getCharacters() {
        return "[[email protected]](/web/20221129003433/https://www.baeldung.com/cdn-cgi/l/email-protection)#";
    }
});

PasswordGenerator passwordGenerator = new PasswordGenerator();
String password = passwordGenerator.generatePassword(10, specialCharacterRule);

Assert.assertTrue(containsOnlyCharactersFromSet(password, "[[email protected]](/web/20221129003433/https://www.baeldung.com/cdn-cgi/l/email-protection)#"));
```

## 5.正匹配规则

我们已经学习了如何生成和验证密码。为此，我们需要定义一组规则。**因此，我们应该知道在`Passay`中有两种类型的规则可用:正匹配规则和负匹配规则。**

首先，让我们看看什么是积极的规则，以及我们如何使用它们。

肯定匹配规则接受包含提供的字符、正则表达式或符合某些限制的密码。

有六个正匹配规则:

*   `AllowedCharacterRule`–定义密码必须包含的所有字符
*   `AllowedRegexRule`–定义密码必须匹配的正则表达式
*   `CharacterRule`–定义一个字符集和密码中应包含的最少字符数
*   `LengthRule`–定义密码的最小长度
*   `CharacterCharacteristicsRule`–检查密码是否满足定义规则的`N`。
*   `LengthComplexityRule`–允许我们为不同的密码长度定义不同的规则

### 5.1.简单的正匹配规则

现在，我们将介绍所有具有简单配置的规则。它们定义了一组合法的字符或模式，或者一个可接受的密码长度。

下面是讨论规则的一个简短示例:

```java
PasswordValidator passwordValidator = new PasswordValidator(
  new AllowedCharacterRule(new char[] { 'a', 'b', 'c' }), 
  new CharacterRule(EnglishCharacterData.LowerCase, 5), 
  new LengthRule(8, 10)
);

RuleResult validate = passwordValidator.validate(new PasswordData("12abc"));

assertFalse(validate.isValid());
assertEquals(
  "ALLOWED_CHAR:{illegalCharacter=1, matchBehavior=contains}", 
  getDetail(validate, 0));
assertEquals(
  "ALLOWED_CHAR:{illegalCharacter=2, matchBehavior=contains}", 
  getDetail(validate, 1));
assertEquals(
  "TOO_SHORT:{minimumLength=8, maximumLength=10}", 
  getDetail(validate, 4));
```

**我们可以看到，如果密码无效，每条规则都给了我们明确的解释。**有通知说密码太短，有两个非法字符。我们还会注意到密码与提供的正则表达式不匹配。

更重要的是，我们被告知它包含的小写字母不足。

### 5.2.`CharacterCharacterisitcsRule`

`CharcterCharacterisitcsRule`比以前提出的规则更复杂。**要创建一个`CharcterCharacterisitcsRule`对象，我们需要提供一个`CharacterRule`s.``** 的列表，更重要的是，我们还要设置其中有多少个对象的密码必须匹配。我们可以这样做:

```java
CharacterCharacteristicsRule characterCharacteristicsRule = new CharacterCharacteristicsRule(
  3, 
  new CharacterRule(EnglishCharacterData.LowerCase, 5), 
  new CharacterRule(EnglishCharacterData.UpperCase, 5), 
  new CharacterRule(EnglishCharacterData.Digit),
  new CharacterRule(EnglishCharacterData.Special)
);
```

Presented `CharacterCharacteristicsRule`要求密码包含四个规则中的三个。

### 5.3.`LengthComplexityRule`

另一方面，`Passay`图书馆为我们提供了`LengthComplexityRule`。**它允许我们定义哪些规则应该应用于哪些长度的密码。**与`CharacterCharacteristicsRule`相反，他们允许我们使用各种规则——不仅仅是`CharacterRule`。

让我们来分析这个例子:

```java
LengthComplexityRule lengthComplexityRule = new LengthComplexityRule();
lengthComplexityRule.addRules("[1,5]", new CharacterRule(EnglishCharacterData.LowerCase, 5));
lengthComplexityRule.addRules("[6,10]", 
  new AllowedCharacterRule(new char[] { 'a', 'b', 'c', 'd' }));
```

正如我们看到的，对于有一到五个字符的密码，我们应用`CharacterRule`。但是对于包含六到十个字符的密码，我们希望密码匹配`AllowedCharacterRule`。

## 6.负匹配规则

**与肯定匹配规则不同，否定匹配规则拒绝包含提供的字符、正则表达式、条目等的密码。**

让我们来看看什么是负匹配规则:

*   `IllegalCharacterRule`–定义密码不得包含的所有字符
*   `IllegalRegexRule`–定义一个不匹配的正则表达式
*   `IllegalSequenceRule`–检查密码是否包含非法字符序列
*   `NumberRangeRule`–定义密码不能包含的数字范围
*   `WhitespaceRule`–检查密码是否包含空格
*   `DictionaryRule`–检查密码是否等于任何字典记录
*   `DictionarySubstringRule`–检查密码是否包含任何字典记录
*   `HistoryRule`–检查密码是否包含任何历史密码参考
*   `DigestHistoryRule`–检查密码是否包含任何摘要的历史密码参考
*   `SourceRule`–检查密码是否包含任何源密码参考
*   `DigestSourceRule`–检查密码是否包含任何摘要源密码参考
*   `UsernameRule`–检查密码是否包含用户名
*   `RepeatCharacterRegexRule`–检查密码是否包含重复的`ASCII`字符

### 6.1.简单的否定匹配规则

首先，我们将看看如何使用简单的规则，如`IllegalCharacterRule`、`IllegalRegexRule`等。这里有一个简短的例子:

```java
PasswordValidator passwordValidator = new PasswordValidator(
  new IllegalCharacterRule(new char[] { 'a' }), 
  new NumberRangeRule(1, 10), 
  new WhitespaceRule()
);

RuleResult validate = passwordValidator.validate(new PasswordData("abcd22 "));

assertFalse(validate.isValid());
assertEquals(
  "ILLEGAL_CHAR:{illegalCharacter=a, matchBehavior=contains}", 
  getDetail(validate, 0));
assertEquals(
  "ILLEGAL_NUMBER_RANGE:{number=2, matchBehavior=contains}", 
  getDetail(validate, 4));
assertEquals(
  "ILLEGAL_WHITESPACE:{whitespaceCharacter= , matchBehavior=contains}", 
  getDetail(validate, 5));
```

这个例子向我们展示了所描述的规则是如何工作的。与正匹配规则类似，它们给我们关于验证的完整反馈。

### 6.2.词典规则

如果我们想检查一个密码是否不等于提供的话。

因此，`Passay`库为我们提供了优秀的工具。让我们来发现`DictionaryRule`和`DictionarySubstringRule`:

```java
WordListDictionary wordListDictionary = new WordListDictionary(
  new ArrayWordList(new String[] { "bar", "foobar" }));

DictionaryRule dictionaryRule = new DictionaryRule(wordListDictionary);
DictionarySubstringRule dictionarySubstringRule = new DictionarySubstringRule(wordListDictionary);
```

我们可以看到字典规则使我们能够提供一个禁用单词的列表。当我们有一个最常见或最容易破解密码的列表时，这是非常有益的。因此，禁止用户使用它们是合理的。

在现实生活中，我们肯定会从文本文件或数据库中加载单词列表。那样的话，我们可以用`WordLists`。它有三个重载方法，接受一个数组`Reader`并创建`ArrayWordList`。

### 6.3.`HistoryRule`和`SourceRule`

此外，`Passay`库给了我们`HistoryRule`和`SourceRule`。他们可以根据不同来源的历史密码或文本内容来验证密码。

让我们来看看这个例子:

```java
SourceRule sourceRule = new SourceRule();
HistoryRule historyRule = new HistoryRule();

PasswordData passwordData = new PasswordData("123");
passwordData.setPasswordReferences(
  new PasswordData.SourceReference("source", "password"), 
  new PasswordData.HistoricalReference("12345")
);

PasswordValidator passwordValidator = new PasswordValidator(
  historyRule, sourceRule);
```

帮助我们检查密码以前是否被使用过。因为这样的做法是不安全的，我们不希望用户使用旧密码。

另一方面，`SourceRule` 允许我们检查密码是否与`SourceReferences`中提供的不同。我们可以避免在不同系统或应用程序中使用相同密码的风险。

值得一提的是，还有像`DigestSourceRule`和`DigestHistoryRule.`这样的规则，我们将在下一段中介绍它们。

### 6.4.摘要规则

在`Passay`库中有两个摘要规则:`DigestHistoryRule`和`DigestSourceRule`。**摘要规则适用于以摘要或哈希形式存储的密码。**因此，为了定义它们，我们需要提供一个`EncodingHashBean`对象。

让我们看看它是如何做到的:

```java
List<PasswordData.Reference> historicalReferences = Arrays.asList(
  new PasswordData.HistoricalReference(
    "SHA256",
    "2e4551de804e27aacf20f9df5be3e8cd384ed64488b21ab079fb58e8c90068ab"
));

EncodingHashBean encodingHashBean = new EncodingHashBean(
  new CodecSpec("Base64"), 
  new DigestSpec("SHA256"), 
  1, 
  false
); 
```

这一次我们通过一个标签和构造函数的编码密码来创建`HistoricalReference`。之后，我们用合适的编解码器和摘要算法实例化了`EncodingHashBean`。

此外，我们可以指定迭代次数以及算法是否加盐。

一旦我们有了编码 bean，我们就可以验证我们的摘要密码:

```java
PasswordData passwordData = new PasswordData("example!");
passwordData.setPasswordReferences(historicalReferences);

PasswordValidator passwordValidator = new PasswordValidator(new DigestHistoryRule(encodingHashBean));

RuleResult validate = passwordValidator.validate(passwordData);

Assert.assertTrue(validate.isValid());
```

我们可以在[洞穴图书馆网页](https://web.archive.org/web/20221129003433/http://www.cryptacular.org/about.html)了解更多关于`EncodingHashinBean`的信息。

### 6.5.`RepeatCharacterRegexRule`

另一个有趣的验证规则是`RepeatCharacterRegexRule`。**我们可以用它来检查密码是否包含重复的`ASCII`字符。**

下面是一个示例代码:

```java
PasswordValidator passwordValidator = new PasswordValidator(new RepeatCharacterRegexRule(3));

RuleResult validate = passwordValidator.validate(new PasswordData("aaabbb"));

assertFalse(validate.isValid());
assertEquals("ILLEGAL_MATCH:{match=aaa, pattern=([^\\x00-\\x1F])\\1{2}}", getDetail(validate, 0));
```

### 6.6.`UsernameRule`

本章我们要讨论的最后一个规则是`UsernameRule`。**它使我们能够禁止在密码中使用用户名。**

正如我们之前所学的，我们应该将用户名存储在`PasswordData`中:

```java
PasswordValidator passwordValidator = new PasswordValidator(new UsernameRule());

PasswordData passwordData = new PasswordData("testuser1234");
passwordData.setUsername("testuser");

RuleResult validate = passwordValidator.validate(passwordData);

assertFalse(validate.isValid());
assertEquals("ILLEGAL_USERNAME:{username=testuser, matchBehavior=contains}", getDetail(validate, 0));
```

## 7.定制消息

`Passay`库使我们能够定制验证规则返回的消息。**首先，我们应该定义消息，并将它们分配给错误代码。**

我们可以把它们放入一个简单的文件中。让我们看看有多简单:

```java
TOO_LONG=Password must not have more characters than %2$s.
TOO_SHORT=Password must not contain less characters than %2$s.
```

一旦我们有消息，我们必须加载该文件。最后，我们可以将它传递给`PasswordValidator`对象。

下面是一个示例代码:

```java
URL resource = this.getClass().getClassLoader().getResource("messages.properties");
Properties props = new Properties();
props.load(new FileInputStream(resource.getPath()));

MessageResolver resolver = new PropertiesMessageResolver(props); 
```

正如我们所看到的，我们已经加载了`message.properties`文件并将其传递给了`Properties`对象。然后，我们可以使用`Properties`对象来创建`PropertiesMessageResolver`。

让我们看一下如何使用消息解析器的例子:

```java
PasswordValidator validator = new PasswordValidator(
  resolver, 
  new LengthRule(8, 16), 
  new WhitespaceRule()
);

RuleResult tooShort = validator.validate(new PasswordData("XXXX"));
RuleResult tooLong = validator.validate(new PasswordData("ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ"));

Assert.assertEquals(
  "Password must not contain less characters than 16.", 
  validator.getMessages(tooShort).get(0));
Assert.assertEquals(
  "Password must not have more characters than 16.", 
  validator.getMessages(tooLong).get(0));
```

这个例子清楚地表明，我们可以用配备了消息解析器的验证器来翻译所有的错误代码。

## 8.结论

在本教程中，我们学习了如何使用`Passay`库。我们已经分析了几个例子，说明了这个库如何能够很容易地用于密码验证。提供的规则涵盖了确保密码安全的大多数常见方式。

但是我们应该记住，Passay 库本身并不能保证我们的密码安全。首先，我们应该了解什么是通用规则，然后使用库来实现它们。

所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221129003433/https://github.com/eugenp/tutorials/tree/master/libraries-security)