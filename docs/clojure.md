# Clojure 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/clojure>

## 1。简介

Clojure 是一种完全在 Java 虚拟机上运行的函数式编程语言，与 Scala 和 Kotlin 的方式类似。 **Clojure 被认为是 Lisp 的衍生物**，任何有过其他 Lisp 语言经验的人都会很熟悉。

本教程介绍了 Clojure 语言，介绍了如何开始使用它以及它如何工作的一些关键概念。

## 2。安装 Clojure

Clojure 可以作为安装程序和方便的脚本在 Linux 和 macOS 上使用。遗憾的是，现阶段 Windows 还没有这样的安装程序。

然而，Linux 脚本可能在 Cygwin 或 Windows Bash 中工作。还有一个在线服务可以用来测试语言，旧版本有一个独立的版本可以使用。

### 2.1。独立下载

独立的 JAR 文件可以从 [Maven Central](https://web.archive.org/web/20220630012534/https://repo1.maven.org/maven2/org/clojure/clojure/1.8.0/clojure-1.8.0.jar) 下载。不幸的是，由于 JAR 文件被分割成了更小的模块，比 1.8.0 版本新的版本不再容易以这种方式工作。

一旦下载了这个 JAR 文件，我们就可以把它当作一个可执行的 JAR 来使用，就像一个交互式 REPL 一样:

```java
$ java -jar clojure-1.8.0.jar
Clojure 1.8.0
user=>
```

### 2.2。REPL 的网络接口

https://repl.it/languages/clojure 的 Clojure REPL 网站提供了一个网络界面，我们无需下载任何东西就可以尝试。目前，它只支持 Clojure 1.8.0，不支持更新的版本。

### 2.3。MacOS 上的安装程序

如果您使用 macOS 并安装了 Homebrew，那么可以很容易地安装最新版本的 Clojure:

```java
$ brew install clojure
```

在撰写本文时，这将支持最新版本的 clo jure–1 . 10 . 0。安装完成后，我们可以简单地通过使用`clojure`或`clj`命令来加载 REPL:

```java
$ clj
Clojure 1.10.0
user=>
```

### 2.4。Linux 上的安装程序

我们可以使用自安装 shell 脚本在 Linux 上安装工具:

```java
$ curl -O https://download.clojure.org/install/linux-install-1.10.0.411.sh
$ chmod +x linux-install-1.10.0.411.sh
$ sudo ./linux-install-1.10.0.411.sh
```

与 macOS 安装程序一样，这些将在 Clojure 的最新版本中可用，并且可以使用`clojure`或`clj`命令来执行。

## 3。Clojure REPL 简介

以上所有选项都让我们可以访问 Clojure REPL。这是 Java 9 和更高版本的 JShell 工具的直接 Clojure 等价物，允许我们直接输入 Clojure 代码并立即看到结果。**这是一种试验和发现某些语言功能如何工作的奇妙方式。**

一旦加载了 REPL，我们将看到一个提示，可以输入并立即执行任何标准的 Clojure 代码。这包括简单的 Clojure 构造，以及与其他 Java 库的交互——尽管它们需要在要加载的类路径中可用。

REPL 的提示是我们正在工作的当前名称空间的指示。对于我们的大部分工作，这是`user`名称空间，因此提示将是:

```java
user=>
```

本文剩余部分的所有内容都将假设我们可以访问 Clojure REPL，并且可以直接在任何这样的工具中工作。

## 4。语言基础知识

Clojure 语言看起来与许多其他基于 JVM 的语言非常不同，一开始可能看起来很不寻常。它被认为是 [Lisp](https://web.archive.org/web/20220630012534/https://en.wikipedia.org/wiki/Lisp_(programming_language)) 的一种方言，与其他 Lisp 语言有着非常相似的语法和功能。

与其他 Lisp 方言一样，我们用 Clojure 编写的许多代码都是以列表的形式表达的。然后可以对列表进行评估以产生结果——以更多列表或简单值的形式。

例如:

```java
(+ 1 2) ; = 3
```

这是一个包含三个元素的列表。“+”符号表示我们正在执行此呼叫添加。然后，剩余的元素将用于该调用。因此，这等于“1 + 2”。

通过在这里使用列表语法，这可以被简单地扩展。例如，我们可以这样做:

```java
(+ 1 2 3 4 5) ; = 15
```

而这个求值为“1 + 2 + 3 + 4 + 5”。

还要注意分号字符。这在 Clojure 中用来表示一个注释，而不是我们在 Java 中看到的表达式的结尾。

### 4.1。简单的**类型**

Clojure 构建在 JVM 之上，因此我们可以像其他 Java 应用程序一样访问相同的标准类型。类型通常是自动推断的，不需要显式指定。

例如:

```java
123 ; Long
1.23 ; Double
"Hello" ; String
true ; Boolean
```

我们还可以使用特殊的前缀或后缀来指定一些更复杂的类型:

```java
42N ; clojure.lang.BigInt
3.14159M ; java.math.BigDecimal
1/3 ; clojure.lang.Ratio
#"[A-Za-z]+" ; java.util.regex.Pattern
```

注意，使用的是`clojure.lang.BigInt`类型，而不是`java.math.BigInteger`。这是因为 Clojure 类型有一些小的优化和修复。

### 4.2。关键词和符号

Clojure 给了我们关键字和符号的概念。关键字仅指它们本身，通常用于映射键之类的东西。另一方面，符号是用来指代其他事物的名称。例如，变量定义和函数名都是符号。

我们可以使用以冒号为前缀的名称来构造关键字:

```java
user=> :kw
:kw
user=> :a
:a
```

**关键字与其自身直接相等，而不与其他任何事物相等:**

```java
user=> (= :a :a)
true
user=> (= :a :b)
false
user=> (= :a "a")
false
```

Clojure 中大多数其他不是简单值的东西都被认为是符号。**它们评估它们所引用的任何东西**，而一个关键字总是评估它自己:

```java
user=> (def a 1)
#'user/a
user=> :a
:a
user=> a
1
```

### 4.3。名称空间

Clojure 语言有组织代码的名称空间的概念。我们编写的每一段代码都存在于一个名称空间中。

默认情况下，REPL 在`user`名称空间中运行——如提示“user= >所示。

**我们可以使用`ns`关键字**来创建和更改名称空间:

```java
user=> (ns new.ns)
nil
new.ns=>
```

一旦我们更改了名称空间，旧名称空间中定义的任何内容都不再可用，而新名称空间中定义的任何内容现在都可用。

**我们可以通过完全限定名称空间来访问它们的定义**。例如，名称空间`clojure.string`定义了一个函数`upper-case`。

如果我们在`clojure.string`名称空间中，我们可以直接访问它。如果不是，那么我们需要将其限定为`clojure.string/upper-case`:

```java
user=> (clojure.string/upper-case "hello")
"HELLO"
user=> (upper-case "hello") ; This is not visible in the "user" namespace
Syntax error compiling at (REPL:1:1).
Unable to resolve symbol: upper-case in this context
user=> (ns clojure.string)
nil
clojure.string=> (upper-case "hello") ; This is visible because we're now in the "clojure.string" namespace
"HELLO"
```

**我们还可以使用`require`** 关键字**以更简单的方式从另一个名称空间访问定义**。我们可以通过两种主要方式来使用它——用一个较短的名称定义一个名称空间以便于使用，以及从另一个没有任何前缀的名称空间直接访问定义:

```java
clojure.string=> (require '[clojure.string :as str])
nil
clojure.string=> (str/upper-case "Hello")
"HELLO"

user=> (require '[clojure.string :as str :refer [upper-case]])
nil
user=> (upper-case "Hello")
"HELLO"
```

这两个都只影响当前的名称空间，所以改变到一个不同的名称空间将需要有新的`requires. `这有助于保持我们的名称空间更干净，让我们只访问我们需要的。

### 4.4。变量

一旦我们知道如何定义简单的值，我们就可以将它们赋给变量。我们可以使用关键字`def`来实现:

```java
user=> (def a 123)
#'user/a
```

**一旦我们完成了这些，我们就可以在任何我们想要的地方使用符号`a`** **来表示这个值:**

```java
user=> a
123
```

变量定义可以简单，也可以复杂。

例如，要将一个变量定义为数字之和，我们可以这样做:

```java
user=> (def b (+ 1 2 3 4 5))
#'user/b
user=> b
15
```

注意，我们从来不需要声明变量或者指出它是什么类型。Clojure 自动为我们决定了这一切。

如果我们试图使用一个尚未定义的变量，那么我们将得到一个错误:

```java
user=> unknown
Syntax error compiling at (REPL:0:0).
Unable to resolve symbol: unknown in this context
user=> (def c (+ 1 unknown))
Syntax error compiling at (REPL:1:8).
Unable to resolve symbol: unknown in this context
```

请注意，`def`函数的输出看起来与输入略有不同。定义一个变量`a`会返回一串`‘user/a`。这是因为结果是一个符号，并且这个符号是在当前命名空间中定义的。

### 4.5。功能

我们已经看到了一些如何在 Clojure 中调用函数的例子。我们创建一个列表，从要调用的函数开始，然后是所有的参数。

当这个列表求值时，我们从函数中获得返回值。例如:

```java
user=> (java.time.Instant/now)
#object[java.time.Instant 0x4b6690c0 "2019-01-15T07:54:01.516Z"]
user=> (java.time.Instant/parse "2019-01-15T07:55:00Z")
#object[java.time.Instant 0x6b8d96d9 "2019-01-15T07:55:00Z"]
user=> (java.time.OffsetDateTime/of 2019 01 15 7 56 0 0 java.time.ZoneOffset/UTC)
#object[java.time.OffsetDateTime 0xf80945f "2019-01-15T07:56Z"]
```

我们还可以嵌套函数调用，因为当我们希望将一个函数调用的输出作为参数传递给另一个函数调用时:

```java
user=> (java.time.OffsetDateTime/of 2018 01 15 7 57 0 0 (java.time.ZoneOffset/ofHours -5))
#object[java.time.OffsetDateTime 0x1cdc4c27 "2018-01-15T07:57-05:00"]
```

同样，**如果我们愿意，我们也可以定义我们的函数**。**使用`fn`命令**创建功能:

```java
user=> (fn [a b]
  (println "Adding numbers" a "and" b)
  (+ a b)
)
#object[user$eval165$fn__166 0x5644dc81 "[[email protected]](/web/20220630012534/https://www.baeldung.com/cdn-cgi/l/email-protection)"]
```

不幸的是，**这并没有给这个函数一个可以使用的名字**。相反，我们可以使用`def, `定义一个符号来表示这个函数，就像我们看到的变量一样:

```java
user=> (def add
  (fn [a b]
    (println "Adding numbers" a "and" b)
    (+ a b)
  )
)
#'user/add
```

既然我们已经定义了这个函数，我们可以像调用任何其他函数一样调用它:

```java
user=> (add 1 2)
Adding numbers 1 and 2
3
```

为了方便起见， **Clojure 还允许我们使用`defn`在一个单独的 go** 中定义一个带有名称的函数。

例如:

```java
user=> (defn sub [a b]
  (println "Subtracting" b "from" a)
  (- a b)
)
#'user/sub
user=> (sub 5 2)
Subtracting 2 from 5
3
```

### 4.6。Let 和局部变量

**`def`调用定义了一个当前名称空间**的全局符号。这通常不是执行代码时所希望的。相反， **Clojure 提供了`let`调用来定义块**的局部变量。这在函数内部使用时特别有用，因为您不希望变量泄漏到函数之外。

例如，我们可以定义我们的子函数:

```java
user=> (defn sub [a b]
  (def result (- a b))
  (println "Result: " result)
  result
)
#'user/sub
```

但是，使用这种方法会产生以下意想不到的副作用:

```java
user=> (sub 1 2)
Result:  -1
-1
user=> result ; Still visible outside of the function
-1
```

相反，让我们用`let`重写它:

```java
user=> (defn sub [a b]
  (let [result (- a b)]
    (println "Result: " result)
    result
  )
)
#'user/sub
user=> (sub 1 2)
Result:  -1
-1
user=> result
Syntax error compiling at (REPL:0:0).
Unable to resolve symbol: result in this context
```

这一次,`result`符号在函数外部是不可见的。或者，实际上，在使用它的`let`块之外。

## 5。收藏

到目前为止，我们主要是与简单的值进行交互。我们也看到了列表，但仅此而已。Clojure 确实有一整套可以使用的集合，包括列表、向量、地图和集合:

*   向量是值的有序列表——任何任意值都可以放入向量，包括其他集合。
*   集合是一个无序的值的集合，并且不能包含相同的值超过一次。
*   映射是一组简单的键/值对。使用关键字作为 map 中的键是非常常见的，但是我们可以使用任何我们喜欢的值，包括其他集合。
*   列表非常类似于向量。区别类似于 Java 中的一个`ArrayList`和一个`LinkedList`。通常，向量是首选，但是如果我们想在开始处添加元素，或者如果我们只想按顺序访问元素，那么列表会更好。

### 5.1。构建集合

可以使用简写符号或使用函数调用来创建每一个:

```java
; Vector
user=> [1 2 3]
[1 2 3]
user=> (vector 1 2 3)
[1 2 3]

; List
user=> '(1 2 3)
(1 2 3)
user=> (list 1 2 3)
(1 2 3)

; Set
user=> #{1 2 3}
#{1 3 2}
user=> (hash-set 1 2 3)
#{1 3 2}

; Map
user=> {:a 1 :b 2}
{:a 1, :b 2}
user=> (hash-map :a 1 :b 2)
{:b 2, :a 1}
```

请注意，`Set`和`Map`示例返回值的顺序不同。这是因为这些集合本质上是无序的，我们看到的内容取决于它们在内存中的表示方式。

我们还可以看到，创建列表的语法与表达式的标准 Clojure 语法非常相似。事实上，Clojure 表达式是一个被求值的列表，而这里的撇号表示我们想要实际的值列表，而不是对它求值。

当然，我们可以像分配任何其他值一样，将集合分配给变量。我们也可以使用一个集合作为另一个集合中的键或值。

列表被认为是一个`seq`。这意味着该类实现了`ISeq`接口。所有其他集合都可以使用`seq`函数转换为`seq`:

```java
user=> (seq [1 2 3])
(1 2 3)
user=> (seq #{1 2 3})
(1 3 2)
user=> (seq {:a 1 2 3})
([:a 1] [2 3])
```

### 5.2。访问收藏

一旦我们有了一个集合，我们就可以与它交互来再次获得值。我们如何做到这一点稍微取决于所讨论的集合，因为它们每个都有不同的语义。

向量是唯一允许我们通过索引获得任意值的集合。这是通过将向量和索引计算为表达式来实现的:

```java
user=> (my-vector 2) ; [1 2 3]
3
```

**我们可以使用相同的语法对地图进行同样的操作**:

```java
user=> (my-map :b)
2
```

**我们也有访问向量和列表的函数，以获得列表的第一个值、最后一个值和剩余值:**

```java
user=> (first my-vector)
1
user=> (last my-list)
3
user=> (next my-vector)
(2 3)
```

**映射有额外的函数来获取键和值的完整列表:**

```java
user=> (keys my-map)
(:a :b)
user=> (vals my-map)
(1 2)
```

我们对集合唯一真正的访问是查看一个特定的元素是否是一个成员。

这看起来非常类似于访问任何其他集合:

```java
user=> (my-set 1)
1
user=> (my-set 5)
nil
```

### 5.3。识别收藏

我们已经看到，我们访问集合的方式根据我们拥有的集合类型而变化。我们有一组函数可以用来确定这一点，既可以用特定的方式，也可以用更通用的方式。

我们的每个集合都有一个特定的函数来确定给定值是否属于该类型——`list?`表示列表,`set?`表示集合，等等。此外，还有`seq?`用于确定给定值是否是任何类型的`seq`，以及`associative?`用于确定给定值是否允许任何类型的关联访问——这意味着向量和映射:

```java
user=> (vector? [1 2 3]) ; A vector is a vector
true
user=> (vector? #{1 2 3}) ; A set is not a vector
false
user=> (list? '(1 2 3)) ; A list is a list
true
user=> (list? [1 2 3]) ; A vector is not a list
false
user=> (map? {:a 1 :b 2}) ; A map is a map
true
user=> (map? #{1 2 3}) ; A set is not a map
false
user=> (seq? '(1 2 3)) ; A list is a seq
true
user=> (seq? [1 2 3]) ; A vector is not a seq
false
user=> (seq? (seq [1 2 3])) ; A vector can be converted into a seq
true
user=> (associative? {:a 1 :b 2}) ; A map is associative
true
user=> (associative? [1 2 3]) ; A vector is associative
true
user=> (associative? '(1 2 3)) ; A list is not associative
false
```

### 5.4。变异集合

在 Clojure 中，和大多数函数式语言一样，所有的集合都是不可变的。我们对集合所做的任何更改都会导致创建一个全新的集合来表示这些更改。这可以带来巨大的效率优势，并意味着没有意外副作用的风险。

然而，我们还必须小心地理解这一点，否则我们集合的预期变化将不会发生。

**使用`conj`** 向向量、列表或集合添加新元素。这在每种情况下都有不同的工作方式，但基本意图是相同的:

```java
user=> (conj [1 2 3] 4) ; Adds to the end
[1 2 3 4]
user=> (conj '(1 2 3) 4) ; Adds to the beginning
(4 1 2 3)
user=> (conj #{1 2 3} 4) ; Unordered
#{1 4 3 2}
user=> (conj #{1 2 3} 3) ; Adding an already present entry does nothing
#{1 3 2}
```

**我们也可以使用`disj`** 从集合中删除条目。请注意，这不适用于列表或向量，因为它们是严格排序的:

```java
user=> (disj #{1 2 3} 2) ; Removes the entry
#{1 3}
user=> (disj #{1 2 3} 4) ; Does nothing because the entry wasn't present
#{1 3 2}
```

**使用`assoc`向地图添加新元素。我们也可以使用`dissoc:`** 从地图中删除条目

```java
user=> (assoc {:a 1 :b 2} :c 3) ; Adds a new key
{:a 1, :b 2, :c 3}
user=> (assoc {:a 1 :b 2} :b 3) ; Updates an existing key
{:a 1, :b 3}
user=> (dissoc {:a 1 :b 2} :b) ; Removes an existing key
{:a 1}
user=> (dissoc {:a 1 :b 2} :c) ; Does nothing because the key wasn't present
{:a 1, :b 2}
```

### 5.5。函数式编程构造

Clojure 本质上是一种函数式编程语言。这意味着**我们可以使用许多传统的函数式编程概念——比如** `map, filter,` **和** `**reduce****. **` **这些通常与其他语言中的功能相同**。不过，确切的语法可能略有不同。

具体来说，这些函数通常以要应用的函数作为第一个参数，以要应用该函数的集合作为第二个参数:

```java
user=> (map inc [1 2 3]) ; Increment every value in the vector
(2 3 4)
user=> (map inc #{1 2 3}) ; Increment every value in the set
(2 4 3)

user=> (filter odd? [1 2 3 4 5]) ; Only return odd values
(1 3 5)
user=> (remove odd? [1 2 3 4 5]) ; Only return non-odd values
(2 4)

user=> (reduce + [1 2 3 4 5]) ; Add all of the values together, returning the sum
15
```

## 6。控制结构

与所有通用语言一样，Clojure 特性需要标准的控制结构，比如条件和循环。

### 6.1。条件句

**条件句由`if`语句**处理。这需要三个参数:一个测试，如果测试是`true`就执行一个块，如果测试是`false`就执行一个块。其中的每一个都可以是一个简单的值，也可以是一个将根据需要进行评估的标准列表:

```java
user=> (if true 1 2)
1
user=> (if false 1 2)
2
```

我们的测试可以是我们需要的任何东西——它不必是一个`true/false`值。它也可以是一个块，通过计算得到我们需要的值:

```java
user=> (if (> 1 2) "True" "False")
"False"
```

这里可以使用所有的标准检查，包括`=, >,`和`<`。还有一组谓词可用于各种其他原因——我们在查看集合时已经看到了一些，例如:

```java
user=> (if (odd? 1) "1 is odd" "1 is even")
"1 is odd"
```

测试可以返回任何值——它不需要仅仅是`true`或`false`。但是，如果值是除了`false`或`nil`之外的任何值，则被认为是`true`。这与 JavaScript 的工作方式不同，在 JavaScript 中，有一大组值被认为是“真-y”而不是`true`:

```java
user=> (if 0 "True" "False")
"True"
user=> (if [] "True" "False")
"True"
user=> (if nil "True" "False")
"False"
```

### 6.2。循环

我们对集合的功能支持处理了大部分循环工作——我们使用标准函数，让语言为我们做迭代，而不是在集合上写循环。

**除此之外，循环完全使用递归**来完成。我们可以编写递归函数，或者我们可以使用`loop `和`recur `关键字来编写递归风格的循环:

```java
user=> (loop [accum [] i 0]
  (if (= i 10)
    accum
    (recur (conj accum i) (inc i))
  ))
[0 1 2 3 4 5 6 7 8 9]
```

`loop`调用启动一个在每次迭代中执行的内部块，并通过设置一些初始参数开始。然后，`recur`调用回调到循环中，提供用于迭代的下一个参数。如果`recur`没有被调用，那么循环结束。

在这种情况下，每当`i `值不等于 10 时，我们就进行循环，然后一旦它等于 10，我们就返回数字的累积向量。

## 7。总结

本文介绍了 Clojure 编程语言，并展示了该语法的工作原理以及您可以用它做的一些事情。这只是一个介绍性的水平，并没有深入到使用该语言可以完成的所有事情。

然而，为什么不把它捡起来，试一试，看看你能用它做些什么。