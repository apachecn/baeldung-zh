# Jenkins 中的脚本管道与声明管道

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/jenkins-scripted-vs-declarative-pipelines>

## 1.概观

“Pipeline-as-code”是允许 DevOps 中的每个人创建和维护 Jenkins 管道的想法。事实上，在生活中有两种应用“管道即代码”原则的方式:脚本式管道和声明式管道。“管道代码”允许 Jenkins 将管道视为常规文件。参与开发运维的人可以存储、共享这些文件，也可以通过 SCM 使用这些文件。

## 2.声明性管道及其优势

声明性管道是“管道即代码”原则的一种更新的方法。它们很容易编写和理解。结构可能看起来有点复杂，但总的来说，它只包含几个基本部分。“管道”块是包含整个管道声明的主块。在本例中，我们将只考虑 [**【代理】****【阶段】****【步骤】**](https://web.archive.org/web/20221208143832/https://www.jenkins.io/doc/book/pipeline/syntax/#declarative-sections) 部分:

*   **管道**–包含整条管道

让我们看看这个结构在我们的声明性管道中是什么样子的:

```java
pipeline {
    agent any
    stages {
        stage('Hello World') {
            steps {
                sh 'echo Hello World'
            }
        }
    }
}
```

**声明管道的另一个重要部分是[指令](https://web.archive.org/web/20221208143832/https://www.jenkins.io/doc/book/pipeline/syntax/#declarative-directives)** 。事实上，指令是包含附加逻辑的一种便捷方式。它们可以帮助将工具包含到管道中，还可以帮助设置触发器、环境变量和参数。一些指令可以提示用户输入附加信息。**有一个易于使用的[生成器](https://web.archive.org/web/20221208143832/https://www.jenkins.io/doc/book/pipeline/getting-started/#directive-generator)可以帮助创建这些指令。**

在前面的示例中，“steps”是一个包含将在阶段中执行的逻辑的指令。有一个专用的[代码片段生成器](https://web.archive.org/web/20221208143832/https://www.jenkins.io/doc/book/pipeline/getting-started/#snippet-generator)，它提供了一种创建步骤的便捷方式。

声明性管道的力量主要来自于指令。通过使用“script”指令，声明式管道可以利用脚本式管道的功能。该指令将内部的行作为脚本管道执行。

## 3.脚本化管道及其优势

脚本化管道是“管道即代码”原则的第一个版本。它们被设计成使用 Groovy 构建的 DSL，提供了出色的功能和灵活性。然而，这也需要一些 Groovy 的基础知识，有时这并不可取。

这类管道对结构的限制较少。还有，它们只有两个基本块:“节点”和“阶段”。“节点”块指定执行特定流水线的机器，而“阶段”块用于对步骤进行分组，这些步骤放在一起表示一个单独的操作。**由于缺少额外的规则和模块，这些管道很容易理解**:

```java
node {
    stage('Hello world') {
        sh 'echo Hello World'
    }
}
```

将脚本化管道视为声明性管道，但只带有阶段。在这种情况下,“节点”块既扮演“管道”块的角色，也扮演来自声明性管道的“代理”指令的角色。

**如上所述，脚本化管道的步骤可以使用相同的[代码片段生成器](https://web.archive.org/web/20221208143832/https://www.jenkins.io/doc/book/pipeline/getting-started/#snippet-generator)生成。**因为这种类型的管道不包含指令，所以步骤包含所有的逻辑。对于非常简单的管道，这可以减少整体代码。

但是，对于一些样板文件设置，它可能需要额外的代码，这可以通过指令来解决。此类管道中更复杂的逻辑通常用 Groovy 实现。

## 4.脚本管道和声明管道的比较

让我们来看一个三步管道，它从 git 中提取一个项目，然后测试、打包和部署它:

```java
pipeline {
    agent any
    tools {
        maven 'maven' 
    }
    stages {
        stage('Test') {
            steps {
                git 'https://github.com/user/project.git'
                sh 'mvn test'
                archiveArtifacts artifacts: 'target/surefire-reports/**'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests' 
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }
        stage('Deploy') {
            steps {
                sh 'echo Deploy'
            }
        }
    }
}
```

正如我们所看到的，所有的逻辑都存在于“步骤”部分中。因此，如果我们想将此声明性管道转换为脚本化管道，这些部分不会改变:

```java
node {
    stage('Test') {
        git 'https://github.com/user/project.git'
        sh 'mvn test'
        archiveArtifacts artifacts: 'target/surefire-reports/**'
    }
    stage('Build') {
        sh 'mvn clean package -DskipTests' 
        archiveArtifacts artifacts: 'target/*.jar'
    }
    stage('Deploy') {
        sh 'echo Deploy'
    }
}
```

**相同功能的脚本化管道看起来比其声明式管道更加密集**。但是，我们应该确保在服务器上正确设置了所有的环境变量。同时，如果有几个 Maven 版本，我们需要直接在管道中改变它们。为此，我们可以直接使用具体的路径或环境变量。

在脚本化管道中，还有一个“withEnv”步骤很有用。另一方面，使用声明式管道，很容易在 Jenkins 配置中更改工具的版本。

前面的例子表明，对于简单的日常任务，这些方法几乎没有区别。如果 steps 可以涵盖管道的所有基本需求，这两种方法将几乎相同。声明性管道仍然是首选，因为它们可以简化一些常见的逻辑。

## 5.结论

脚本管道和声明管道遵循相同的目标，并使用相同的管道子系统。它们之间的主要区别是灵活性和语法。它们只是解决同一问题的两种不同工具，因此，我们可以也应该互换使用。

声明性管道的简洁语法将确保更快、更顺畅地进入这个领域。同时，脚本化管道可以为更有经验的用户提供更大的能力。为了从两个世界中获得最佳效果，我们可以利用声明性管道和脚本指令。