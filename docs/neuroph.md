# 欧米诺简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/neuroph>

## 1。简介

本文看一下[Neuroph](https://web.archive.org/web/20220701012801/http://neuroph.sourceforge.net/)——一个用于创建神经网络和利用机器学习的开源库。

在这篇文章中，我们看看核心概念和几个例子，如何把它们放在一起。

## 2。欧米诺

我们可以通过以下方式与欧米诺互动:

*   基于图形用户界面的工具
*   Java 库

这两种方法都依赖于底层的类层次结构，该层次结构在`neurons`层之外构建人工神经网络。

我们将重点放在编程方面，但将参考 Neuroph 基于 GUI 的方法中的几个共享类来帮助澄清我们正在做的事情。

关于基于 GUI 方法的更多信息，请看一下欧米诺文档。

### 2.1。依赖性

如果为了使用 Neuroph，我们需要添加下面的 Maven 条目:

```java
<dependency>
    <groupId>org.beykery</groupId>
    <artifactId>neuroph</artifactId>
    <version>2.92</version>
</dependency>
```

最新版本可以在 Maven Central 上找到[。](https://web.archive.org/web/20220701012801/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22neuroph%22)

## 3。主要类别和概念

所有使用的基本概念构建块都有相应的 Java 类。

`Neurons`连接到`Layers`上，然后被组合成`NeuralNetworks`。`NeuralNetworks`随后使用`LearningRules`和`DataSets`进行训练。

### 3.1。`Neuron`

`Neuron`类有四个主要属性:

1.  `inputConnection:`之间的加权连接`Neurons`
2.  `inputFunction:` 指定应用于输入连接数据的`weights`和 `vector sums`
3.  `transferFunction:` 指定应用于输出数据的`weights`和 `vector sums`
4.  `output:` 将`transferFunctions`和`inputFunctions`应用于`inputConnection`产生的输出值

这四个主要属性共同决定了行为:

```java
output = transferFunction(inputFunction(inputConnections));
```

### 3.2。`Layer`

**`Layers`本质上是`Neurons`** 的分组，使得`Layer`中的每一个`Neuro` `n`(通常)只与前一个和后一个`Layers`中的`Neurons`相连。

因此，`Layers`通过存在于它们的`Neurons`上的加权函数在它们之间传递信息。

`Neurons`可以添加到图层:

```java
Layer layer = new Layer(); 
layer.addNeuron(n);
```

### 3.3。`NeuralNetwork`

顶级超类 `NeuralNetwork`被细分为几种常见的人工神经网络，包括卷积神经网络(子类`ConvolutionalNetwork`)、Hopfield 神经网络(子类`Hopfield`)和多层感知器神经网络(子类`MultilayerPerceptron`)。

**所有的 `NeuralNetworks`都是由`Layers`和**组成的，它们通常被组织成三分法:

1.  输入层
2.  隐藏层
3.  输出图层

如果我们使用的是`NeuralNetwork`(比如`Perceptron`)的子类的构造函数，我们可以使用这个简单的方法传递`Layer`、每个`Layer`的`Neuron`的数量以及它们的索引:

```java
NeuralNetwork ann = new Perceptron(2, 4, 1);
```

有时我们会想要手动地做这件事(并且很好地看到在引擎盖下面发生了什么)。将`Layer`添加到`NeuralNetwork`的基本操作是这样完成的:

```java
NeuralNetwork ann = new NeuralNetwork();   
Layer layer = new Layer();
ann.addLayer(0, layer);
ann.setInputNeurons(layer.getNeurons()); 
```

第一个参数指定了`NeuralNetwork`中`Layer`的索引；第二个参数指定了`Layer`本身。`Layers`手动添加应使用`ConnectionFactory`类连接:

```java
ann.addLayer(0, inputLayer);    
ann.addLayer(1, hiddenLayerOne); 
ConnectionFactory.fullConnect(ann.getLayerAt(0), ann.getLayerAt(1));
```

第一个和最后一个`Layer`也应该连接:

```java
ConnectionFactory.fullConnect(ann.getLayerAt(0), 
  ann.getLayerAt(ann.getLayersCount() - 1), false);
ann.setOutputNeurons(ann.getLayerAt(
  ann.getLayersCount() - 1).getNeurons());
```

请记住，`NeuralNetwork`的力量很大程度上取决于:

1.  `NeuralNetwork`中`Layers`的数量
2.  每个`Layer`中`Neurons`的数量(以及它们之间的`weighted functions`)，以及
3.  训练算法的有效性/准确性`DataSet`

### 3.4。`NeuralNetwork`训练我们的

使用`DataSet`和`LearningRule`类训练 **`NeuralNetworks` 。**

`DataSet`用于表示和提供要学习的信息或用于训练`NeuralNetwork`的信息。`DataSets` 的特点是他们的 `input size, outputsize,`和成排的 `(DataSetRow).`

```java
int inputSize = 2; 
int outputSize = 1; 
DataSet ds = new DataSet(inputSize, outputSize);

DataSetRow rOne 
  = new DataSetRow(new double[] {0, 0}, new double[] {0});
ds.addRow(rOne);
DataSetRow rTwo 
  = new DataSetRow(new double[] {1, 1}, new double[] {0});
ds.addRow(rTwo);
```

`LearningRule`指定`DataSet`被`NeuralNetwork`教导或训练的方式。`LearningRule`的子类包括`BackPropagation` 和`SupervisedLearning`。

```java
NeuralNetwork ann = new NeuralNetwork();
//...
BackPropagation backPropagation = new BackPropagation();
backPropagation.setMaxIterations(1000);
ann.learn(ds, backPropagation);
```

## 4。将所有这些放在一起

现在，让我们将这些构件组合成一个真实的例子。我们将从**开始，将几个层组合成熟悉的 `input layer`、 `hidden layer`和`output layer`模式**，这是大多数神经网络架构的范例。

### 4.1。层

我们将通过组合四层来组装我们的`NeuralNetwork`。我们的目标是建立一个(2，4，4，1) `NeuralNetwork.`

让我们首先定义我们的输入层:

```java
Layer inputLayer = new Layer();
inputLayer.addNeuron(new Neuron());
inputLayer.addNeuron(new Neuron());
```

接下来，我们实现隐藏层一:

```java
Layer hiddenLayerOne = new Layer();
hiddenLayerOne.addNeuron(new Neuron());
hiddenLayerOne.addNeuron(new Neuron());
hiddenLayerOne.addNeuron(new Neuron());
hiddenLayerOne.addNeuron(new Neuron());
```

隐藏的第二层:

```java
Layer hiddenLayerTwo = new Layer(); 
hiddenLayerTwo.addNeuron(new Neuron()); 
hiddenLayerTwo.addNeuron(new Neuron()); 
hiddenLayerTwo.addNeuron(new Neuron()); 
hiddenLayerTwo.addNeuron(new Neuron());
```

最后，我们定义输出层:

```java
Layer outputLayer = new Layer();
outputLayer.addNeuron(new Neuron()); 
```

### 4.2。`NeuralNetwork`

接下来，我们可以把它们组合成一个`NeuralNetwork`:

```java
NeuralNetwork ann = new NeuralNetwork();
ann.addLayer(0, inputLayer);
ann.addLayer(1, hiddenLayerOne);
ConnectionFactory.fullConnect(ann.getLayerAt(0), ann.getLayerAt(1));
ann.addLayer(2, hiddenLayerTwo);
ConnectionFactory.fullConnect(ann.getLayerAt(1), ann.getLayerAt(2));
ann.addLayer(3, outputLayer);
ConnectionFactory.fullConnect(ann.getLayerAt(2), ann.getLayerAt(3));
ConnectionFactory.fullConnect(ann.getLayerAt(0), 
  ann.getLayerAt(ann.getLayersCount()-1), false);
ann.setInputNeurons(inputLayer.getNeurons());
ann.setOutputNeurons(outputLayer.getNeurons());
```

### 4.3。培训

出于训练的目的，让我们通过指定输入和结果输出向量的大小来组合一个`DataSet` :

```java
int inputSize = 2;
int outputSize = 1;
DataSet ds = new DataSet(inputSize, outputSize);
```

我们向我们的`DataSet`添加一个基本行，遵守上面定义的输入和输出约束——我们在本例中的目标是教会我们的网络进行基本的 XOR(异或)运算:

```java
DataSetRow rOne
  = new DataSetRow(new double[] {0, 1}, new double[] {1});
ds.addRow(rOne);
DataSetRow rTwo
  = new DataSetRow(new double[] {1, 1}, new double[] {0});
ds.addRow(rTwo);
DataSetRow rThree 
  = new DataSetRow(new double[] {0, 0}, new double[] {0});
ds.addRow(rThree);
DataSetRow rFour
  = new DataSetRow(new double[] {1, 0}, new double[] {1});
ds.addRow(rFour);
```

接下来，让我们用内置的`BackPropogation LearningRule`来训练我们的`NeuralNetwork`:

```java
BackPropagation backPropagation = new BackPropagation();
backPropagation.setMaxIterations(1000);
ann.learn(ds, backPropagation); 
```

### 4.4。测试

现在我们的`NeuralNetwork`已经训练好了，让我们来测试一下。对于作为`DataSetRow`传递到我们的`DataSet`中的每一对逻辑值，我们运行以下类型的测试:

```java
ann.setInput(0, 1);
ann.calculate();
double[] networkOutputOne = ann.getOutput(); 
```

需要记住的重要一点是 **`NeuralNetworks` 只输出 0 和 1** 包含区间上的一个值。为了输出一些其他的值，我们必须`normalize`和`denormalize`我们的数据。

在这种情况下，对于逻辑运算来说，0 和 1 非常适合这项工作。输出将是:

```java
Testing: 1, 0 Expected: 1.0 Result: 1.0
Testing: 0, 1 Expected: 1.0 Result: 1.0
Testing: 1, 1 Expected: 0.0 Result: 0.0
Testing: 0, 0 Expected: 0.0 Result: 0.0 
```

我们看到我们的`NeuralNetwork`成功预测了正确答案！

## 5。结论

我们刚刚回顾了 Neuroph 使用的基本概念和类。

关于这个库的更多信息可以在[这里](https://web.archive.org/web/20220701012801/http://neuroph.sourceforge.net/)找到，本文中使用的代码示例可以在 GitHub 上的[找到。](https://web.archive.org/web/20220701012801/https://github.com/eugenp/tutorials/tree/master/libraries)