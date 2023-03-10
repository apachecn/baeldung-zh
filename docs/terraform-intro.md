# Terraform 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/terraform-intro>

## 1.概观

随着公共云提供商(如 AWS、Google 和 Microsoft)越来越受欢迎，基础设施即代码(IaC)已经成为主流实践。**简而言之，它包括管理一组资源(计算、网络、存储等)。)使用开发人员用来管理应用程序代码的相同方法**。

在本教程中，我们将快速浏览 Terraform，这是 DevOps 团队用来自动化基础架构任务的最流行的工具之一。Terraform 的主要吸引力在于，我们只需声明`what`我们的基础设施应该是什么样子，该工具将决定必须采取哪些行动来“具体化”该基础设施。

## 2.简史

根据 GitHub，Terraform 的[第一次提交](https://web.archive.org/web/20220628065609/https://github.com/hashicorp/terraform/tree/649cf336e86da10910f6e50ea3cfaad507a2e336)的日期是 2014 年 5 月 21 日。作者是 Hashicorp 的创始人之一 Mitchell Hashimoto，其中包含一个[自述](https://web.archive.org/web/20220628065609/https://github.com/hashicorp/terraform/blob/649cf336e86da10910f6e50ea3cfaad507a2e336/README.md)文件，描述了我们可以称之为“使命宣言”的内容:

> Terraform 是一种安全有效地建设和改造基础设施的工具。

这个短语很好地描述了它的意图。从那以后，该工具在它所支持的基础设施提供者方面稳步增加了它的能力。

在撰写本文时，Terraform [正式支持大约 130 个提供商](https://web.archive.org/web/20220628065609/https://www.terraform.io/docs/providers/index.html)。其社区支持的供应商页面列出了另外 160 家。其中一些提供者只公开了很少的资源，但其他提供者，如 AWS 或 Azure，拥有数百个资源。

如此多的支持资源使得 Terraform 成为许多 DevOps 工程师的首选工具。此外，使用一个工具来管理多个供应商是一个很大的优势。

## 3.你好，地形

在深入其内部工作的更多细节之前，让我们从基本的东西开始:初始设置和一个快速的“Hello，World”风格的项目。

### 3.1.下载并安装

Terraform 发行版包含一个二进制文件，我们可以从 Hashicorp 的[下载页面](https://web.archive.org/web/20220628065609/https://www.terraform.io/downloads.html)免费下载。没有依赖性，我们可以简单地通过将可执行二进制文件复制到操作系统的`PATH`中的某个文件夹来运行它。

完成这一步后，我们可以用一个简单的命令来检查它是否正常工作:

```java
$ terraform -v
Terraform v0.12.24
```

就是这样，不需要管理员权限！我们可以通过不带任何参数地运行 Terraform 来快速获得可用命令的帮助:

```java
$ terraform
Usage: terraform [-version] [-help] <command> [args]
... help content omitted
```

### 3.2.创建我们的第一个项目

一个 Terraform 项目只是一个包含资源定义的目录中的一组文件。那些按照惯例以`.tf`结尾的文件，使用 [Terraform 的配置语言](https://web.archive.org/web/20220628065609/https://www.terraform.io/docs/configuration/index.html)来定义我们想要创建的资源。

对于我们的“Hello，Terraform”项目，我们的资源将只是一个具有固定内容的文件。让我们通过打开命令 shell 并键入几个命令来看看这是什么样子:

```java
$ cd $HOME
$ mkdir hello-terraform
$ cd hello-terraform
$ cat > main.tf <<EOF
provider "local" {
  version = "~> 1.4"
}
resource "local_file" "hello" {
  content = "Hello, Terraform"
  filename = "hello.txt"
}
EOF
```

`main.tf`文件包含两个块:一个`provider`声明和一个`resource`定义。`provider`声明声明我们将使用版本 1.4 或兼容版本的`local`提供者。

接下来，我们有一个名为`hello`的`resource`定义，类型为`local_file` **`.`** 这种`resource`类型，顾名思义，就是本地文件系统上具有给定内容的文件。

### 3.3.`init,` `plan,`和`apply`

现在，让我们继续在这个项目上运行 Terraform。因为这是我们第一次运行这个项目，我们需要用`init`命令初始化它:

```java
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "local" (hashicorp/local) 1.4.0...

Terraform has been successfully initialized!
... more messages omitted
```

在这一步中，Terraform 扫描我们的项目文件并下载任何需要的`provider`——在我们的例子中是`local` 提供者。

接下来，我们使用`plan`命令来验证 Terraform 将执行什么操作来创建我们的资源。这一步的工作方式非常类似于其他构建系统中的“预演”特性，比如 GNU 的 make 工具:

```java
$ terraform plan
... messages omitted
Terraform will perform the following actions:

  # local_file.hello will be created
  + resource "local_file" "hello" {
      + content              = "Hello, Terraform"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "hello.txt"
      + id                   = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
... messages omitted 
```

这里，Terraform 告诉我们，它需要创建一个新的资源，这是意料之中的，因为它还不存在。我们还可以看到我们设置的值和一对权限属性。因为我们没有在资源定义中提供这些，所以提供者将采用默认值。

我们现在可以使用`apply `命令进行实际的资源创建:

```java
$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # local_file.hello will be created
  + resource "local_file" "hello" {
      + content              = "Hello, Terraform"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "hello.txt"
      + id                   = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

local_file.hello: Creating...
local_file.hello: Creation complete after 0s [id=392b5481eae4ab2178340f62b752297f72695d57]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

现在，我们可以验证该文件是使用指定的内容创建的:

```java
$ cat hello.txt
Hello, Terraform
```

一切都好！现在，让我们看看如果我们重新运行`apply` 命令会发生什么，这一次使用`-auto-approve` 标志，这样 Terraform 会立即运行而不要求任何确认:

```java
$ terraform apply -auto-approve
local_file.hello: Refreshing state... [id=392b5481eae4ab2178340f62b752297f72695d57]

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
```

这一次，Terraform 什么也没做，因为文件已经存在。然而，这还不是全部。**有时资源存在，但有人可能改变了它的一个属性，这种情况通常被称为“配置漂移”** `.`让我们看看 Terraform 在这种情况下的表现:

```java
$ echo foo > hello.txt
$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

local_file.hello: Refreshing state... [id=392b5481eae4ab2178340f62b752297f72695d57]

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # local_file.hello will be created
  + resource "local_file" "hello" {
      + content              = "Hello, Terraform"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "hello.txt"
      + id                   = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
... more messages omitted
```

**Terraform 检测到了`hello.txt`文件内容的变化，并生成了一个恢复它的计划。**由于`local`提供者缺乏对就地修改的支持，我们看到该计划只包含一个步骤——重新创建文件。

我们现在可以再次运行`apply`，结果，它会将文件的内容恢复到预期的内容:

```java
$ terraform apply -auto-approve
... messages omitted
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

$ cat hello.txt
Hello, Terraform
```

## 4.核心概念

现在我们已经介绍了基础知识，让我们进入 Terraform 的核心概念。

### 4.1.提供者

一个`p` `rovider`很像一个操作系统的设备驱动程序。**它公开了一组使用通用抽象的`resource` 类型，从而掩盖了如何创建、修改和销毁资源的细节，对用户来说非常透明**。

Terraform 根据给定项目的资源，根据需要自动从其[公共注册表](https://web.archive.org/web/20220628065609/https://registry.terraform.io/)下载提供程序。它也可以使用自定义插件，这些插件必须由用户手动安装。最后，一些内置的提供程序是主二进制文件的一部分，并且总是可用的。

除了少数例外，使用提供者需要用一些参数对其进行配置。这些因提供商而异，但一般来说，我们需要提供凭证，以便它可以访问其 API 并提交请求。

虽然不是绝对必要的，但显式声明我们将在 Terraform 项目中使用哪个提供者并告知其版本被认为是一个好的做法。为此，我们使用可用于任何`provider`声明的`version`属性:

```java
provider "kubernetes" {
  version = "~> 1.10"
}
```

**在这里，由于我们没有提供任何额外的参数，Terraform 将在其他地方寻找所需的参数**。在这种情况下，提供者的实现使用与`kubectl`相同的位置来查找连接参数。其他常见的方法是使用环境变量和变量文件`,` ，它们只是包含键值对的文件。

### 4.2.资源

**在 Terraform 中，`r` `esource` 是在给定提供者的上下文中可以作为 CRUD 操作目标的任何东西。**一些例子是 EC2 实例、Azure MariaDB 或 DNS 条目。

让我们看一个简单的资源定义:

```java
resource "aws_instance" "web" {
  ami = "some-ami-id"
  instance_type = "t2.micro"
}
```

首先，我们总是有开始定义的关键字`resource`。接下来，我们有资源类型`,`，它通常遵循`provider_type`约定。在上面的例子中，`aws_instance`是由 AWS 提供者定义的资源类型，用于定义 EC2 实例。之后，是用户定义的资源名称，**，它在同一个模块**中对于这个资源类型必须是唯一的——稍后将在模块中详细介绍。

最后，我们有一个包含一系列参数的块，用作资源规范。关于资源的一个关键点是，一旦创建了资源，我们就可以使用表达式来查询它们的属性。同样重要的是，**我们可以使用这些属性作为其他资源的参数**。

为了说明这是如何工作的，让我们通过在非默认 VPC(虚拟私有云)中创建 EC2 实例来扩展前面的示例:

```java
resource "aws_instance" "web" {
  ami = "some-ami-id"
  instance_type = "t2.micro"
  subnet_id = aws_subnet.frontend.id
}
resource "aws_subnet" "frontend" {
  vpc_id = aws_vpc.apps.id
  cidr_block = "10.0.1.0/24"
}
resource "aws_vpc" "apps" {
  cidr_block = "10.0.0.0/16"
} 
```

这里，我们使用来自 VPC 资源的`id` 属性作为前端的`vpc_id`参数的值。接下来，它的`id` 参数成为 EC2 实例的参数。**请注意，这种特殊的语法需要 Terraform 版本 0.12 或更高版本**。以前的版本使用了更麻烦的`“${expression}”`语法，它仍然可用，但被认为是遗留的。

这个例子还展示了 Terraform 的一个优势:不管我们在项目中声明资源的顺序如何，它都会根据解析资源时构建的依赖图，计算出创建或更新资源的正确顺序。

### 4.3.`count`和`for_each`元参数

`count`和`for_each` 元参数允许我们创建任何资源的多个实例。它们之间的主要区别在于，`count` 期望一个非负数，而`for_each `接受一个值列表或映射。

例如，让我们使用`count`在 AWS 上创建一些 EC2 实例:

```java
resource "aws_instance" "server" {
  count = var.server_count 
  ami = "ami-xxxxxxx"
  instance_type = "t2.micro"
  tags = {
    Name = "WebServer - ${count.index}"
  }
}
```

**在使用`count`的资源中，我们可以在表达式**中使用`count`对象。这个对象只有一个属性:`index`，它保存每个实例的索引(从零开始)。

同样，我们可以使用`for_each`元参数来创建基于映射的那些实例:

```java
variable "instances" {
  type = map(string)
}
resource "aws_instance" "server" {
  for_each = var.instances 
  ami = each.value
  instance_type = "t2.micro"
  tags = {
    Name = each.key
  }
}
```

这一次，我们使用从标签到 AMI (Amazon 机器映像)名称的映射来创建我们的服务器。在我们的资源中，我们可以使用`each`对象，该对象允许我们访问特定实例的当前`key`和`value `。

**关于`count `和`for_each` 的一个关键点是，虽然我们可以给它们分配表达式，但是 Terraform 必须能够在执行任何资源动作**之前解析它们的值。因此，我们不能使用依赖于其他资源的输出属性的表达式。

### 4.4.数据源

从某种意义上来说，数据源的工作方式很像“只读”资源，我们可以获得现有数据源的信息，但不能创建或更改它们。它们通常用于获取创建其他资源所需的参数。

一个典型的例子是 AWS provider 中可用的`aws_ami`数据源，我们使用它从现有的 AMI 中恢复属性:

```java
data "aws_ami" "ubuntu" {
  most_recent = true
  filter {
    name   = "name"
    values = ["ubunimg/hvm-ssd/ubuntu-trusty-14.04-amd64-server-*"]
  }
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  owners = ["099720109477"] # Canonical
}
```

这个例子定义了一个名为“ubuntu”的`data`源，它查询 AMI 注册表并返回与所定位图像相关的几个属性。然后我们可以在其他资源定义中使用这些属性，在属性名前面加上前缀`data`:

```java
resource "aws_instance" "web" {
  ami = data.aws_ami.ubuntu.id 
  instance_type = "t2.micro"
}
```

### 4.5.状态

**terra form 项目的状态是一个文件，它存储了在给定项目**的上下文中创建的资源的所有详细信息。例如，如果我们在项目中声明一个`azure_resourcegroup`资源并运行 Terraform，状态文件将存储它的标识符。

状态文件的主要目的是提供关于已经存在的资源的信息，所以当我们修改我们的资源定义时，Terraform 可以计算出它需要做什么。

关于状态文件的重要一点是它们可能包含敏感信息。示例包括用于创建数据库的初始密码、私钥等等。

Terraform 使用`backend`的概念来存储和检索状态文件。默认的后端是`local` 后端，它使用项目根文件夹中的一个文件作为它的存储位置。我们还可以通过在项目的一个`.tf`文件的`terraform`块中声明来配置一个替代的`remote` 后端:

```java
terraform {
  backend "s3" {
    bucket = "some-bucket"
    key = "some-storage-key"
    region = "us-east-1"
  }
}
```

### 4.6.模块

Terraform 模块的主要特点是允许我们在多个项目中重用资源定义，或者在单个项目中进行更好的组织。这很像我们在标准编程中所做的:不是一个包含所有代码的文件，而是跨多个文件和包组织我们的代码。

模块只是包含一个或多个资源定义文件的目录。事实上，即使我们将所有代码放在一个文件/目录中，我们仍然在使用模块——在这种情况下，只有一个模块。重要的一点是，子目录不是模块的一部分。相反，父模块必须使用`module`声明显式包含它们:

```java
module "networking" {
  source = "./networking"
  create_public_ip = true
}
```

这里我们引用了位于“networking”子目录中的一个模块，并向它传递了一个参数——在本例中是一个`boolean`值。

值得注意的是，在其当前版本中，Terraform 不允许使用`count`和`for_each`来创建一个模块的多个实例。

### 4.7.输入变量

任何模块，包括顶层或主模块，都可以使用`variable `块定义来定义几个输入变量:

```java
variable "myvar" {
  type = string
  default = "Some Value"
  description = "MyVar description"
}
```

一个变量有一个`type` **`, `** ，可以是`string`、`map`或`set`等等。它也`may`有一个默认值和描述。对于在顶层模块定义的变量，Terraform 将使用几个来源为变量分配实际值:

*   `-var`命令行选项
*   `.tfvar`文件，使用命令行选项或扫描众所周知的文件/位置
*   以`TF_VAR_`开头的环境变量
*   变量的`default` 值(如果存在)

至于在嵌套或外部模块中定义的变量，任何没有默认值的变量都必须使用`module`引用中的参数来提供。**如果我们试图使用一个需要输入变量值的模块，但我们未能提供，Terraform 将产生一个错误。**

一旦定义完毕，我们就可以在表达式中使用带有前缀`var`的变量:

```java
resource "xxx_type" "some_name" {
  arg = var.myvar
}
```

### 4.8.输出值

根据设计，模块的使用者无权访问模块中创建的任何资源。然而，有时我们需要这些属性中的一些作为另一个模块或资源的输入。**为了解决这些情况，一个模块可以定义`output `块，这些块暴露了所创建资源的子集**:

```java
output "web_addr" {
  value = aws_instance.web.private_ip
  description = "Web server's private IP address"
}
```

这里我们定义了一个名为“web_addr”的输出值，它包含了我们的模块创建的 EC2 实例的 IP 地址。现在任何引用我们模块的模块都可以在表达式中使用这个值，如`module.module_name.web_addr` **，**，其中`module_name`是我们在相应的`module`声明中使用的名称。

### 4.9.局部变量

局部变量像标准变量一样工作，但是它们的范围被限制在声明它们的模块中。使用局部变量有助于减少代码重复，尤其是在处理模块的输出值时:

```java
locals {
  vpc_id = module.network.vpc_id
}
module "network" {
  source = "./network"
}
module "service1" {
  source = "./service1"
  vpc_id = local.vpc_id
}
module "service2" {
  source = "./service2"
  vpc_id = local.vpc_id
}
```

这里，局部变量`vpc_id` 从`network` 模块接收输出变量的值。稍后，我们将这个值作为参数传递给`service1` 和`service2` 模块。

### 4.10.工作区

Terraform 工作空间允许我们为同一个项目保存多个状态文件。当我们在一个项目中第一次运行 Terraform 时，生成的状态文件会进入`default`工作区。稍后，我们可以用`terraform workspace new`命令创建一个新的工作区，可选地提供一个现有的状态文件作为参数。

**我们可以像在常规 VCS** 中使用分支一样使用工作区。例如，我们可以为每个目标环境——开发、QA、生产——提供一个工作区，通过切换工作区，我们可以在添加新资源时进行`terraform apply`更改。

鉴于这种工作方式，工作区是管理同一套配置的多个版本——或者“化身”——的绝佳选择。对于那些不得不处理臭名昭著的“在我的环境中工作”问题的人来说，这是一个好消息，因为它允许我们确保所有环境看起来都一样。

在某些场景中，基于我们所针对的特定工作空间，禁用某些资源的创建可能会很方便。对于那些场合，我们可以使用`terraform.workspace`预定义变量。这个变量包含当前工作空间的名称，我们可以像在表达式中使用其他变量一样使用它。

## 5.结论

Terraform 是一个非常强大的工具，可以帮助我们在项目中采用“基础设施即代码”实践。然而，这种力量也伴随着挑战。在本文中，我们提供了这个工具的一个快速概述，这样我们可以更好地理解它的功能和基本概念。

像往常一样，GitHub 上的所有代码[都是可用的。](https://web.archive.org/web/20220628065609/https://github.com/eugenp/tutorials/tree/master/terraform)