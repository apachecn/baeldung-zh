# Nashorn 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nashorn>

## 1。简介

本文关注的是从 Java 8 开始 JVM 的新的默认 JavaScript 引擎。

许多复杂的技术被用来使`Nashorn`比它的前身`Rhino,` 性能更高，所以这是一个值得的改变。

让我们来看看它的一些使用方法。

## 2。命令行

JDK 1.8 包括一个名为`jjs` 的命令行解释器，可以用来运行 JavaScript 文件，或者，如果启动时没有参数，可以作为一个 REPL(交互式 shell):

```java
$ $JAVA_HOME/bin/jjs hello.js
Hello World
```

这里文件`hello.js` 包含一条指令:`print(“Hello World”);`

相同的代码可以以交互方式运行:

```java
$ $JAVA_HOME/bin/jjs
jjs> print("Hello World")
Hello World
```

您还可以通过添加一个`#!$JAVA_HOME/bin/jjs` 作为第一行来指示*nix 运行时使用`jjs` 来运行目标脚本:

```java
#!$JAVA_HOME/bin/jjs
var greeting = "Hello World";
print(greeting);
```

然后该文件可以正常运行:

```java
$ ./hello.js
Hello World
```

## 3。嵌入式脚本引擎

从 JVM 内部运行 JavaScript 的第二种可能也是更常见的方式是通过`ScriptEngine.` JSR-223 定义了一组脚本 API，允许可插入的脚本引擎架构，可用于任何动态语言(当然，前提是它有 JVM 实现)。

让我们创建一个 JavaScript 引擎:

```java
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");

Object result = engine.eval(
   "var greeting='hello world';" +
   "print(greeting);" +
   "greeting");
```

这里我们创建一个新的`ScriptEngineManager` ，并立即要求它给我们一个名为`nashorn`的`ScriptEngine` 。然后，我们传递几个指令并获得结果，正如所料，结果是一个`String`“`hello world`”。

## 4。向脚本传递数据

通过定义一个`Bindings` 对象并将其作为第二个参数传递给`eval` 函数，可以将数据传递给引擎:

```java
Bindings bindings = engine.createBindings();
bindings.put("count", 3);
bindings.put("name", "baeldung");

String script = "var greeting='Hello ';" +
  "for(var i=count;i>0;i--) { " +
  "greeting+=name + ' '" +
  "}" +
  "greeting";

Object bindingsResult = engine.eval(script, bindings);
```

运行这个代码片段会产生:“`Hello baeldung baeldung baeldung`”。

## 5。调用 JavaScript 函数

当然，可以从 Java 代码中调用 JavaScript 函数:

```java
engine.eval("function composeGreeting(name) {" +
  "return 'Hello ' + name" +
  "}");
Invocable invocable = (Invocable) engine;

Object funcResult = invocable.invokeFunction("composeGreeting", "baeldung");
```

这将返回“`Hello baeldung`”。

## 6。使用 Java 对象

因为我们运行在 JVM 中，所以可以在 JavaScript 代码中使用本地 Java 对象。

这是通过使用一个`Java` 对象来实现的:

```java
Object map = engine.eval("var HashMap = Java.type('java.util.HashMap');" +
  "var map = new HashMap();" +
  "map.put('hello', 'world');" +
  "map");
```

## 7。语言扩展

**`Nashorn` 的目标是 ECMAScript 5.1** ，但它确实提供了一些扩展，使 JavaScript 的使用更好一些。

### 7.1。用 For-Each 迭代集合

`For-each`是一个方便的扩展，使各种集合的迭代更加容易:

```java
String script = "var list = [1, 2, 3, 4, 5];" +
  "var result = '';" +
  "for each (var i in list) {" +
  "result+=i+'-';" +
  "};" +
  "print(result);";

engine.eval(script);
```

这里，我们通过使用`for-each` 迭代构造来连接数组的元素。

结果输出将是`1-2-3-4-5-`。

### 7.2。函数文字

在简单的函数声明中，可以省略花括号:

```java
function increment(in) ++in
```

显然，这只能用于简单的单行函数。

### 7.3。条件捕获子句

可以添加仅在指定条件为真时才执行的保护性 catch 子句:

```java
try {
    throw "BOOM";
} catch(e if typeof e === 'string') {
    print("String thrown: " + e);
} catch(e) {
    print("this shouldn't happen!");
}
```

这将打印“`String thrown: BOOM`”。

### 7.4。类型化数组和类型转换

可以使用 Java 类型化数组，也可以在 JavaScript 数组之间进行转换:

