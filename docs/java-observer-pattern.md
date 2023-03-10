# Java 中的观察者模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-observer-pattern>

## 1。概述

在本教程中，我们将描述 Observer 模式，并看看几个 Java 实现备选方案。

## 2。观察者模式是什么？

观察者是一种行为设计模式。它规定了对象之间的通信:`observable`和`observers`。**`observable`是一个通知`observers`其状态变化的对象。**

例如，新闻机构可以在收到新闻时通知频道。接收新闻是改变新闻机构状态的原因，它导致渠道被通知。

看看我们自己怎么实现吧。

首先，我们将定义`NewsAgency`类:

```java
public class NewsAgency {
    private String news;
    private List<Channel> channels = new ArrayList<>();

    public void addObserver(Channel channel) {
        this.channels.add(channel);
    }

    public void removeObserver(Channel channel) {
        this.channels.remove(channel);
    }

    public void setNews(String news) {
        this.news = news;
        for (Channel channel : this.channels) {
            channel.update(this.news);
        }
    }
}
```

`NewsAgency`是一个可观测值，当`news`更新时，`NewsAgency`的状态也随之改变。当变化发生时，`NewsAgency`通过调用它们的`update()`方法通知观察者。

为了能够做到这一点，可观察对象需要保持对观察者的引用。在我们的例子中，它是`channels`变量。

现在让我们看看观察者这个`Channel` 类会是什么样子。它应该有`update()`方法，当`NewsAgency`的状态改变时调用该方法:

```java
public class NewsChannel implements Channel {
    private String news;

    @Override
    public void update(Object news) {
        this.setNews((String) news);
    } 

    // standard getter and setter
}
```

`Channel`接口只有一个方法:

```java
public interface Channel {
    public void update(Object o);
}
```

现在，如果我们将`NewsChannel`的实例添加到观察者`,`的列表中，并更改`NewsAgency`的状态，那么`NewsChannel`的实例将被更新:

```java
NewsAgency observable = new NewsAgency();
NewsChannel observer = new NewsChannel();

observable.addObserver(observer);
observable.setNews("news");
assertEquals(observer.getNews(), "news");
```

Java 核心库中有一个预定义的`Observer`接口，这使得 observer 模式的实现更加简单。我们来看看。

## 3。`Observer`实现同

[`java.util.Observer`](https://web.archive.org/web/20221005171216/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Observer.html) 接口定义了`update()` 方法，所以没有必要像我们在上一节所做的那样自己定义它。

让我们看看如何在我们的实现中使用它:

```java
public class ONewsChannel implements Observer {

    private String news;

    @Override
    public void update(Observable o, Object news) {
        this.setNews((String) news);
    }

    // standard getter and setter
} 
```

这里，第二个论点来自`Observable,`，我们将在下面看到。

为了定义可观察的`,`,我们需要扩展 Java 的`Observable`类:

```java
public class ONewsAgency extends Observable {
    private String news;

    public void setNews(String news) {
        this.news = news;
        setChanged();
        notifyObservers(news);
    }
}
```

注意，我们不需要直接调用观察者的`update()`方法。我们只需要调用`setChanged()`和`notifyObservers()`，剩下的工作由`Observable`类来完成。

它还包含一个观察者列表，并公开了维护该列表的方法，`addObserver()`和`deleteObserver().`

为了测试结果，我们只需要将观察者添加到这个列表中，并设置新闻:

```java
ONewsAgency observable = new ONewsAgency();
ONewsChannel observer = new ONewsChannel();

observable.addObserver(observer);
observable.setNews("news");
assertEquals(observer.getNews(), "news");
```

`Observer`接口并不完美，从 Java 9 开始就被弃用了。缺点之一是`Observable`不是一个接口，它是一个类，这就是为什么子类不能用作可观察对象。

此外，开发人员可以覆盖一些`Observable`的同步方法，破坏它们的线程安全。

现在我们来看看 [`ProperyChangeListener`](https://web.archive.org/web/20221005171216/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/beans/PropertyChangeListener.html) 界面，推荐过`Observer`。

## 4。`PropertyChangeListener`实现与

**在这个实现中，一个可观察对象必须保持对 [`PropertyChangeSupport`](https://web.archive.org/web/20221005171216/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/beans/PropertyChangeSupport.html) 实例的引用。**当类的属性改变时，发送通知给观察者是有帮助的。

让我们来定义可观察的:

```java
public class PCLNewsAgency {
    private String news;

    private PropertyChangeSupport support;

    public PCLNewsAgency() {
        support = new PropertyChangeSupport(this);
    }

    public void addPropertyChangeListener(PropertyChangeListener pcl) {
        support.addPropertyChangeListener(pcl);
    }

    public void removePropertyChangeListener(PropertyChangeListener pcl) {
        support.removePropertyChangeListener(pcl);
    }

    public void setNews(String value) {
        support.firePropertyChange("news", this.news, value);
        this.news = value;
    }
}
```

使用这个`support`，我们可以添加和删除观察者，并在可观察的状态改变时通知他们:

```java
support.firePropertyChange("news", this.news, value);
```

这里，第一个参数是被观察属性的名称。第二个和第三个参数分别是它的新旧值。

观察员应实施 `[PropertyChangeListener](https://web.archive.org/web/20221005171216/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/beans/PropertyChangeListener.html)`:

```java
public class PCLNewsChannel implements PropertyChangeListener {

    private String news;

    public void propertyChange(PropertyChangeEvent evt) {
        this.setNews((String) evt.getNewValue());
    }

    // standard getter and setter
}
```

由于`PropertyChangeSupport`类为我们做了连接，我们可以从事件中恢复新的属性值。

让我们测试一下实现，以确保它也能工作:

```java
PCLNewsAgency observable = new PCLNewsAgency();
PCLNewsChannel observer = new PCLNewsChannel();

observable.addPropertyChangeListener(observer);
observable.setNews("news");

assertEquals(observer.getNews(), "news");
```

## 5。结论

在本文中，我们研究了在 Java 中实现`Observer`设计模式的两种方法。我们了解到`PropertyChangeListener`方法是首选。

本文的源代码 可以在 GitHub 的[上找到。](https://web.archive.org/web/20221005171216/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-behavioral)