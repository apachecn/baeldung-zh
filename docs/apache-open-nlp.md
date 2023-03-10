# Apache OpenNLP 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-open-nlp>

## 1。概述

Apache OpenNLP 是一个开源的自然语言处理 Java 库。

它提供了一个 API，用于命名实体识别、句子检测、词性标注和标记化等用例。

在本教程中，我们将看看如何在不同的用例中使用这个 API。

## 2。Maven 设置

首先，我们需要将主依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.apache.opennlp</groupId>
    <artifactId>opennlp-tools</artifactId>
    <version>1.8.4</version>
</dependency>
```

最新的稳定版本可以在 [Maven Central](https://web.archive.org/web/20220724194915/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22opennlp-tools%22) 上找到。

一些用例需要经过训练的模型。你可以在这里下载预定义的模型[，在这里](https://web.archive.org/web/20220724194915/http://opennlp.sourceforge.net/models-1.5/)下载这些模型[的详细信息。](https://web.archive.org/web/20220724194915/https://opennlp.apache.org/models.html)

## 3。句子检测

先从理解什么是句子开始。

**句子检测是关于识别句子的开始和结束**，这通常取决于手边的语言。这也被称为“句子边界歧义消除”(SBD)。

在某些情况下，**句子检测相当具有挑战性，因为句点字符**的多义性。句号通常表示一个句子的结尾，但也可以出现在电子邮件地址、缩写、小数点和许多其他地方。

对于大多数 NLP 任务，对于句子检测，我们需要一个经过训练的模型作为输入，我们希望它位于`/resources`文件夹中。

为了实现句子检测，我们加载模型并将其传递到的实例中。然后，我们简单地将一个文本传递到`sentDetect()`方法中，以便在句子边界处分割它:

```java
@Test
public void givenEnglishModel_whenDetect_thenSentencesAreDetected() 
  throws Exception {

    String paragraph = "This is a statement. This is another statement." 
      + "Now is an abstract word for time, "
      + "that is always flying. And my email address is [[email protected]](/web/20220724194915/https://www.baeldung.com/cdn-cgi/l/email-protection)";

    InputStream is = getClass().getResourceAsStream("/models/en-sent.bin");
    SentenceModel model = new SentenceModel(is);

    SentenceDetectorME sdetector = new SentenceDetectorME(model);

    String sentences[] = sdetector.sentDetect(paragraph);
    assertThat(sentences).contains(
      "This is a statement.",
      "This is another statement.",
      "Now is an abstract word for time, that is always flying.",
      "And my email address is [[email protected]](/web/20220724194915/https://www.baeldung.com/cdn-cgi/l/email-protection)");
}
```

注意:后缀“ME”在 Apache OpenNLP 的许多类名中使用，代表一种基于“最大熵”的算法。

## 4。标记化

既然我们可以将一个文本语料库分成句子，我们就可以开始更详细地分析一个句子。

标记化的目标是将一个句子分成更小的部分，称为标记。通常，这些标记是单词、数字或标点符号。

OpenNLP 中有三种类型的标记化器。

### 4.1。使用`TokenizerME`

在这种情况下，我们首先需要加载模型。我们可以从[这里的](https://web.archive.org/web/20220724194915/http://opennlp.sourceforge.net/models-1.5/)下载模型文件，放在`/resources`文件夹中，然后从那里加载。

接下来，我们将使用加载的模型创建一个`TokenizerME`的实例，并使用`tokenize()`方法对任何`String:`执行标记化

```java
@Test
public void givenEnglishModel_whenTokenize_thenTokensAreDetected() 
  throws Exception {

    InputStream inputStream = getClass()
      .getResourceAsStream("/models/en-token.bin");
    TokenizerModel model = new TokenizerModel(inputStream);
    TokenizerME tokenizer = new TokenizerME(model);
    String[] tokens = tokenizer.tokenize("Baeldung is a Spring Resource.");

    assertThat(tokens).contains(
      "Baeldung", "is", "a", "Spring", "Resource", ".");
}
```

正如我们所看到的，记号赋予器已经将所有单词和句点字符识别为单独的记号。这个记号赋予器也可以与定制的训练模型一起使用。

### 4.2。`WhitespaceTokenizer`

顾名思义，这个记号赋予器只是使用空白字符作为分隔符将句子分割成记号:

```java
@Test
public void givenWhitespaceTokenizer_whenTokenize_thenTokensAreDetected() 
  throws Exception {

    WhitespaceTokenizer tokenizer = WhitespaceTokenizer.INSTANCE;
    String[] tokens = tokenizer.tokenize("Baeldung is a Spring Resource.");

    assertThat(tokens)
      .contains("Baeldung", "is", "a", "Spring", "Resource.");
  }
