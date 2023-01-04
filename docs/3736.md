# 如何在 YAML 为 POJO 定义地图？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/yaml-map-pojo>

## 1.概观

在本教程中，我们将介绍如何使用 YAML 文件中定义的属性来配置 POJO 类中的`Map`的值。

## 2.波乔和 YAML

POJO 类是普通的旧 Java 对象。YAML 是一种人类可读的结构化数据格式，它使用缩进来表示嵌套。

### 2.1.简单的`Map`例子

让我们想象一下，我们正在运营一个在线商店，我们正在创建一个翻译服装尺寸的服务。起初，我们只在英国卖衣服。我们想知道标签“S”、“M”、“L”等指的是什么英国尺寸。我们创建我们的 POJO 配置类:

```
@ConfigurationProperties(prefix = "t-shirt-size")
public class TshirtSizeConfig {

    private Map<String, Integer> simpleMapping;

    public TshirtSizeConfig(Map<String, Integer> simpleMapping) {
        this.simpleMapping = simpleMapping;
    }

    //getters and setters..
} 
```

注意带有`prefix`值的 [`@ConfigurationProperties`](/web/20220524003218/https://www.baeldung.com/configuration-properties-in-spring-boot) 。我们将在 YAML 文件中的同一个根值下定义映射，这将在下一节中看到。

我们还需要记住在我们的`Application.class`上使用以下注释来启用配置属性:

```
@EnableConfigurationProperties(TshirtSizeConfig.class)
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### 2.2.YAML 构型

现在我们将`t-shirt-size`添加到我们的 YAML 配置中。

我们可以在我们的`application.yml`文件中使用以下结构:

```
t-shirt-size:
  simple-mapping:
    XS: 6
    S:  8
    M:  10
    L:  12
    XL: 14
```

注意缩进和空格。YAML 用缩进来表示嵌套。建议的语法是每个嵌套级别两个空格。

注意我们是如何将`simple-mapping`与破折号一起使用的，但是我们在类中的属性名是`simpleMapping`。带破折号的 YAML 属性将自动转换为代码中的大小写形式。

### 2.3.更复杂的`Map`例子

在我们成功的英国商店之后，我们现在需要考虑将尺寸转换成其他国家的尺寸。例如，我们现在想知道在法国和美国标签“S”的尺寸是多少。我们需要在配置中添加另一层数据。

我们可以用更复杂的映射来改变我们的`application.yml`:

```
t-shirt-size:
  complex-mapping:
    XS:
      uk: 6
      fr: 34
      us: 2
    S:
      uk: 8
      fr: 36
      us: 4
    M:
      uk: 10
      fr: 38
      us: 6
    L:
      uk: 12
      fr: 40
      us: 8
    XL:
      uk: 14
      fr: 42
      us: 10 
```

POJO 中的相应字段将是地图的地图:

```
private Map<String, Map<String, Integer>> complexMapping;
```

## 3.结论

在本文中，我们看到了如何在 YAML 配置文件中为简单的 POJO 定义简单和更复杂的嵌套映射。

这篇文章的代码可以在 GitHub 的[上找到](https://web.archive.org/web/20220524003218/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-3)