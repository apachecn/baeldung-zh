# 用 Java 中的有限自动机验证输入

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-finite-automata>

## 1.概观

如果你学过 CS，你无疑上过关于编译器或类似的课程；在这些课程中，教授有限自动机(也称为有限状态机)的概念。这是一种形式化语言语法规则的方式。

你可以在这里和这里阅读更多关于主题[的内容。](https://web.archive.org/web/20220525005723/https://www.cs.rochester.edu/u/nelson/courses/csc_173/fa/fa.html)

那么这个被遗忘的概念如何对我们这些不需要为构建新的编译器而烦恼的高级程序员有所帮助呢？

事实证明，这个概念可以简化很多业务场景，并为我们提供推理复杂逻辑的工具。

举个简单的例子，我们也可以在没有外部第三方库的情况下验证输入。

## 2.该算法

简而言之，这样的机器声明状态和从一个状态到另一个状态的方式。如果让一个流通过它，可以用下面的算法(伪代码)验证它的格式:

```java
for (char c in input) {
    if (automaton.accepts(c)) {
        automaton.switchState(c);
        input.pop(c);
    } else {
        break;
    }
}
if (automaton.canStop() && input.isEmpty()) {
    print("Valid");
} else {
    print("Invalid");
}
```

我们说自动机“接受”给定的字符，如果有任何箭头从当前状态出发，上面有字符。切换状态意味着指针被跟随，当前状态被箭头指向的状态所取代。

最后，当循环结束时，我们检查自动机是否“可以停止”(当前状态是双圆圈的)并且输入已经耗尽。

## 3.一个例子

让我们为一个 JSON 对象编写一个简单的验证器，来看看运行中的算法。这是接受对象的自动机:

[![](img/87043bceedd3e80a17a69d9c890040e4.png)](/web/20220525005723/https://www.baeldung.com/wp-content/uploads/2017/03/json_dfa-2.png)

请注意，value 可以是以下值之一:string、integer、boolean、null 或其他 JSON 对象。为了简洁起见，在我们的例子中，我们只考虑字符串。

### 3.1。代码

实现一个有限的[状态机](/web/20220525005723/https://www.baeldung.com/cs/state-machines)非常简单。我们有以下内容:

```java
public interface FiniteStateMachine {
    FiniteStateMachine switchState(CharSequence c);
    boolean canStop();
}

interface State {
    State with(Transition tr);
    State transit(CharSequence c);
    boolean isFinal();
}

interface Transition {
    boolean isPossible(CharSequence c);
    State state();
} 
```

它们之间的关系是:

*   状态机有一个 current `State`并告诉我们它是否可以停止(状态是否是最终的)
*   一个`State`有一个可以遵循的转换列表(向外的箭头)
*   一个`Transition`告诉我们字符是否被接受，并给我们下一个`State`

```java
publi class RtFiniteStateMachine implements FiniteStateMachine {

    private State current;

    public RtFiniteStateMachine(State initial) {
        this.current = initial;
    }

    public FiniteStateMachine switchState(CharSequence c) {
        return new RtFiniteStateMachine(this.current.transit(c));
    }

    public boolean canStop() {
        return this.current.isFinal();
    }
}
```

注意， **`FiniteStateMachine`的实现是不可变的**。这主要是因为它的单个实例可以多次使用。

接下来，我们有了实现`RtState`。为了流畅起见，`with(Transition)`方法在添加过渡后返回实例。一个`State`也告诉我们它是否是最终的(双圆圈)。

```java
public class RtState implements State {

    private List<Transition> transitions;
    private boolean isFinal;

    public RtState() {
        this(false);
    }

    public RtState(boolean isFinal) {
        this.transitions = new ArrayList<>();
        this.isFinal = isFinal;
    }

    public State transit(CharSequence c) {
        return transitions
          .stream()
          .filter(t -> t.isPossible(c))
          .map(Transition::state)
          .findAny()
          .orElseThrow(() -> new IllegalArgumentException("Input not accepted: " + c));
    }

    public boolean isFinal() {
        return this.isFinal;
    }

    @Override
    public State with(Transition tr) {
        this.transitions.add(tr);
        return this;
    }
}
```

最后，`RtTransition`检查转换规则并给出下一个`State`:

```java
public class RtTransition implements Transition {

    private String rule;
    private State next;
    public State state() {
        return this.next;
    }

    public boolean isPossible(CharSequence c) {
        return this.rule.equalsIgnoreCase(String.valueOf(c));
    }

    // standard constructors
}
```

上面的代码是[这里的](https://web.archive.org/web/20220525005723/https://github.com/eugenp/tutorials/tree/master/algorithms-miscellaneous-1/src/main/java/com/baeldung/algorithms/automata)。有了这个实现，您应该能够构建任何状态机。开头描述的算法非常简单:

```java
String json = "{\"key\":\"value\"}";
FiniteStateMachine machine = this.buildJsonStateMachine();
for (int i = 0; i < json.length(); i++) {
    machine = machine.switchState(String.valueOf(json.charAt(i)));
}

assertTrue(machine.canStop());
```

检查测试类 [RtFiniteStateMachineTest](https://web.archive.org/web/20220525005723/https://github.com/eugenp/tutorials/blob/ef7400484c2409ae25a82874e16a1b8d53ba2b32/algorithms/src/test/java/algorithms/RtFiniteStateMachineLongRunningUnitTest.java) 以查看`buildJsonStateMachine()`方法。请注意，它比上图多添加了几个状态，以便正确捕捉字符串周围的引号。

## 4.结论

有限自动机是很好的工具，可以用来验证结构化数据。

然而，它们并不广为人知，因为当涉及复杂输入时它们会变得复杂(因为一个过渡只能用于一个字符)。然而，在检查一组简单的规则时，它们非常有用。

最后，如果您想使用有限状态机完成一些更复杂的工作，那么 [StatefulJ](https://web.archive.org/web/20220525005723/https://github.com/statefulj/statefulj) 和 [squirrel](https://web.archive.org/web/20220525005723/https://github.com/hekailiang/squirrel) 是两个值得研究的库。

你可以在 GitHub 上查看代码样本[。](https://web.archive.org/web/20220525005723/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-1)