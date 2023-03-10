# 面向 Java 的 Tensorflow 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/tensorflow-java>

## 1.概观

[TensorFlow](https://web.archive.org/web/20220626083438/https://www.tensorflow.org/) 是一个用于数据流编程的**开源库。这最初是由 Google 开发的，可用于多种平台。虽然 TensorFlow 可以在单核上工作，但它可以像**一样轻松受益于多个 CPU、GPU 或可用的 TPU**。**

在本教程中，我们将介绍 TensorFlow 的基础知识以及如何在 Java 中使用它。请注意，TensorFlow Java API 是一个实验性的 API，因此不包含在任何稳定性保证中。我们将在教程的后面讨论使用 TensorFlow Java API 的可能用例。

## 2.基础

张量流计算基本上围绕**两个基本概念:图和会话**。让我们快速浏览一遍，以获得完成本教程其余部分所需的背景知识。

### 2.1\. TensorFlow Graph

首先，让我们了解张量流程序的基本构件。**计算用 TensorFlow** 中的图形表示。图通常是操作和数据的有向非循环图，例如:

 [![TensorFlow-Graph-1-1](img/7d8796b98349a600f82785d2921efcc4.png)](/web/20220626083438/https://www.baeldung.com/wp-content/uploads/2019/03/TensorFlow-Graph-1-1.jpg)

上图显示了以下等式的计算图:

```java
f(x, y) = z = a*x + b*y
```

张量流计算图由两个元素组成:

1.  **张量:这些是 TensorFlow 中数据的核心单位。**它们被表示为计算图中的边，描述了数据在图中的流动。张量可以有任意维数的形状。张量的维数通常被称为它的秩。所以标量是秩为 0 的张量，向量是秩为 1 的张量，矩阵是秩为 2 的张量，以此类推。
2.  操作:这些是计算图中的节点。它们指的是可以发生在输入运算的张量上的各种各样的计算。它们也经常产生张量，这些张量是从计算图形的运算中产生的。

### 2.2.张量流会话

现在，一个张量流图仅仅是一个计算的示意图，它实际上不包含任何值。此类**图必须在所谓的 TensorFlow 会话中运行，以便对图中的张量进行评估**。会话可以从图中获取一组张量作为输入参数进行计算。然后它在图中向后运行，运行所有必要的节点来计算这些张量。

有了这些知识，我们现在就可以把它应用到 Java API 中了！

## 3.Maven 设置

我们将建立一个快速 Maven 项目，用 Java 创建并运行 TensorFlow 图。我们只需要 [`tensorflow`依赖](https://web.archive.org/web/20220626083438/https://search.maven.org/search?q=g:org.tensorflow%20AND%20a:tensorflow%20AND%20v:1.12.0):

```java
<dependency>
    <groupId>org.tensorflow</groupId>
    <artifactId>tensorflow</artifactId>
    <version>1.12.0</version>
</dependency>
```

## 4.创建图表

现在让我们尝试使用 TensorFlow Java API 来构建我们在上一节中讨论过的图表。更准确地说，在本教程中，我们将使用 TensorFlow Java API 来求解由以下等式表示的函数:

```java
z = 3*x + 2*y
```

第一步是声明和初始化一个图形:

```java
Graph graph = new Graph()
```

现在，我们必须定义所有需要的操作。记住，TensorFlow 中的**操作消耗并产生零个或多个张量**。此外，图中的每个节点都是一个包含常量和占位符的操作。这可能看起来违背直觉，但是请忍耐一下！

类`Graph`有一个名为`opBuilder()`的通用函数，可以在 TensorFlow 上构建任何类型的操作。

### 4.1.定义常数

首先，让我们在上图中定义常量操作。请注意，**常数运算将需要一个张量作为其值**:

```java
Operation a = graph.opBuilder("Const", "a")
  .setAttr("dtype", DataType.fromClass(Double.class))
  .setAttr("value", Tensor.<Double>create(3.0, Double.class))
  .build();		
Operation b = graph.opBuilder("Const", "b")
  .setAttr("dtype", DataType.fromClass(Double.class))
  .setAttr("value", Tensor.<Double>create(2.0, Double.class))
  .build();
```

这里，我们定义了一个常量类型的`Operation`，在`Tensor`中输入值为 2.0 和 3.0 的`Double`。开始时，这看起来有点令人不知所措，但这就是 Java API 目前的情况。在 Python 这样的语言中，这些结构要简洁得多。

### 4.2.定义占位符

虽然我们需要为常量提供值，但是**占位符在定义时**不需要值。当图表在会话中运行时，需要提供占位符的值。我们将在教程的后面讨论这一部分。

现在，让我们看看如何定义占位符:

```java
Operation x = graph.opBuilder("Placeholder", "x")
  .setAttr("dtype", DataType.fromClass(Double.class))
  .build();			
Operation y = graph.opBuilder("Placeholder", "y")
  .setAttr("dtype", DataType.fromClass(Double.class))
  .build();
```

请注意，我们不需要为占位符提供任何值。运行时，这些值将作为`Tensors`输入。

### 4.3.定义函数

最后，我们需要定义方程的数学运算，即乘法和加法来得到结果。

在 TensorFlow 中，这些也不过是`Operation`而已，而`Graph.opBuilder()`再次变得很方便:

```java
Operation ax = graph.opBuilder("Mul", "ax")
  .addInput(a.output(0))
  .addInput(x.output(0))
  .build();			
Operation by = graph.opBuilder("Mul", "by")
  .addInput(b.output(0))
  .addInput(y.output(0))
  .build();
Operation z = graph.opBuilder("Add", "z")
  .addInput(ax.output(0))
  .addInput(by.output(0))
  .build();
```

这里，我们定义了 there `Operation`，两个用于乘以我们的输入，最后一个用于累加中间结果。请注意，这里的运算接收的张量只不过是我们先前运算的输出。

请注意，我们正在使用索引“0”从`Operation`获取输出`Tensor`。正如我们之前讨论的，**一个`Operation`可以产生一个或多个`Tensor`** ，因此在检索它的句柄时，我们需要提到索引。因为我们知道我们的操作只返回一个`Tensor`，‘0’就可以了！

## 5.可视化图表

随着图表变大，很难在图表上保留一个标签。这使得以某种方式将它形象化变得很重要。我们总是可以创建一个像我们之前创建的小图一样的手绘，但是对于大图来说不实用。 **TensorFlow 提供了一个名为 TensorBoard 的实用程序来实现这个**。

不幸的是，Java API 不能生成 TensorBoard 使用的事件文件。但是使用 Python 中的 API，我们可以生成如下事件文件:

```java
writer = tf.summary.FileWriter('.')
......
writer.add_graph(tf.get_default_graph())
writer.flush()
```

如果这在 Java 的上下文中没有意义，请不要介意，这只是为了完整性而添加的，对继续本教程的剩余部分没有必要。

我们现在可以在 TensorBoard 中加载并可视化事件文件，如下所示:

```java
tensorboard --logdir .
```

[![mul](img/4d3b3e11f5002f77e8630de3f6a75dcf.png)](/web/20220626083438/https://www.baeldung.com/wp-content/uploads/2019/03/Screenshot-2019-03-25-at-16.55.39.png)

TensorBoard 是 TensorFlow 安装的一部分。

注意这和之前手动绘制的图形的相似性！

## 6.使用会话

我们现在已经在 TensorFlow Java API 中为我们的简单方程创建了一个计算图。但是我们如何运行它呢？在解决这个问题之前，让我们看看我们刚刚创建的`Graph`的状态是什么。如果我们试图打印我们最后的`Operation`“z”的输出:

```java
System.out.println(z.output(0));
```

这将导致类似以下的结果:

```java
<Add 'z:0' shape=<unknown> dtype=DOUBLE>
```

这不是我们所期望的！但是如果我们回想一下我们之前讨论的内容，这实际上是有意义的。**我们刚刚定义的`Graph`还没有运行，所以其中的张量实际上没有任何实际值。**上面的输出只是说这将是一个`Double`类型的`Tensor`。

现在让我们定义一个`Session` 来运行我们的`Graph`:

```java
Session sess = new Session(graph)
```

最后，我们现在准备运行我们的图表，并获得我们一直期待的输出:

```java
Tensor<Double> tensor = sess.runner().fetch("z")
  .feed("x", Tensor.<Double>create(3.0, Double.class))
  .feed("y", Tensor.<Double>create(6.0, Double.class))
  .run().get(0).expect(Double.class);
System.out.println(tensor.doubleValue());
```

那我们在这里做什么？这应该是相当直观的:

*   从`Session`中获取一个`Runner`
*   通过名称“z”定义要提取的`Operation`
*   为占位符“x”和“y”输入张量
*   运行`Session`中的`Graph`

现在我们看到标量输出:

```java
21.0
```

这是我们所期望的，不是吗！

## 7.Java API 的用例

在这一点上，TensorFlow 对于执行基本操作来说可能听起来有些过了。但是，当然， **TensorFlow 意味着运行比这个大得多的图。**

此外，**它在现实世界模型中处理的张量在大小和等级上要大得多**。这些是 TensorFlow 真正使用的实际机器学习模型。

不难看出，随着图的大小增加，在 TensorFlow 中使用核心 API 会变得非常麻烦。为此， **TensorFlow 提供了像 [Keras](https://web.archive.org/web/20220626083438/https://www.tensorflow.org/guide/keras) 这样的高级 API 来处理复杂的模型**。不幸的是，目前还没有官方对 Java 上的 Keras 的支持。

然而，我们可以**使用 Python 来定义和训练复杂的模型**，要么直接在 TensorFlow 中，要么使用 Keras 之类的高级 API。随后，我们可以**导出一个经过训练的模型，并使用 TensorFlow Java API 在 Java** 中使用它。

现在，我们为什么要做这样的事情呢？这对于我们希望在运行 Java 的现有客户端中使用机器学习功能的情况特别有用。例如，为 Android 设备上的用户图片推荐标题。然而，有几个例子，我们对机器学习模型的输出感兴趣，但不一定想用 Java 创建和训练这个模型。

这就是 TensorFlow Java API 的主要用途。我们将在下一节讨论如何实现这一点。

## 8.使用保存的模型

我们现在将了解如何将 TensorFlow 中的模型保存到文件系统中，并可能以完全不同的语言和平台加载回来。 **TensorFlow 提供 API，以一种称为[协议缓冲区](https://web.archive.org/web/20220626083438/https://developers.google.com/protocol-buffers/)的语言和平台中性结构生成模型文件。**

### 8.1.将模型保存到文件系统

我们将首先定义我们之前在 Python 中创建的相同图形，并将其保存到文件系统中。

让我们看看我们可以在 Python 中做到这一点:

```java
import tensorflow as tf
graph = tf.Graph()
builder = tf.saved_model.builder.SavedModelBuilder('./model')
with graph.as_default():
  a = tf.constant(2, name='a')
  b = tf.constant(3, name='b')
  x = tf.placeholder(tf.int32, name='x')
  y = tf.placeholder(tf.int32, name='y')
  z = tf.math.add(a*x, b*y, name='z')
  sess = tf.Session()
  sess.run(z, feed_dict = {x: 2, y: 3})
  builder.add_meta_graph_and_variables(sess, [tf.saved_model.tag_constants.SERVING])
  builder.save()
```

作为本教程在 Java 中的重点，除了它生成了一个名为“saved_model.pb”的文件之外，我们不太关注 Python 中这段代码的细节。顺便提一下，与 Java 相比，在定义类似的图形时一定要注意简洁！

### 8.2.从文件系统加载模型

我们现在将“saved_model.pb”加载到 Java 中。Java TensorFlow API 有`SavedModelBundle`可以处理保存的模型:

```java
SavedModelBundle model = SavedModelBundle.load("./model", "serve");	
Tensor<Integer> tensor = model.session().runner().fetch("z")
  .feed("x", Tensor.<Integer>create(3, Integer.class))
  .feed("y", Tensor.<Integer>create(3, Integer.class))
  .run().get(0).expect(Integer.class);	
System.out.println(tensor.intValue());
```

现在应该很直观地理解上面的代码在做什么了。它只是从协议缓冲区加载模型图，并使其中的会话可用。从那以后，我们几乎可以对这个图做任何事情，就像我们对局部定义的图所做的一样。

## 9.结论

综上所述，在本教程中，我们浏览了与张量流计算图相关的基本概念。我们看到了如何使用 TensorFlow Java API 来创建和运行这样一个图表。然后，我们讨论了与 TensorFlow 相关的 Java API 的用例。

在此过程中，我们还了解了如何使用 TensorBoard 可视化图形，以及如何使用协议缓冲区保存和重新加载模型。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626083438/https://github.com/eugenp/tutorials/tree/master/tensorflow-java)