# 用 Java 枚举实现简单状态机

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-enum-simple-state-machine>

## 1。概述

在本教程中，我们将看看[状态机](/web/20221207040444/https://www.baeldung.com/cs/state-machines)以及如何使用枚举在 Java 中实现它们。

我们还将解释与为每个状态使用一个接口和一个具体的类相比，这种实现的优势。

## 2。Java 枚举

Java 枚举是一种特殊类型的类，它定义了一个常量列表。这允许**类型安全的实现和更可读的代码**。

例如，假设我们有一个人力资源软件系统，可以批准员工提交的休假请求。该请求由团队领导审核，并上报给部门经理。部门经理是负责批准请求的人。

保存休假请求状态的最简单的枚举是:

```java
public enum LeaveRequestState {
    Submitted,
    Escalated,
    Approved
}
```

我们可以参考这个枚举的常量:

```java
LeaveRequestState state = LeaveRequestState.Submitted;
```

枚举也可以包含方法。我们可以在一个 enum 中写一个抽象方法，它会强制每个 enum 实例实现这个方法。这对于状态机的实现非常重要，我们将在下面看到。

由于 Java 枚举隐式扩展了类`java.lang.Enum`，它们不能扩展另一个类。然而，它们可以实现一个接口，就像任何其他类一样。

下面是一个包含抽象方法的枚举示例:

```java
public enum LeaveRequestState {
    Submitted {
        @Override
        public String responsiblePerson() {
            return "Employee";
        }
    },
    Escalated {
        @Override
        public String responsiblePerson() {
            return "Team Leader";
        }
    },
    Approved {
        @Override
        public String responsiblePerson() {
            return "Department Manager";
        }
    };

    public abstract String responsiblePerson();
}
```

请注意最后一个枚举常量末尾分号的用法。当常量后面有一个或多个方法时，分号是必需的。

在这种情况下，我们用一个`responsiblePerson()`方法扩展了第一个例子。这告诉我们负责执行每个动作的人。所以，如果我们试图检查负责`Escalated`状态的人，它会给我们“组长”:

```java
LeaveRequestState state = LeaveRequestState.Escalated;
assertEquals("Team Leader", state.responsiblePerson());
```

同样，如果我们检查谁负责批准请求，它将为我们提供“部门经理”:

```java
LeaveRequestState state = LeaveRequestState.Approved;
assertEquals("Department Manager", state.responsiblePerson());
```

## 3。状态机

状态机——也称为有限状态机或有限自动机——是一种用于构建抽象机器的计算模型。这些机器在给定时间只能处于一种状态。每种状态都是系统的一种状态，变化到另一种状态。这些状态变化被称为转换。

有了图表和符号，数学会变得复杂，但对我们程序员来说事情就简单多了。

[状态模式](/web/20221207040444/https://www.baeldung.com/java-state-design-pattern)是众所周知的 GoF 二十三种设计模式之一。这个模式借用了数学中模型的概念。**它允许一个对象根据其状态封装同一对象的不同行为。我们可以对状态之间的转换进行编程，然后定义单独的状态。**

为了更好地解释这个概念，我们将扩展休假请求示例来实现一个状态机。

## 4。枚举为状态机

我们将关注 Java 中状态机的 enum 实现。其他实现也是可能的，我们将在下一节比较它们。

使用 enum 实现状态机的要点是**我们不必处理显式设置状态**。相反，我们可以提供如何从一个状态转换到下一个状态的逻辑。让我们开始吧:

```java
public enum LeaveRequestState {

    Submitted {
        @Override
        public LeaveRequestState nextState() {
            return Escalated;
        }

        @Override
        public String responsiblePerson() {
            return "Employee";
        }
    },
    Escalated {
        @Override
        public LeaveRequestState nextState() {
            return Approved;
        }

        @Override
        public String responsiblePerson() {
            return "Team Leader";
        }
    },
    Approved {
        @Override
        public LeaveRequestState nextState() {
            return this;
        }

        @Override
        public String responsiblePerson() {
            return "Department Manager";
        }
    };

    public abstract LeaveRequestState nextState(); 
    public abstract String responsiblePerson();
}
```

在这个例子中，**状态机的转换是使用 enum 的抽象方法**实现的。更准确地说，在每个枚举常量上使用`nextState()`,我们指定到下一个状态的转换。如果需要，我们还可以实现一个`previousState()`方法。

下面是一个测试来检查我们的实现:

```java
LeaveRequestState state = LeaveRequestState.Submitted;

state = state.nextState();
assertEquals(LeaveRequestState.Escalated, state);

state = state.nextState();
assertEquals(LeaveRequestState.Approved, state);

state = state.nextState();
assertEquals(LeaveRequestState.Approved, state);
```

我们在`Submitted`初始状态启动休假请求。然后，我们通过使用上面实现的 `nextState()`方法来验证状态转换。

请注意，**因为`Approved`是最终状态，所以不会发生其他转换**。

## 5。用 Java 枚举实现状态机的优势

带有接口和实现类的状态机的[实现可能需要开发和维护大量代码。](/web/20221207040444/https://www.baeldung.com/java-state-design-pattern)

因为 Java enum 最简单的形式是一个常量列表，所以我们可以使用 enum 来定义我们的状态。由于枚举也可以包含行为，我们可以使用方法来提供状态之间的转换实现。

将所有的逻辑放在一个简单的枚举中可以得到一个简洁明了的解决方案。

## 6。结论

在本文中，我们研究了状态机以及如何使用枚举在 Java 中实现它们。我们举了一个例子，并进行了测试。

最后，我们还讨论了使用枚举实现状态机的优点。作为接口和实现解决方案的替代方案，枚举提供了更清晰、更易于理解的状态机实现。

和往常一样，本文中提到的所有代码片段都可以在我们的 [GitHub 资源库](https://web.archive.org/web/20221207040444/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-3)中找到。