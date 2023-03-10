# 在 JavaFX ListView 中显示自定义项目

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javafx-listview-display-custom-items>

## 1.介绍

JavaFX 是一个强大的工具，旨在为不同平台构建应用程序 UI。它不仅提供 UI 组件，还提供不同的有用工具，比如属性和可观察集合。

组件便于管理收藏。也就是说，我们不需要明确定义`DataModel`或者更新`ListView`元素。一旦`ObjervableList`中发生变化，它就会反映在`ListView`小部件中。

然而，这种方法需要一种在 JavaFX `ListView`中显示自定义项目的方式。本教程描述了一种在`ListView`中设置域对象外观的方法。

## 2.细胞工厂

### 2.1.默认行为

默认情况下，JavaFX 中的`ListView`使用`toString()`方法来显示对象。

因此，显而易见的方法是覆盖它:

```java
public class Person {
    String firstName;
    String lastName;

    @Override
    public String toString() {
        return firstName + " " + lastName;
    }
} 
```

这种方法适用于学习和概念示例。然而，这不是最好的方法。

首先，我们的域类承担显示实现。因此，这种方法与单一责任原则相矛盾。

第二，其他子系统可能使用`toString()`。例如，我们使用`toString()`方法来记录对象的状态。日志可能需要比`ListView`更多的字段。因此，在这种情况下，一个单独的`toString()`实现无法满足每个模块的需求。

### 2.2.在 ListView 中显示自定义对象的单元格工厂

让我们考虑一种在 JavaFX `ListView`中显示自定义对象的更好方法。

`ListView`中的每个项目都显示有一个`ListCell`类的实例。`ListCell`有一个属性叫`text`。一个单元格显示其`text`值。

因此，为了定制`ListCell`实例中的文本，我们应该更新它的`text`属性。我们能在哪里做它？`ListCell`有一个方法叫`updateItem`。当该项目的单元格出现时，它调用`updateItem`。当单元格发生变化时，`updateItem` 方法也会运行。所以我们应该从默认的`ListCell`类继承我们自己的实现。在这个实现中，我们需要覆盖`updateItem`。

但是我们如何让`ListView`使用我们的定制实现而不是默认实现呢？

`ListView`可能有细胞工厂。细胞工厂默认为`null`。我们应该设置它来定制`ListView`显示对象的方式。

让我们用一个例子来说明细胞工厂:

```java
public class PersonCellFactory implements Callback<ListView<Person>, ListCell<Person>> {
    @Override
    public ListCell<Person> call(ListView<Person> param) {
        return new ListCell<>(){
            @Override
            public void updateItem(Person person, boolean empty) {
                super.updateItem(person, empty);
                if (empty || person == null) {
                    setText(null);
                } else {
                    setText(person.getFirstName() + " " + person.getLastName());
                }
            }
        };
    }
}
```

`CellFactory`应该实现一个 JavaFX 回调。JavaFX 中的*回调*接口类似于标准的 Java `Function`接口。但是由于历史原因，JavaFX 使用了一个`Callback`接口。

我们应该调用`updateItem`方法的默认实现。该实现触发默认操作，例如将单元格连接到对象并为空列表显示一行。

方法`updateItem`的默认实现也调用`setText`。然后，它设置将在单元格中显示的文本。

### 2.3.使用自定义小部件在 JavaFX ListView 中显示自定义项目

`ListCell`为我们提供了一个将自定义小部件设置为内容的机会。我们应该做的就是使用`setGraphics()`而不是`setCell().`来在自定义小部件中显示我们的域对象

假设，我们必须将每一行显示为一个`CheckBox`。让我们来看看合适的细胞工厂:

```java
public class CheckboxCellFactory implements Callback<ListView<Person>, ListCell<Person>> {
    @Override
    public ListCell<Person> call(ListView<Person> param) {
        return new ListCell<>(){
            @Override
            public void updateItem(Person person, boolean empty) {
                super.updateItem(person, empty);
                if (empty) {
                    setText(null);
                    setGraphic(null);
                } else if (person != null) {
                    setText(null);
                    setGraphic(new CheckBox(person.getFirstName() + " " + person.getLastName()));
                } else {
                    setText("null");
                    setGraphic(null);
                }
            }
        };
    }
}
```

在这个例子中，我们将`text`属性设置为`null`。如果`text`和`graphic`属性都存在，文本将显示在小部件旁边。

当然，我们可以基于自定义元素数据设置`CheckBox`回调逻辑和其他属性。它需要一些编码，就像设置小部件文本一样。

## 3.结论

在本文中，我们考虑了一种在 JavaFX `ListView`中显示定制项目的方法。我们看到`ListView`允许一种非常灵活的方式来设置它。我们甚至可以在 ListView 单元格中显示定制的小部件。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/javafx)