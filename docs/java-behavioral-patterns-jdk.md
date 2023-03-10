# 核心 Java 中的行为模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-behavioral-patterns-jdk>

## 1。简介

最近，我们研究了[创造性设计模式](/web/20220627080214/https://www.baeldung.com/java-creational-design-patterns)以及在 JVM 和其他核心库中何处可以找到它们。现在我们来看看[行为设计模式](/web/20220627080214/https://www.baeldung.com/design-patterns-series)。**这些关注的是我们的物体如何相互作用，或者我们如何与它们相互作用。**

## 2.责任链

[责任链](/web/20220627080214/https://www.baeldung.com/chain-of-responsibility-pattern)模式允许对象实现一个公共接口，并允许每个实现在适当的时候委托给下一个实现。**这允许我们构建一个实现链，其中每个实现在调用链中下一个元素之前或之后执行一些动作**:

```java
interface ChainOfResponsibility {
    void perform();
}
```

```java
class LoggingChain {
    private ChainOfResponsibility delegate;

    public void perform() {
        System.out.println("Starting chain");
        delegate.perform();
        System.out.println("Ending chain");
    }
}
```

在这里，我们可以看到一个示例，其中我们的实现在委托调用之前和之后打印输出。

我们不需要拜访代表。我们可以决定不这样做，而是提前终止链。例如，如果有一些输入参数，我们可以验证它们，如果它们无效，我们可以提前终止。

### 2.1.JVM 中的例子

[Servlet 过滤器](https://web.archive.org/web/20220627080214/https://www.oracle.com/java/technologies/filters.html)是 JEE 生态系统中以这种方式工作的一个例子。一个实例接收 servlet 请求和响应，一个`FilterChain`实例代表整个过滤器链。**然后每个都应该执行它的工作，然后要么终止链，要么调用`chain.doFilter()`将控制传递给下一个过滤器**:

```java
public class AuthenticatingFilter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
      throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        if (!"MyAuthToken".equals(httpRequest.getHeader("X-Auth-Token")) {
             return;
        }
        chain.doFilter(request, response);
    }
}
```

## 3.命令

[命令](/web/20220627080214/https://www.baeldung.com/java-command-pattern)模式允许我们将一些具体的行为或者命令封装在一个公共接口之后，这样它们就可以在运行时被正确触发。

通常我们会有一个命令接口、一个接收命令实例的接收者实例和一个负责调用正确命令实例的调用者。**然后，我们可以定义命令接口的不同实例，以在接收器上执行不同的操作**:

```java
interface DoorCommand {
    perform(Door door);
}
```

```java
class OpenDoorCommand implements DoorCommand {
    public void perform(Door door) {
        door.setState("open");
    }
}
```

这里，我们有一个命令实现，它将把一个`Door`作为接收者，并使门变得“打开”。然后，当我们的调用者希望打开一扇给定的门时，可以调用这个命令，这个命令封装了如何做到这一点。

将来，我们可能需要改变我们的`OpenDoorCommand`来检查门是否先被锁上。**这种改变将完全在命令之内，接收者和调用者类不需要有任何改变。**

### 3.1.JVM 中的例子

这种模式的一个非常常见的例子是 Swing 中的`Action`类:

```java
Action saveAction = new SaveAction();
button = new JButton(saveAction)
```

这里，`SaveAction`是命令，使用这个类的 Swing `JButton` 组件是调用者，`Action`实现是用一个`ActionEvent`作为接收者来调用的。

## 4.迭代程序

迭代器模式允许我们处理集合中的元素，并依次与每个元素交互。我们用它来编写函数，在一些元素上采用任意迭代器，而不考虑它们来自哪里。源可以是有序列表、无序集合或无限流:

```java
void printAll<T>(Iterator<T> iter) {
    while (iter.hasNext()) {
        System.out.println(iter.next());
    }
}
```

### 4.1.JVM 中的例子

**所有的 JVM 标准集合都通过公开一个对集合中的元素返回一个`Iterator<T>`的`iterator()`方法**来实现[迭代器模式](/web/20220627080214/https://www.baeldung.com/java-iterator)。流也实现相同的方法，除了在这种情况下，它可能是一个无限流，所以迭代器可能永远不会终止。

## 5.纪念品

**[Memento](/web/20220627080214/https://www.baeldung.com/java-memento-design-pattern)模式允许我们编写能够改变状态的对象，然后恢复到它们之前的状态。**本质上是对象状态的“撤销”功能。

这可以通过在任何时候调用 setter 时存储以前的状态来相对容易地实现:

```java
class Undoable {
    private String value;
    private String previous;

    public void setValue(String newValue) {
        this.previous = this.value;
        this.value = newValue;
    }

    public void restoreState() {
        if (this.previous != null) {
            this.value = this.previous;
            this.previous = null;
        }
    }
}
```

这提供了撤消对对象所做的最后一次更改的能力。

这通常是通过将整个对象状态包装在一个称为 Memento 的对象中来实现的。这允许在单个操作中保存和恢复整个状态，而不是必须单独保存每个字段。

### 5.1.JVM 中的例子

**[JavaServer Faces](/web/20220627080214/https://www.baeldung.com/spring-jsf) 提供了一个名为`StateHolder`的接口，允许实现者保存和恢复他们的状态**。有几个标准组件实现了这一点，包括单个组件——例如，`HtmlInputFile`、`HtmlInputText`或`HtmlSelectManyCheckbox`——以及复合组件，例如`HtmlForm`。

## 6.观察者

[观察者](/web/20220627080214/https://www.baeldung.com/java-observer-pattern)模式允许一个对象向其他人表明已经发生了变化。通常我们会有一个主体——发出事件的对象，以及一系列观察者——接收这些事件的对象。观察者将向受试者登记他们希望被告知的变化。一旦发生这种情况，**受试者身上发生的任何变化都会通知观察者**:

```java
class Observable {
    private String state;
    private Set<Consumer<String>> listeners = new HashSet<>;

    public void addListener(Consumer<String> listener) {
        this.listeners.add(listener);
    }

    public void setState(String newState) {
        this.state = state;
        for (Consumer<String> listener : listeners) {
            listener.accept(newState);
        }
    }
}
```

这需要一组事件侦听器，并在每次状态随着新的状态值改变时调用每个事件侦听器。

### 6.1.JVM 中的例子

**Java 有一对标准的类允许我们这样做——`java.beans.PropertyChangeSupport`和`java.beans.PropertyChangeListener`。**

充当一个类，可以添加和删除观察者，并且可以通知他们所有的状态变化。`PropertyChangeListener`是一个接口，我们的代码可以实现它来接收已经发生的任何变化:

```java
PropertyChangeSupport observable = new PropertyChangeSupport();

// Add some observers to be notified when the value changes
observable.addPropertyChangeListener(evt -> System.out.println("Value changed: " + evt));

// Indicate that the value has changed and notify observers of the new value
observable.firePropertyChange("field", "old value", "new value");
```

请注意，还有另外一对似乎更合适的类——`java.util.Observer`和`java.util.Observable`。但是在 Java 9 中不推荐使用这些，因为它们不够灵活和不可靠。

## 7.战略

[策略](/web/20220627080214/https://www.baeldung.com/java-strategy-pattern)模式允许我们编写通用代码，然后将特定的策略插入其中，为我们的具体情况提供所需的特定行为。

这通常通过一个表示策略的接口来实现。然后，客户端代码能够根据具体情况的需要编写实现该接口的具体类。例如，我们可能有一个系统，在这个系统中，我们需要通知最终用户，并将通知机制实现为可插入的策略:

```java
interface NotificationStrategy {
    void notify(User user, Message message);
}
```

```java
class EmailNotificationStrategy implements NotificationStrategy {
    ....
}
```

```java
class SMSNotificationStrategy implements NotificationStrategy {
    ....
}
```

然后，我们可以在运行时决定实际使用这些策略中的哪一个来将消息发送给这个用户。我们还可以编写新的策略来使用，同时对系统的其余部分产生最小的影响。

### 7.1.JVM 中的例子

**标准的 Java 库广泛使用这种模式，通常以起初看起来不明显的方式使用**。例如，Java 8 中引入的[流 API](/web/20220627080214/https://www.baeldung.com/java-streams) 大量使用了这种模式。提供给`map()`、`filter()`和其他方法的 lambdas 都是提供给泛型方法的可插拔策略。

不过，例子可以追溯到更早。Java 1.2 中引入的`Comparator`接口是一种策略，可以根据需要对集合中的元素进行排序。我们可以提供不同的`Comparator`实例，根据需要以不同的方式对相同的列表进行排序:

```java
// Sort by name
Collections.sort(users, new UsersNameComparator());

// Sort by ID
Collections.sort(users, new UsersIdComparator());
```

## 8.模板方法

当我们想要编排几个不同的方法一起工作时，使用[模板方法](/web/20220627080214/https://www.baeldung.com/java-template-method-pattern)模式。**我们将使用模板方法和一组一个或多个抽象方法**定义一个基类——要么未实现，要么通过一些默认行为实现。**模板方法然后以固定的模式调用这些抽象方法。**我们的代码实现了这个类的一个子类，并根据需要实现了这些抽象方法:

```java
class Component {
    public void render() {
        doRender();
        addEventListeners();
        syncData();
    }

    protected abstract void doRender();

    protected void addEventListeners() {}

    protected void syncData() {}
}
```

这里，我们有一些任意的 UI 组件。我们的子类将实现`doRender()`方法来实际呈现组件。我们也可以选择实现`addEventListeners()`和`syncData()`方法。当我们的 UI 框架呈现这个组件时，它将保证这三个组件以正确的顺序被调用。

### 8.1.JVM 中的例子

**Java 集合使用的`AbstractList`、`AbstractSet,`和`AbstractMap`有很多这种模式的例子。**例如，`indexOf()`和`lastIndexOf()`方法都是根据`listIterator()`方法工作的，它有一个默认的实现，但是在一些子类中被覆盖。同样，`add(T)` 和`addAll(int, T)` 方法都是根据`add(int, T)`方法工作的，后者没有默认实现，需要由子类实现。

**Java IO 在`InputStream`、`OutputStream`、`Reader,`和`Writer`中也利用了这种模式**。例如，`InputStream`类有几个根据`read(byte[], int, int)`工作的方法，需要子类来实现。

## 9.访问者

**[访问者](/web/20220627080214/https://www.baeldung.com/java-visitor-pattern)模式允许我们的代码以一种类型安全的方式处理各种子类，而不需要求助于`instanceof`检查。**我们将有一个 visitor 接口，为我们需要支持的每个具体子类提供一个方法。我们的基类将会有一个`accept(Visitor)`方法。每个子类都将调用这个 visitor 上的适当方法，并将自己传入。这允许我们在每个方法中实现具体的行为，每个方法都知道它将与具体的类型一起工作:

```java
interface UserVisitor<T> {
    T visitStandardUser(StandardUser user);
    T visitAdminUser(AdminUser user);
    T visitSuperuser(Superuser user);
}
```

```java
class StandardUser {
    public <T> T accept(UserVisitor<T> visitor) {
        return visitor.visitStandardUser(this);
    }
}
```

这里我们有我们的`UserVisitor`接口，上面有三个不同的访问者方法。我们的例子`StandardUser`调用适当的方法，同样的事情将在`AdminUser`和`Superuser`中完成。然后，我们可以根据需要让我们的访问者使用这些功能:

```java
class AuthenticatingVisitor {
    public Boolean visitStandardUser(StandardUser user) {
        return false;
    }
    public Boolean visitAdminUser(AdminUser user) {
        return user.hasPermission("write");
    }
    public Boolean visitSuperuser(Superuser user) {
        return true;
    }
}
```

我们的`StandardUser`从来没有权限，我们的`Superuser`总是有权限，我们的`AdminUser`可能有权限，但这需要在用户本身中查找。

### 9.1.JVM 中的例子

Java NIO2 框架用 [`Files.walkFileTree()`](/web/20220627080214/https://www.baeldung.com/java-nio2-file-visitor) 使用这种模式。这需要一个`FileVisitor`的实现，它有处理遍历文件树的各种不同方面的方法。**我们的代码可以用它来搜索文件，打印出匹配的文件，处理目录中的许多文件，或者许多其他需要在目录中工作的事情**:

```java
Files.walkFileTree(startingDir, new SimpleFileVisitor() {
    public FileVisitResult visitFile(Path file, BasicFileAttributes attr) {
        System.out.println("Found file: " + file);
    }

    public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) {
        System.out.println("Found directory: " + dir);
    }
});
```

## 10.结论

在本文中，我们已经了解了用于对象行为的各种设计模式。我们还看了核心 JVM 中使用的这些模式的例子，因此我们可以看到许多应用程序已经从中受益。