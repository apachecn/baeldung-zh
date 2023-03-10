# Vue.js 前端带有 Spring Boot 后端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-vue-js>

## 1.概观

在本教程中，我们将查看一个示例应用程序，它使用 Vue.js 前端呈现单个页面，同时使用 Spring Boot 作为后端。

我们还将利用百里香叶向模板传递信息。

## 2.Spring Boot 设置

应用程序`pom.xml`使用 [`spring-boot-starter-thymeleaf`](https://web.archive.org/web/20220701154854/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-boot-starter-thymeleaf) 依赖项以及常用的`[spring-boot-starter-web](https://web.archive.org/web/20220701154854/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-web%22)`进行模板渲染:

```java
<dependency> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId> 
    <version>2.4.0</version>
</dependency> 
<dependency> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId> 
    <version>2.4.0</version>
</dependency>
```

默认情况下，百里香在`templates/`寻找视图模板，我们将为`src/main/resources/templates/index.html`添加一个空的`index.html`。我们将在下一节更新它的内容。

最后，我们的 Spring Boot 控制器将在`src/main/java`:

```java
@Controller
public class MainController {
    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("eventName", "FIFA 2018");
        return "index";
    }
} 
```

这个控制器使用`model.addAttribute`通过 Spring Web 模型对象呈现一个带有传递给视图的数据的模板。

让我们使用以下代码运行应用程序:

```java
mvn spring-boot:run
```

浏览至`http://localhost:8080`查看索引页面。当然，这会是空的。

我们的目标是让页面打印出类似这样的内容:

```java
Name of Event: FIFA 2018

Lionel Messi
Argentina's superstar

Christiano Ronaldo
Portugal top-ranked player
```

## 3.使用 Vue 渲染数据。Js 组件

### 3.1.模板的基本设置

在模板中，让我们加载 Vue.js 和 Bootstrap(可选)来呈现用户界面:

```java
// in head tag

<!-- Include Bootstrap -->

//  other markup

// at end of body tag
<script 
  src="https://cdn.jsdelivr.net/npm/[[email protected]](/web/20220701154854/https://www.baeldung.com/cdn-cgi/l/email-protection)/dist/vue.js">
</script>
<script 
  src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.21.1/babel.min.js">
</script> 
```

这里我们从 CDN 加载 Vue.js，但是如果更好的话，你也可以托管它。

我们在浏览器中加载 Babel，这样我们就可以在页面中编写一些符合 ES6 的代码，而不必运行 transpilation 步骤。

在现实世界的应用程序中，您可能会使用 Webpack 和 Babel transpiler 之类的工具进行构建，而不是使用浏览器内的 Babel。

现在让我们保存页面并使用`mvn spring-boot` : `run`命令重新启动。我们刷新浏览器以查看我们的更新；还没什么有趣的。

接下来，让我们设置一个空的 div 元素，我们将把用户界面附加到这个元素上:

```java
<div id="contents"></div>
```

接下来，我们在页面上设置一个 Vue 应用程序:

```java
<script type="text/babel">
    var app = new Vue({
        el: '#contents'
    });
</script>
```

刚刚发生了什么？这段代码在页面上创建了一个 Vue 应用程序。我们用 CSS 选择器`#contents`将它附加到元素上。

这是指页面上的空的`div`元素。应用程序现在设置为使用 Vue.js！

### 3.2.在模板中显示数据

接下来，让我们创建一个显示从 Spring controller 传来的'`eventName`'属性的 header，并使用百里香的特性渲染它:

```java
<div class="lead">
    <strong>Name of Event:</strong>
    <span th:text="${eventName}"></span>
</div>
```

现在让我们给 Vue 应用程序附加一个'`data'`属性来保存我们的球员数据数组，这是一个简单的 JSON 数组。

我们的 Vue 应用程序现在看起来像这样:

```java
<script type="text/babel">
    var app = new Vue({
        el: '#contents',
        data: {
            players: [
                { id: "1", 
                  name: "Lionel Messi", 
                  description: "Argentina's superstar" },
                { id: "2", 
                  name: "Christiano Ronaldo", 
                  description: "World #1-ranked player from Portugal" }
            ]
        }
    });
</script>
```

现在 Vue.js 知道了一个叫做`players`的数据属性。

### 3.3.使用 Vue.js 组件呈现数据

接下来，让我们创建一个名为`player-card`的 Vue.js 组件，它只呈现一个`player`。**记得在创建 Vue 应用之前注册这个组件。**

不然 Vue 找不到:

```java
Vue.component('player-card', {
    props: ['player'],
    template: `<div class="card">
        <div class="card-body">
            <h6 class="card-title">
                {{ player.name }}
            </h6>
            <p class="card-text">
                <div>
                    {{ player.description }}
                </div>
            </p>
        </div>
        </div>`
});
```

最后，让我们遍历 app 对象中的一组玩家，并为每个玩家呈现一个`player-card`组件:

```java
<ul>
    <li style="list-style-type:none" v-for="player in players">
        <player-card
          v-bind:player="player" 
          v-bind:key="player.id">
        </player-card>
    </li>
</ul>
```

这里的逻辑是名为`v-for,`的 Vue 指令，它将遍历`players`数据属性中的每个玩家，并为`<li>`元素中的每个`player`条目呈现一个`player-card`。

`v-bind:player`意味着`player-card`组件将被赋予一个名为`player`的属性，其值将是当前正在处理的`player`循环变量。需要`v-bind:key`使每个`<li>`元素都是独一无二的。

通常，`player.id`是一个很好的选择，因为它已经是惟一的了。

现在，如果您重新加载该页面，观察在`devtools`中生成的 HTML 标记，它看起来将类似于:

```java
<ul>
    <li style="list-style-type: none;">
        <div class="card">
            // contents
        </div>
    </li>
    <li style="list-style-type: none;">
        <div class="card">
            // contents
        </div>
    </li>
</ul>
```

工作流改进说明:每次修改代码时，重启应用程序和刷新浏览器会变得很麻烦。

因此，为了让生活更简单，请参考[这篇文章](/web/20220701154854/https://www.baeldung.com/spring-boot-devtools)中关于如何使用 Spring Boot `devtools`和自动重启。

## 4.结论

在这篇简短的文章中，我们回顾了如何使用 Spring Boot 作为后端，使用`Vue.js`作为前端来设置一个 web 应用程序。这种方法可以构成更强大和可伸缩的应用程序的基础，而这只是大多数这类应用程序的起点。

像往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220701154854/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-vue)