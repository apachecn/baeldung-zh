# Hibernate load()方法中的代理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-proxy-load-method>

## 1。概述

在本教程中，我们将了解在 Hibernate 的`load()`方法的上下文中什么是代理。

对于刚接触 Hibernate 的读者，可以考虑先熟悉一下[的基础知识](/web/20221129020801/https://www.baeldung.com/hibernate-4-spring)。

## 2。代理人简介`load()`方法

**根据定义，[代理](https://web.archive.org/web/20221129020801/https://www.dictionary.com/browse/proxy)是“被授权充当另一个代理或替代者的职能”**。

当我们调用`Session.load()`来创建我们想要的实体类的`uninitialized proxy `时，这也适用于 Hibernate。

简单来说，Hibernate 使用`CGLib`库对我们的实体类进行子类化。除了`@Id`方法，代理实现将所有其他属性方法委托给 Hibernate 会话来填充实例，有点像:

```java
public class HibernateProxy extends MyEntity {
    private MyEntity target;

    public String getFirstName() {
        if (target == null) {
            target = readFromDatabase();
        }
        return target.getFirstName();
    }
}
```

这个子类将被返回，而不是直接查询数据库。

一旦某个实体方法被调用，该实体就会被加载，并在此时变成一个`initialized proxy.`

## 3。`loading`代理人和懒人

### 3.1。单一实体

让我们把`Employee `想成一个实体。首先，我们假设它与任何其他表都没有关系。

如果我们使用`Session.load()`来实例化一个`Employee`:

```java
Employee albert = session.load(Employee.class, new Long(1));
```

然后 Hibernate 会创建一个未初始化的代理`Employee`。**它将包含我们给它的 ID，但除此之外没有其他值，因为我们还没有访问数据库。**

然而，一旦我们调用了`albert`上的一个方法:

```java
String firstName = albert.getFirstName();
```

然后 Hibernate 将在`employee` 数据库表中查询主键为 1 的实体，用相应行中的属性填充`albert`。

如果找不到行，Hibernate 抛出一个`ObjectNotFoundException`。

### 3.2。一对多关系

现在，让我们也创建一个`Company `实体，其中一个`Company `有许多`Employees:`

```java
public class Company {
    private String name;
    private Set<Employee> employees;
}
```

如果我们这次对公司使用`Session.load() `:

```java
Company bizco = session.load(Company.class, new Long(1));
String name = bizco.getName();
```

然后像以前一样填充公司的属性，除了雇员集有一点不同。

**看，我们只查询了公司行，但是代理将保留雇员集，直到我们根据获取策略调用`getEmployees`。**

### 3.3。多对一关系

相反方向的情况是相似的:

```java
public class Employee {
    private String firstName;
    private Company workplace;
}
```

如果我们再次使用`load()`:

```java
Employee bob = session.load(Employee.class, new Long(2));
String firstName = bob.getFirstName();
```

**`bob `现在将被初始化，实际上，`workplace`现在将被设置为未初始化的代理，这取决于获取策略。**

## 4。懒惰加载

现在，`load() `不会总是给我们一个未初始化的代理。事实上， [`Session ` java doc](https://web.archive.org/web/20221129020801/https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/Session.html#load(java.lang.Class,%20java.io.Serializable)) 提醒我们(重点后加):

> 当访问非标识符方法时，这个方法`might`返回按需初始化的代理实例。

发生这种情况的一个简单例子是批量大小。

假设我们在我们的`Employee `实体上使用`@BatchSize`:

```java
@Entity
@BatchSize(size=5)
class Employee {
    // ...
}
```

这次我们有三名员工:

```java
Employee catherine = session.load(Employee.class, new Long(3));
Employee darrell = session.load(Employee.class, new Long(4));
Employee emma = session.load(Employee.class, new Long(5));
```

如果我们在`catherine`上调用`getFirstName `:

```java
String cathy = catherine.getFirstName();
```

然后，实际上，Hibernate 可能决定一次加载所有三个雇员，将这三个雇员都变成初始化的代理。

然后，当我们叫`darrell`的名字时:

```java
String darrell = darrell.getFirstName();
```

然后 **Hibernate 根本不打数据库。**

## 5。急切加载

### 5.1。使用`get()`

我们也可以完全绕过代理，让 Hibernate 使用`Session.get()`加载真实的东西:

```java
Employee finnigan = session.get(Employee.class, new Long(6));
```

这将立即调用数据库，而不是返回一个代理。

**实际上，如果`finnigan `不存在，它将返回`null`而不是`ObjectNotFoundException`。**

### 5.2。性能影响

在`get()`方便的同时，`load()`在数据库上可以轻一点。

例如，假设`gerald `将为一家新公司工作:

```java
Employee gerald = session.get(Employee.class, new Long(7));
Company worldco = (Company) session.load(Company.class, new Long(2));
employee.setCompany(worldco);        
session.save(employee);
```

因为我们知道在这种情况下我们只会改变`employee `记录，所以`,` 称`load()`为`Company `是明智的。

**如果我们在`Company`上调用`get()`，那么我们已经从数据库中加载了所有不必要的数据。**

## 6。结论

在本文中，我们简要地了解了`Hibernate`代理是如何工作的，以及它如何影响带有实体及其关系的`load`方法。

此外，我们快速看了一下`load()`与`get().`的不同之处

像往常一样，教程附带的完整源代码可以在 GitHub 上获得。