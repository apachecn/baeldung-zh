# Jenkinsfile 中的注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/jenkinsfile-comments>

## 1。概述

在本教程中，我们将看到如何在`Jenkinsfile`中使用注释。我们将讨论不同类型的注释及其语法。

## 2。`Jenkinsfile`评论一

`Jenkinsfile`的语法是基于 Groovy 的，所以可以使用 [Groovy 语法](/web/20220625074232/https://www.baeldung.com/groovy-language)进行注释。让我们举一个简单的 Pipeline Linter 的例子，试着把它注释掉。

### 2.1。单行注释

**`Jenkinsfile`中的单行注释与我们在 Java、C++和 C#** 等流行语言中看到的一样。

它们以两个正斜杠(`//`)开始。在`Jenkinsfile`中， `//`和行尾之间的任何文本都被注释并忽略。

让我们使用单行注释来注释掉一个基本的管道定义:

```
//pipeline {
//    agent any
//    stages {
//        stage('Initialize') {
//            steps {
//                echo 'Hello World'
//            }
//        }
//    }
//}
```

### 2.2。块注释

`Jenkinsfile`中的块注释用于注释掉代码块。同样，模式类似于 Java 和 C++。

块注释**以正斜杠后跟星号(/*)开始，以星号后跟正斜杠(*/)** 结束。开始字符(/*)和结束字符(*/)将被添加到适当的位置，以将所选块标记为注释。

在本例中，我们将使用块注释来注释掉管道定义:

```
/*
pipeline {
    agent any
    stages {
        stage('Initialize') {
            steps {
                echo 'Placeholder.'
            }
        }
    }
}
*/ 
```

### 2.3。Shell 脚本中的注释

在 shell `(sh) `部分中，我们将**使用 shell 注释字符 hash `(#)`** 进行注释:

```
pipeline {
    agent any
    stages {
        stage('Initialize') {
            steps {
                sh '''
                cd myFolder
                # This is a comment in sh & I am changing the directory to myFolder
                '''
            }
        }
    }
}
```

## 3。结论

在这篇短文中，我们在一个`Jenkinsfile`中讨论了不同类型的评论。首先，我们看了单行注释。接下来，我们看到了如何使用块注释。最后，我们展示了如何在`Jenkinsfile`的 shell 部分添加注释。