```

我们可以看到这个句子被空格分开了，因此我们得到了“Resource”(以句点字符结尾)作为单个标记，而不是单词“Resource”和句点字符的两个不同标记。

### 4.3。`SimpleTokenizer`

这个分词器比`WhitespaceTokenizer`稍微复杂一点，它把句子分成单词、数字和标点符号。这是默认行为，不需要任何模型:

```java
@Test
public void givenSimpleTokenizer_whenTokenize_thenTokensAreDetected() 
  throws Exception {

    SimpleTokenizer tokenizer = SimpleTokenizer.INSTANCE;
    String[] tokens = tokenizer
      .tokenize("Baeldung is a Spring Resource.");

    assertThat(tokens)
      .contains("Baeldung", "is", "a", "Spring", "Resource", ".");
  }
```

## 5。命名实体识别

现在我们已经理解了标记化，让我们看看基于成功标记化的第一个用例:命名实体识别(NER)。

NER 的目标是在给定的文本中找到命名的实体，如人、地点、组织和其他命名的事物。

OpenNLP 为人名、日期和时间、位置和组织使用预定义的模型。我们需要使用`TokenNameFinderModel` 和加载模型，并将其传递给`NameFinderME.`的实例，然后我们可以使用`find()`方法在给定文本中查找命名实体:

```java
@Test
public void 
  givenEnglishPersonModel_whenNER_thenPersonsAreDetected() 
  throws Exception {

    SimpleTokenizer tokenizer = SimpleTokenizer.INSTANCE;
    String[] tokens = tokenizer
      .tokenize("John is 26 years old. His best friend's "  
        + "name is Leonard. He has a sister named Penny.");

    InputStream inputStreamNameFinder = getClass()
      .getResourceAsStream("/models/en-ner-person.bin");
    TokenNameFinderModel model = new TokenNameFinderModel(
      inputStreamNameFinder);
    NameFinderME nameFinderME = new NameFinderME(model);
    List<Span> spans = Arrays.asList(nameFinderME.find(tokens));

    assertThat(spans.toString())
      .isEqualTo("[[0..1) person, [13..14) person, [20..21) person]");
}
```

正如我们在断言中看到的，结果是一个包含标记的开始和结束索引的`Span` 对象列表，这些标记组成了文本中的命名实体。

## 6.词性标注

另一个需要输入一系列标记的用例是词性标注。

**词类(POS)标识一个词的类型。** OpenNLP 对不同的词类使用以下标签:

*   **NN–**名词，单数或复数
*   **DT—**限定词
*   **VB–**动词，基本形式
*   VBD-动词，过去式
*   VBZ-动词，第三人称单数现在时
*   **IN—**介词或从属连词
*   **NNP–**专有名词，单数
*   **TO—**单词“TO”
*   **JJ—**形容词

这些标记与 Penn Tree Bank 中定义的标记相同。完整列表请参考[本列表](https://web.archive.org/web/20220724194915/https://www.ling.upenn.edu/courses/Fall_2003/ling001/penn_treebank_pos.html)。

类似于 NER 的例子，我们加载适当的模型，然后在一组标记上使用`POSTaggerME`及其方法`tag()`来标记句子:

```java
@Test
public void givenPOSModel_whenPOSTagging_thenPOSAreDetected() 
  throws Exception {

    SimpleTokenizer tokenizer = SimpleTokenizer.INSTANCE;
    String[] tokens = tokenizer.tokenize("John has a sister named Penny.");

    InputStream inputStreamPOSTagger = getClass()
      .getResourceAsStream("/models/en-pos-maxent.bin");
    POSModel posModel = new POSModel(inputStreamPOSTagger);
    POSTaggerME posTagger = new POSTaggerME(posModel);
    String tags[] = posTagger.tag(tokens);

    assertThat(tags).contains("NNP", "VBZ", "DT", "NN", "VBN", "NNP", ".");
}
```

`tag()`方法将令牌映射到一个 POS 标签列表中。示例中的结果是:

1.  “约翰”——NNP(专有名词)
2.  “有”——VBZ(动词)
3.  “a”——DT(限定词)
4.  “姐姐”——NN(名词)
5.  “命名”——VBZ(动词)
6.  “便士”——**NNP(专有名词)**
***   "."–句号**

 **## 7.词汇化

现在我们有了句子中标记的词性信息，我们可以进一步分析文本。

**词汇化是将一个可以带有时态、性别、语气或其他信息的词形****映射到该词的基本形式的过程——也称为其“词汇”**。

词条分类器将一个单词及其词性标签作为输入，并返回该单词的词条。因此，在词汇化之前，句子应该通过一个标记器和词性标注器。

Apache OpenNLP 提供了两种类型的词汇化:

*   **统计—**需要一个使用训练数据构建的词条分类器模型，以找到给定单词的词条
*   **基于字典-**需要一个包含单词、词性标签和相应词条的所有有效组合的字典

对于统计词汇化，我们需要训练一个模型，而对于字典词汇化，我们只需要一个像[这样的字典文件。](https://web.archive.org/web/20220724194915/https://raw.githubusercontent.com/richardwilly98/elasticsearch-opennlp-auto-tagging/master/src/main/resources/models/en-lemmatizer.dict)

让我们看一个使用字典文件的代码示例:

```java
@Test
public void givenEnglishDictionary_whenLemmatize_thenLemmasAreDetected() 
  throws Exception {

    SimpleTokenizer tokenizer = SimpleTokenizer.INSTANCE;
    String[] tokens = tokenizer.tokenize("John has a sister named Penny.");

    InputStream inputStreamPOSTagger = getClass()
      .getResourceAsStream("/models/en-pos-maxent.bin");
    POSModel posModel = new POSModel(inputStreamPOSTagger);
    POSTaggerME posTagger = new POSTaggerME(posModel);
    String tags[] = posTagger.tag(tokens);
    InputStream dictLemmatizer = getClass()
      .getResourceAsStream("/models/en-lemmatizer.dict");
    DictionaryLemmatizer lemmatizer = new DictionaryLemmatizer(
      dictLemmatizer);
    String[] lemmas = lemmatizer.lemmatize(tokens, tags);

    assertThat(lemmas)
      .contains("O", "have", "a", "sister", "name", "O", "O");
}
```

正如我们所见，我们得到了每个记号的引理。“O”表示无法确定该词条，因为该单词是专有名词。所以，我们没有“约翰”和“佩妮”的引理。

但是我们已经确定了句子中其他单词的引理:

*   曾经有过
*   答答
*   姐妹——姐妹
*   命名–名称

## 8.组块

词类信息在组块中也很重要——将句子分成语法上有意义的词组，如名词组或动词组。

与之前类似，我们在调用`chunk()`方法:之前，对句子进行标记，并对标记使用词性标注

```java
@Test
public void 
  givenChunkerModel_whenChunk_thenChunksAreDetected() 
  throws Exception {

    SimpleTokenizer tokenizer = SimpleTokenizer.INSTANCE;
    String[] tokens = tokenizer.tokenize("He reckons the current account 
      deficit will narrow to only 8 billion.");

    InputStream inputStreamPOSTagger = getClass()
      .getResourceAsStream("/models/en-pos-maxent.bin");
    POSModel posModel = new POSModel(inputStreamPOSTagger);
    POSTaggerME posTagger = new POSTaggerME(posModel);
    String tags[] = posTagger.tag(tokens);

    InputStream inputStreamChunker = getClass()
      .getResourceAsStream("/models/en-chunker.bin");
    ChunkerModel chunkerModel
     = new ChunkerModel(inputStreamChunker);
    ChunkerME chunker = new ChunkerME(chunkerModel);
    String[] chunks = chunker.chunk(tokens, tags);
    assertThat(chunks).contains(
      "B-NP", "B-VP", "B-NP", "I-NP", 
      "I-NP", "I-NP", "B-VP", "I-VP", 
      "B-PP", "B-NP", "I-NP", "I-NP", "O");
}
```

正如我们所看到的，我们从 chunker 获得了每个令牌的输出。“B”表示组块的开始，“I”表示组块的延续，“O”表示没有组块。

解析我们示例的输出，我们得到 6 个块:

1.  “他”——名词短语
2.  “计算”——动词短语
3.  “经常账户赤字”——名词短语
4.  “将缩小”——动词短语
5.  “to”——介词短语
6.  “只有 80 亿”——名词短语

## 9.语言检测

除了已经讨论过的用例， **OpenNLP 还提供了一个语言检测 API，允许识别某个文本的语言。**

对于语言检测，我们需要一个训练数据文件。这种文件包含用某种语言写的句子。每一行都用正确的语言标记，以便为机器学习算法提供输入。

用于语言检测的样本训练数据文件可在此处下载[。](https://web.archive.org/web/20220724194915/https://github.com/apache/opennlp/blob/master/opennlp-tools/src/test/resources/opennlp/tools/doccat/DoccatSample.txt)

我们可以将训练数据文件加载到一个`LanguageDetectorSampleStream,`中定义一些训练数据参数，创建一个模型，然后用这个模型来检测一个文本的语言:

```java
@Test
public void 
  givenLanguageDictionary_whenLanguageDetect_thenLanguageIsDetected() 
  throws FileNotFoundException, IOException {

    InputStreamFactory dataIn
     = new MarkableFileInputStreamFactory(
       new File("src/main/resources/models/DoccatSample.txt"));
    ObjectStream lineStream = new PlainTextByLineStream(dataIn, "UTF-8");
    LanguageDetectorSampleStream sampleStream
     = new LanguageDetectorSampleStream(lineStream);
    TrainingParameters params = new TrainingParameters();
    params.put(TrainingParameters.ITERATIONS_PARAM, 100);
    params.put(TrainingParameters.CUTOFF_PARAM, 5);
    params.put("DataIndexer", "TwoPass");
    params.put(TrainingParameters.ALGORITHM_PARAM, "NAIVEBAYES");

    LanguageDetectorModel model = LanguageDetectorME
      .train(sampleStream, params, new LanguageDetectorFactory());

    LanguageDetector ld = new LanguageDetectorME(model);
    Language[] languages = ld
      .predictLanguages("estava em uma marcenaria na Rua Bruno");
    assertThat(Arrays.asList(languages))
      .extracting("lang", "confidence")
      .contains(
        tuple("pob", 0.9999999950605625),
        tuple("ita", 4.939427661577956E-9), 
        tuple("spa", 9.665954064665144E-15),
        tuple("fra", 8.250349924885834E-25)));
}
```

结果是一个最可能的语言列表以及一个置信度分数。

并且，借助丰富的型号 ，我们可以通过这种类型的检测实现非常高的精度。

## 5.结论

我们在这里探索了很多，从 OpenNLP 有趣的功能。我们专注于一些有趣的功能来执行 NLP 任务，如词条化、词性标注、标记化、句子检测、语言检测等。

一如既往，以上所有的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220724194915/https://github.com/eugenp/tutorials/tree/master/apache-libraries)**