```java
function arrays(arr) {
    var javaIntArray = Java.to(arr, "int[]");
    print(javaIntArray[0]);
    print(javaIntArray[1]);
    print(javaIntArray[2]);
}
```

`Nashorn` 在这里执行一些类型转换，以确保来自动态类型 JavaScript 数组的所有值都适合纯整数的 Java 数组。

用参数 `[100, “1654”, true]` 调用上述函数的结果是输出 100、1654 和 1(都是数字)。

`String`和布尔值被隐式转换成它们的逻辑整数对应物。

### 7.5。用`Object.setPrototypeOf` 设置对象的原型

`Nashorn` 定义一个 API 扩展，使我们能够改变对象的原型:

```java
Object.setPrototypeOf(obj, newProto)
```

这个函数通常被认为是比`Object.prototype.__proto__`更好的选择，所以它应该是在所有新代码中设置对象原型的首选方式。

### 7.6。神奇的`__noSuchProperty__`和`__noSuchMethod__`

可以在一个对象上定义方法，每当访问一个`undefined`属性或调用一个`undefined`方法时，这些方法都会被调用:

```java
var demo = {
    __noSuchProperty__: function (propName) {
        print("Accessed non-existing property: " + propName);
    },

    __noSuchMethod__: function (methodName) {
        print("Invoked non-existing method: " + methodName);
    }
};

demo.doesNotExist;
demo.callNonExistingMethod()
```

这将打印:

```java
Accessed non-existing property: doesNotExist
Invoked non-existing method: callNonExistingMethod
```

### 7.7。用`Object.bindProperties` 绑定对象属性

`Object.bindProperties`可用于将一个对象的属性绑定到另一个对象:

```java
var first = {
    name: "Whiskey",
    age: 5
};

var second = {
    volume: 100
};

Object.bindProperties(first, second);

print(first.volume);

second.volume = 1000;
print(first.volume);
```

请注意，这创建的是一个“活动”绑定，对源对象的任何更新也可以通过绑定目标看到。

### 7.8。地点

当前文件名、目录和行可以从全局变量`__FILE__, __DIR__, __LINE__:`中获得

```java
print(__FILE__, __LINE__, __DIR__)
```

### 7.9。String.prototype 的扩展

`Nashorn` 在`String`原型上提供了两个简单但非常有用的扩展。这些是`trimRight` 和`trimLeft` 函数，不出所料，它们返回删除了空格的`String`的副本:

```java
print("   hello world".trimLeft());
print("hello world     ".trimRight());
```

将打印两次“hello world ”,不带前导或尾随空格。

### 7.10。`Java.asJSONCompatible`功能

使用这个函数，我们可以获得一个与 Java JSON 库期望兼容的对象。

也就是说，如果它本身或者通过它可传递到达的任何对象是一个 JavaScript 数组，那么这样的对象将被公开为`JSObject`，它还实现了用于公开数组元素的`List`接口。

```java
Object obj = engine.eval("Java.asJSONCompatible(
  { number: 42, greet: 'hello', primes: [2,3,5,7,11,13] })");
Map<String, Object> map = (Map<String, Object>)obj;

System.out.println(map.get("greet"));
System.out.println(map.get("primes"));
System.out.println(List.class.isAssignableFrom(map.get("primes").getClass()));
```

这将打印“`hello`”，接着是`[2, 3, 5, 7, 11, 13]`，接着是`true.`

## 8.加载脚本

也可以从`ScriptEngine`中加载另一个 JavaScript 文件:

```java
load('classpath:script.js')
```

也可以从 URL 加载脚本:

```java
load('/script.js')
```

请记住，JavaScript 没有名称空间的概念，所以所有东西都堆在全局范围内。这使得加载的脚本可能与您的代码或彼此之间产生命名冲突。这可以通过使用`loadWithNewGlobal` 功能来缓解:

```java
var math = loadWithNewGlobal('classpath:math_module.js')
math.increment(5);
```

用下面的`math_module.js`:

```java
var math = {
    increment: function(num) {
        return ++num;
    }
};

math;bai
```

这里我们定义了一个名为`math`的对象，它有一个名为`increment.` 的函数。使用这个范例，我们甚至可以模拟基本的模块化！

## 8。结论

本文探讨了`Nashorn J` avaScript 引擎的一些特性。这里展示的例子使用了字符串脚本，但是对于现实生活中的场景，您很可能希望将脚本保存在单独的文件中，并使用`Reader` 类加载它们。

和往常一样，这篇文章中的代码都可以在 GitHub 的[上找到。](https://web.archive.org/web/20220808071643/https://github.com/eugenp/tutorials/tree/master/language-interop)