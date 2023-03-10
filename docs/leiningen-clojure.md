# Clojure 的 Leiningen 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/leiningen-clojure>

## 1。简介

Leiningen 是我们 Clojure 项目的现代构建系统。它也是完全用 Clojure 编写和配置的。

它的工作方式类似于 Maven，为我们提供了描述项目的声明性配置，而不需要配置要执行的确切步骤。

让我们来看看如何开始使用 Leiningen 来构建我们的 Clojure 项目。

## 2。安装雷宁根

Leiningen 可以单独下载，也可以从不同系统的大量[包管理器中下载。](https://web.archive.org/web/20221013010210/https://github.com/technomancy/leiningen/wiki/Packaging)

独立下载适用于 Windows 和 T2 的 Linux 和 Mac。在所有情况下，下载文件，必要时使其可执行，然后就可以使用了。

第一次运行该脚本时，它将下载 Leiningen 应用程序的其余部分，然后从现在开始，它将被缓存:

```java
$ ./lein
Downloading Leiningen to /Users/user/.lein/self-installs/leiningen-2.8.3-standalone.jar now...
.....
Leiningen is a tool for working with Clojure projects.

Several tasks are available:
.....

Run `lein help $TASK` for details.

.....
```

## 3。创建新项目

**一旦 Leiningen 安装完毕，我们就可以通过调用`lein new`来使用它创建一个新项目。**

这将使用一组选项中的特定模板创建一个项目:

*   `app`–用于创建应用程序
*   `default`–用于创建一般项目结构，通常用于库
*   `plugin`–用于创建 Leiningen 插件
*   `template`–用于为未来项目创建新的 Leiningen 模板

例如，要创建一个名为“我的项目”的新应用程序，我们将执行:

```java
$ ./lein new app my-project
Generating a project called my-project based on the 'app' template.
```

这为我们提供了一个包含以下内容的项目:

*   构建定义—`project.clj`
*   源目录-`src `-包括初始源文件-`src/my_project/core.clj`
*   测试目录-`test`-包括初始测试文件-`test/my_project/core_test.clj`
*   一些附加文档文件—`README.md, LICENSE, CHANGELOG.md`和`doc/intro.md`

**看看我们的构建定义，我们会发现它告诉我们要构建什么，而不是如何构建:**

```java
(defproject my-project "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "EPL-2.0 OR GPL-2.0-or-later WITH Classpath-exception-2.0"
            :url "https://www.eclipse.org/legal/epl-2.0/"}
  :dependencies [[org.clojure/clojure "1.9.0"]]
  :main ^:skip-aot my-project.core
  :target-path "target/%s"
  :profiles {:uberjar {:aot :all}})
```

这告诉我们:

*   项目的详细信息，包括项目名称、版本、描述、主页和许可证详细信息。
*   执行应用程序时使用的主命名空间
*   依赖项列表
*   将输出构建到的目标路径
*   构建 uberjar 的概要文件

注意，主源名称空间是`my-project.core`，位于文件`my_project/core.clj. `中。Clojure 不鼓励使用单段名称空间——相当于 Java 项目中的顶级类。

此外，文件名是用下划线而不是连字符生成的，因为 JVM 在文件名中使用连字符时存在一些问题。

生成的代码非常简单:

```java
(ns my-project.core
  (:gen-class))

(defn -main
  "I don't do a whole lot ... yet."
  [& args]
  (println "Hello, World!"))
```

另外，注意 Clojure 在这里只是一个依赖项。这使得使用任何版本的 Clojure 库来编写项目变得很简单，尤其是在同一个系统上运行多个不同的版本。

如果我们改变这种依赖关系，那么我们将得到替代版本。

## 4。建造和运行

如果我们不能构建、运行并打包发布，我们的项目就没有多大价值，所以让我们接下来看看这个。

### 4.1。发射 REPL

**一旦我们有了一个项目，我们可以使用`lein repl`** 在里面启动一个 REPL。这将为我们提供一个 REPL，其中包含了项目中已经在类路径上可用的所有内容——包括所有项目文件以及所有依赖项。

它还让我们进入为项目定义的主名称空间:

```java
$ lein repl
nREPL server started on port 62856 on host 127.0.0.1 - nrepl://127.0.0.1:62856
[]REPL-y 0.4.3, nREPL 0.5.3
Clojure 1.9.0
Java HotSpot(TM) 64-Bit Server VM 1.8.0_77-b03

    Docs: (doc function-name-here)
          (find-doc "part-of-name-here")
  Source: (source function-name-here)
 Javadoc: (javadoc java-object-or-class-here)
    Exit: Control+D or (exit) or (quit)
 Results: Stored in vars *1, *2, *3, an exception in *e

my-project.core=> (-main)
Hello, World!
nil
```

这将执行当前名称空间中的函数`-main`，我们在上面已经看到了。

### 4.2。运行应用程序

如果我们正在处理一个应用程序项目——使用`lein new app`创建——那么**我们可以简单地从命令行运行应用程序。这是使用`lein run`** 完成的:

```java
$ lein run
Hello, World!
```

这将在我们的`project.clj`文件中定义为`:main`的名称空间中执行名为`-main`的函数。

### 4.3。建造图书馆

如果我们正在处理一个库项目——使用`lein new default`创建——那么**我们可以将库构建到一个 JAR 文件中，以便包含在其他项目中**。

我们有两种方法可以做到这一点——使用`lein jar`或`lein install`。区别仅仅在于输出 JAR 文件放在哪里。

**如果我们使用`lein jar`，那么它将把它放在本地`target`目录**:

```java
$ lein jar
Created /Users/user/source/me/my-library/target/my-library-0.1.0-SNAPSHOT.jar
```

**如果我们使用`lein install`，那么它将构建 JAR 文件，生成一个`pom.xml`文件，然后将这两个文件放入本地 Maven 资源库**(通常在用户主目录的`.m2/repository`下)

```java
$ lein install
Created /Users/user/source/me/my-library/target/my-library-0.1.0-SNAPSHOT.jar
Wrote /Users/user/source/me/my-library/pom.xml
Installed jar and pom into local repo.
```

### 4.4。构建 Uberjar

如果我们正在进行一个应用项目， **Leiningen 给了我们构建一个叫做 uberjar** 的东西的能力。这是一个 JAR 文件，包含项目本身和所有的依赖项，并允许它按原样运行。

```java
$ lein uberjar
Compiling my-project.core
Created /Users/user/source/me/my-project/target/uberjar/my-project-0.1.0-SNAPSHOT.jar
Created /Users/user/source/me/my-project/target/uberjar/my-project-0.1.0-SNAPSHOT-standalone.jar
```

文件`my-project-0.1.0-SNAPSHOT.jar`是一个包含本地项目的 JAR 文件，文件`my-project-0.1.0-SNAPSHOT-standalone.jar`包含运行应用程序所需的一切。

```java
$ java -jar target/uberjar/my-project-0.1.0-SNAPSHOT-standalone.jar
Hello, World!
```

## 5。依赖性

虽然我们可以自己编写项目所需的所有东西，但是重用他人代表我们已经完成的工作通常要好得多。我们可以通过让我们的项目依赖于这些其他库来做到这一点。

### 5.1。向我们的项目添加依赖关系

为了向我们的项目添加依赖项，我们需要将它们正确地添加到我们的`project.clj`文件中。

依赖项被表示为一个向量，由相关依赖项的名称和版本组成。我们已经看到 Clojure 本身是作为一个依赖项添加的，以`[org.clojure/clojure “1.9.0”]`的形式编写。

如果我们想要添加其他依赖项，我们可以通过将它们添加到关键字`:dependencies`旁边的 vector 中来实现。例如，如果我们想要依赖于`clj-json`,我们将更新文件:

```java
 :dependencies [[org.clojure/clojure "1.9.0"] [clj-json "0.5.3"]]
```

**一旦完成，如果我们启动我们的 REPL——或任何其他方式来构建或运行我们的项目——那么 Leiningen 将确保依赖项被下载并在类路径中可用**:

```java
$ lein repl
Retrieving clj-json/clj-json/0.5.3/clj-json-0.5.3.pom from clojars
Retrieving clj-json/clj-json/0.5.3/clj-json-0.5.3.jar from clojars
nREPL server started on port 62146 on host 127.0.0.1 - nrepl://127.0.0.1:62146
REPL-y 0.4.3, nREPL 0.5.3
Clojure 1.9.0
Java HotSpot(TM) 64-Bit Server VM 1.8.0_77-b03
    Docs: (doc function-name-here)
          (find-doc "part-of-name-here")
  Source: (source function-name-here)
 Javadoc: (javadoc java-object-or-class-here)
    Exit: Control+D or (exit) or (quit)
 Results: Stored in vars *1, *2, *3, an exception in *e

my-project.core=> (require '(clj-json [core :as json]))
nil
my-project.core=> (json/generate-string {"foo" "bar"})
"{\"foo\":\"bar\"}"
my-project.core=>
```

我们也可以在项目内部使用它们。例如，我们可以如下更新生成的`src/my_project/core.clj`文件:

```java
(ns my-project.core
  (:gen-class))

(require '(clj-json [core :as json]))

(defn -main
  "I don't do a whole lot ... yet."
  [& args]
  (println (json/generate-string {"foo" "bar"})))
```

然后运行它将完全按照预期进行:

```java
$ lein run
{"foo":"bar"}
```

### 5.2。寻找依赖关系

通常，很难找到我们想要在项目中使用的依赖项。 **Leiningen 自带一个内置于**的搜索功能，让这一切变得更简单。这是使用`lein search`完成的。

例如，我们可以找到我们的 JSON 库:

```java
$ lein search json
Searching central ...
[com.jwebmp/json "0.63.0.60"]
[com.ufoscout.coreutils/json "3.7.4"]
[com.github.iarellano/json "20190129"]
.....
Searching clojars ...
[cheshire "5.8.1"]
  JSON and JSON SMILE encoding, fast.
[json-html "0.4.4"]
  Provide JSON and get a DOM node with a human representation of that JSON
[ring/ring-json "0.5.0-beta1"]
  Ring middleware for handling JSON
[clj-json "0.5.3"]
  Fast JSON encoding and decoding for Clojure via the Jackson library.
.....
```

**这将搜索我们的项目与**合作的所有存储库——在本例中，是 Maven Central 和 [Clojars](https://web.archive.org/web/20221013010210/https://clojars.org/) 。然后，它返回要放入我们的`project.clj`文件的确切字符串，以及库的描述(如果可用的话)。

## 6。测试我们的项目

Clojure 内置了对应用程序单元测试的支持，Leiningen 可以在我们的项目中利用这一点。

我们生成的项目包含位于`test`目录中的测试代码，以及位于`src`目录中的源代码。它还包括一个单一的，失败的默认测试——在`test/my_project/core-test.clj`发现:

```java
(ns my-project.core-test
  (:require [clojure.test :refer :all]
            [my-project.core :refer :all]))

(deftest a-test
  (testing "FIXME, I fail."
    (is (= 0 1))))
```

这将从我们的项目中导入`my-project.core`名称空间，从核心 Clojure 语言中导入`clojure.test`名称空间。然后我们用`deftest`和`testing `调用定义一个测试。

我们可以立即看到测试的名称，以及它被故意写为失败的事实——它断言`0 == 1`。

**让我们使用`lein test`命令**来运行它，并立即看到测试运行和失败:

```java
$ lein test
lein test my-project.core-test

lein test :only my-project.core-test/a-test

FAIL in (a-test) (core_test.clj:7)
FIXME, I fail.
expected: (= 0 1)
  actual: (not (= 0 1))

Ran 1 tests containing 1 assertions.
1 failures, 0 errors.
Tests failed.
```

如果我们改为修复测试，改为断言`1 == 1`，那么我们将得到一个传递消息:

```java
$ lein test
lein test my-project.core-test

Ran 1 tests containing 1 assertions.
0 failures, 0 errors.
```

这是一个更加简洁的输出，只显示了我们需要知道的内容。这意味着**当有失败时，他们会立即站出来。**

如果我们愿意，我们还可以运行特定的测试子集。命令行允许提供一个名称空间，并且只执行该名称空间中的测试:

```java
$ lein test my-project.core-test

lein test my-project.core-test

Ran 1 tests containing 1 assertions.
0 failures, 0 errors.

$ lein test my-project.unknown

lein test my-project.unknown

Ran 0 tests containing 0 assertions.
0 failures, 0 errors.
```

## 7。总结

本文展示了如何开始使用 Leiningen 构建工具，以及如何使用它来管理基于 Clojure 的项目——包括可执行应用程序和共享库。

为什么不在下一个项目中尝试一下，看看效果如何。