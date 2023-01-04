# 流浪者指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/vagrant-guide>

## 1。概述

在本教程中，我们将会看到如何使用 vagger 来管理我们的开发环境。我们将讨论一些使用案例，看看基本的流浪者，然后尝试不同的配置选项。

在接下来的步骤中，我们需要在我们的系统上安装 travel[。我们还需要](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/installation) [VirtualBox](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/providers/virtualbox) 来运行这些例子。本教程使用的是流浪者 2.3.2 版。

## 2。用例

**流浪者帮助我们创建和配置[虚拟机](/web/20221129083849/https://www.baeldung.com/cs/containers-vs-virtual-machines#1-what-can-virtual-machines-do)环境。**我们可以一次性描述我们的环境，并在任何想要重新创建的时候使用该配置。因此，它可以加速项目的启动过程，并消除错误配置的风险。

我们甚至可以在本地镜像生产环境。因此，我们的测试可以更加准确，并且我们可以减少部署应用时的意外问题。这也有助于重现生产环境中出现的问题。

## 3。`Vagrantfile`

**流浪者用一个特殊的配置文件来描述一个叫做`Vagrantfile.`** 的环境，命名背后的原因很简单，这些配置文件的名字应该是`Vagrantfile.`

**每个项目可以有一个`Vagrantfile`。**然而，当我们运行流浪者命令时，它**在我们的文件系统上寻找其他的`Vagrantfile`并且[合并相关的](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/vagrantfile#load-order)来构建最终的配置**。例如，我们可以在我们的流浪者主目录中有一个`Vagrantfile`，带有一些默认的配置，并覆盖项目目录中特定于项目的值。

`Vagrantfile` s 使用 [Ruby 编程语言](https://web.archive.org/web/20221129083849/https://www.ruby-lang.org/en/)，但是我们不一定需要了解 Ruby 来创建、理解或者修改这些文件。很多时候，我们使用简单的运算，比如变量赋值。另一方面，如果我们很了解 Ruby，我们可以利用它，[很容易地创建更复杂的流浪文件](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/vagrantfile/tips#loop-over-vm-definitions)。

让我们在名为`vagrant-start`的目录中创建我们的流浪者项目。**我们可以使用 [`init`命令](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/cli/init)** 在这个目录中创建一个初始的`Vagrantfile`:

```java
$ vagrant init hashicorp/bionic64
```

它产生以下输出，我们可以在目录中找到生成的`Vagrantfile`:

```java
A `Vagrantfile` has been placed in this directory. You are now ready to `vagrant up` your first virtual environment! Please read the comments in the Vagrantfile as well as documentation on `vagrantup.com` for more information on using Vagrant. 
```

让我们看一下生成的文件，了解每个部分的作用。为了清楚起见，我们从文件中删除了一些注释:

```java
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
end
```

**首先我们可以看到[版本的配置](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/vagrantfile/version)** : `Vagrant.configure(“2”)`。这意味着 vagger 将使用配置对象的版本 2。这一点很重要，因为不同版本的 vagger 之间向后兼容。

**在这之后，我们可以看到虚拟机** : `config.vm.box = “hashicorp/bionic64”.`的[配置:这是一个简单的变量赋值——我们设置盒子](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/vagrantfile/machine_settings)的[名称。该值正是我们在`init`命令中提供的值。我们将在下一节更详细地讨论盒子。](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/vagrantfile/machine_settings#config-vm-box)

**让我们用 [`validate`命令](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/cli/validate)** 来验证我们的`Vagrantfile`:

```java
$ vagrant validate
```

输出应该如下所示:

```java
Vagrantfile validated successfully.
```

这个命令在我们修改文件的任何时候都很有用，我们希望确保它仍然有效。

## 4。盒子

在《流浪》中，一个环境的基本图像被称为盒子。这些是版本化映像，它们帮助我们快速克隆虚拟机。

在上一节中，我们创建了我们的`Vagrantfile`，并将我们的盒子名称设置为“`[hashicorp/bionic64](https://web.archive.org/web/20221129083849/https://app.vagrantup.com/hashicorp/boxes/bionic64)`”。这里指的是 Ubuntu 18.04 LTS 环境。换句话说，我们只要用这个盒子就可以用 Ubuntu 启动一个虚拟机。

我们可以在[流浪云](https://web.archive.org/web/20221129083849/https://app.vagrantup.com/boxes/search)中找到其他公共的、预定义的盒子。它们可以作为建立我们的环境的良好起点。但是，我们必须小心，因为只有两种类型的[官方包厢](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/boxes#official-boxes)。这些是 [HashiCorp](https://web.archive.org/web/20221129083849/https://app.vagrantup.com/hashicorp) 和[便当盒](https://web.archive.org/web/20221129083849/https://app.vagrantup.com/bento)。

我们甚至可以[创建自己的基盒](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/boxes/base)上传到流浪云。

### 4.1。管理箱

让我们使用之前定义的`Vagrantfile.` **来启动一个虚拟机。流浪者中的 [`up`命令](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/cli/up)创建并配置我们的环境**:

```java
$ vagrant up
```

**该命令下载指定的盒子并启动虚拟机:**

```java
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'hashicorp/bionic64' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: >= 0
==> default: Loading metadata for box 'hashicorp/bionic64'
    default: URL: https://vagrantcloud.com/hashicorp/bionic64
==> default: Adding box 'hashicorp/bionic64' (v1.0.282) for provider: virtualbox
    default: Downloading: https://vagrantcloud.com/hashicorp/boxes/bionic64/versions/1.0.282/providers/virtualbox.box
==> default: Successfully added box 'hashicorp/bionic64' (v1.0.282) for 'virtualbox'! 
```

我们可以看到，流浪者没有在我们的机器上找到盒子，所以它从流浪者云下载了它。为此，它必须查看盒子的[元数据。**盒子可以有一个可选的元数据文件，其中包含关于版本、提供者的信息，或者只是盒子的名称和描述。**](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/boxes/format#box-metadata)

**我们可以用 [`vagrant box`命令](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/cli/box)直接管理箱子。**让我们列出下载的盒子:

```java
$ vagrant box list hashicorp/bionic64 (virtualbox, 1.0.282)
```

这意味着当我们以后想要使用它时，流浪者将不再需要下载这个盒子。

让我们使用 [`vagrant destroy`命令](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/cli/destroy)停止虚拟机:

```java
$ vagrant destroy
```

此命令会停止虚拟机并删除所有相关资源。**但是，下载的盒子仍然可以在本地使用。**要删除它们，我们需要运行 [`vagrant box remove`命令](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/cli/box#box-remove)。

## 5。供应商

盒子是流浪者中的包格式，但是我们需要[提供者](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/providers)能够运行它们。此外，盒子是特定于提供商的。这意味着只为 Hyper-V 定义的流浪者型盒子与 VirtualBox 不兼容。然而，一个盒子可以支持多个提供者来克服这个限制。

在我们之前的例子中，我们使用 VirtualBox，因为它是默认的提供者。**流浪者默认支持三个提供者: [VirtualBox](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/providers/virtualbox) 、 [Hyper-V](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/providers/hyperv) 和 [Docker](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/providers/docker) 。**换句话说，默认情况下，我们可以在自己的游民环境中运行 VirtualBox 或者 Hyper-V 虚拟机，甚至 Docker 容器。

让我们列出可用的框，并检查该命令的输出:

```java
$ vagrant box list
```

输出由两个重要部分组成:

```java
hashicorp/bionic64 (virtualbox, 1.0.282)
```

首先，我们可以看到我们在前面的例子中使用的盒子名称。除此之外，括号中还有提供者和盒子的版本。这是有用的信息，因此我们知道特定的机器使用哪个提供者。

可以使用流浪者的插件系统安装额外的提供者[。这应该是一个相当简单的过程，因为](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/providers/installation) [CLI 提供了管理插件的命令](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/cli/plugin)。此外，我们可以[开发新的定制提供商](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/plugins/providers)来进一步拓展我们的可能性。

## 6。供应

通常，简单的盒子对于我们的用例来说是不够的。他们只为我们提供必需品。我们需要安装和配置必要的组件来拥有一个有用的开发环境。我们每次都可以手动完成，但是这既费时又容易出错。 **[供应](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/provisioning)自动化定制开发环境的过程。[当我们第一次用`vagrant up`命令创建环境时](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/provisioning#when-provisioning-happens)就会发生，但是我们也可以[手动运行](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/cli/provision)。**

我们可以有[多个供应商](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/provisioning/basic_usage#multiple-provisioners)每个项目。除此之外，还有不同类型的置备程序。例如，我们可以[运行 shell 脚本](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/provisioning/shell)或[将文件](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/provisioning/file)复制到虚拟机。让我们在`Vagrantfile:`中定义一个简单的 shell provisioner

```java
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"

  config.vm.provision "shell",
    inline: "echo Hello from provisioner"
end 
```

这将在供应运行时打印“`Hello from provisioner`”消息:

```java
$ vagrant up ==> default: Running provisioner: shell...
    default: Running: inline script
    default: Hello from provisioner
```

## 7。联网

在前面的步骤中，我们成功地创建并初始化了我们的开发环境，但是我们还不能访问它。为了访问机器，我们需要配置[联网](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/networking)。我们可以连接到[私有](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/networking/private_network)或[公共](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/networking/public_network)网络或配置[端口转发](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/networking/forwarded_ports)。

最基本的配置选项是端口转发。在这种情况下，我们在主机的一个端口上访问客户机的一个端口。

让我们创建一个新的开发环境并进行配置。首先，对于我们的示例，我们需要创建一个名为“`provision.sh`”的新配置脚本，它在我们创建环境时安装 Nginx:

```java
#!/usr/bin/env bash

apt-get -y update
apt-get install -y nginx
service nginx start
```

在此之后，让我们创建一个使用此脚本进行配置的`Vagrantfile`:

```java
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"

  config.vm.provision :shell, path: "provision.sh"

  config.vm.network "forwarded_port", guest: 80, host: 8080
end
```

我们使用我们的`Vagrantfile`中的 [`config.vm.network`](https://web.archive.org/web/20221129083849/https://developer.hashicorp.com/vagrant/docs/vagrantfile/machine_settings#config-vm-network) 设置将我们主机的端口`8080`转发到来宾中的端口`80`。让我们运行这个例子，并在浏览器中打开`[http://localhost:8080/](https://web.archive.org/web/20221129083849/http://localhost:8080/)`。我们可以看到一个 Nginx 欢迎屏幕，这意味着我们可以访问虚拟机内部的 web 服务器。

## 8。结论

在本文中，我们看到了一个强大的工具，它帮助我们在项目中采用基础设施即代码的实践。我们使用虚拟机初始化了一个开发环境。

我们使用`Vagrantfile` s 来指定我们想要运行哪个机器，并使用 provisioning 来自动化初始设置。最后，我们配置了端口转发来访问虚拟机内部的 web 服务器。