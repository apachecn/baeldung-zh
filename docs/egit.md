# EGit 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/egit>

## 1。概述

在本文中，我们将探索 EGit——Eclipse 的 JGit 库的一个发展。

## 2。埃及设置〔t1〕

在本文中，我们将使用以下工具:

*   Eclipse Neon.3 版本 4.6.3
*   埃及外挂程式 4.8 版

### 2.1。在 Eclipse 中安装 EGit

从 Eclipse Juno 开始，EGit 包含在 Eclipse 本身中。

对于老版本的 Eclipse，我们可以通过`Help -> Install New Software`安装插件，并提供 URL[http://download.eclipse.org/egit/updates](https://web.archive.org/web/20220626202205/http://download.eclipse.org/egit/updates):

[![install egit](img/22b24db8bb9ac426e83d7ea079764685.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/install-egit.png)

### 2.2。识别罪犯

Git 需要跟踪提交后的用户，因此我们应该在通过 EGit 提交时提供我们的身份。

这是通过`Preferences -> Team -> Git -> Configuration`和点击`Add Entry`来完成的，包括`user.name`和`user.email`的信息:

## [![config](img/326d70632d2ae5f84694947fef862c80.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/config.png)

## 3。储存库

### 3.1。存储库视图

EGit 附带了存储库视图，允许我们:

*   浏览我们的本地存储库
*   添加和初始化本地存储库
*   删除存储库
*   克隆远程存储库
*   签出项目
*   管理分支

要打开存储库视图，请单击`Window -> Show View -> Other -> Git -> Git Repositories:`

[![repositories view](img/6ef5942de75741a344e2d837d6037f6a.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/repositories-view.png)

### 3.2。创建新的存储库

我们需要创建一个项目，右击它选择`Team -> Share Project` **、**和`Create.`

从这里，我们选择存储库目录并单击`Finish:`

[![create repo2](img/8431664f0e37ac941912085cc846e6d0.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/create-repo2.png)

### 3.3。克隆存储库

我们可以从远程 git 服务器克隆一个存储库到我们的本地文件系统。

让我们转到`File -> Import… -> Git -> Projects from Git -> Next -> Clone URI -> Next,`，然后将显示以下窗口:

[![clone2](img/fddfac73d80df70d6002df262ec5d1c0.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/clone2.png)

我们也可以从`Repositories`视图选项卡中的`Clone Remote Repository`工具栏按钮打开同一窗口。

Git 支持多种协议，如 https、ssh、git 等。如果我们粘贴远程存储库的 URI，其他条目将被自动填充。

## 4。分支机构

我们将处理两种类型的分支:

*   分公司
*   远程跟踪分支

### 4.1。创建本地分支

我们可以通过点击`Team -> Repository -> Switch to -> New Branch:`创建一个新的本地分支

[![create branch 1](img/0b13836db68ae0ad5fe03546977da34f.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/create-branch-1.png)

我们可以选择远程跟踪分支作为本地分支的基础。向我们新的本地分支添加上游配置将简化本地变更与远程变更的同步。

建议检查对话框`Configure upstream for push and pull.`中的选项

另一种打开新分支对话框的方法是右击`Repositories view -> Switch To -> New Branch`中的分支

### 4.2。检查分支

在`Repositories`视图中，右键单击分支名称，然后单击`Check Out`:

[![checkout1](img/10f0fe260b8f3e89e6555a9225e1c526.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/checkout1.png)

或者右键单击项目并选择`Team -> Switch To -> select the branch name`:

[![checkout2](img/8672fd47cb4e17af75521ee212f42747.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/checkout2.png)

## 5。用 Git 跟踪文件

### 5.1。跟踪变更

问号标记出现在还不在 Git 控制下的文件上。我们可以通过右击它们并选择`Team -> Add to Index`来跟踪这些新文件。

从这里开始，装饰者应该改为 `(+) sign.`

### 5.2。提交更改

我们希望提交对被跟踪文件的更改。右击这些文件并选择`Team -> Commit:`即可

[![commit window](img/38369a572f9918d126cac81591b75f8e.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/commit-window.png)

默认情况下，作者和提交者来自我们主目录中的`.gitconfig`文件。

我们可以输入提交消息来解释这些更改。此外，通过点击右上角的`Add Signed-off-by`图标，我们可以添加一个*签字人*标签。

### 5.3。检查历史记录

我们可以通过右击文件并选择`Team -> Show in History.`来检查文件的历史

历史记录对话框将显示已检查文件的所有已提交更改:

[![history 1](img/2ed1e47eb76c806d5afaba45737e1e16.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/history-1.png)

我们可以在比较视图中打开上次提交的更改，方法是单击历史选项卡右上角的比较模式图标，然后双击文件列表中的文件名(以下是一个示例:`HelloEgit/src/HelloEgitClass.java`):

[![compare](img/5aa9f4cd2155373ece07193ef30068f9.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/compare.png)

### 5.4。将更改推送到远程存储库

为了推动我们的变更，我们需要一个远程 Git 存储库。

从`Team -> Remote -> Push`我们可以在向导中输入新 Git 远程存储库的 https URL:

[![push](img/8be6952e0add051e98b63a52f01fe010.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/push.png)

接下来的步骤是:

*   选择`Add All Branches Spec`将本地分支名称映射到目标存储库中的相同分支名称
*   按下确认按钮–向导将显示已更改文件的预览
*   最后，我们单击`Finish`将我们的存储库推到远程位置。

如果我们已经从第 4.1 节设置了上游配置，这个配置对话框将不会显示，推送将会容易得多。

### 5.5。从上游获取

如果我们使用基于远程跟踪分支的本地分支，我们现在可以从上游获取变更。

为了从上游获取，我们右键单击`project`并选择`Team -> Fetch from Upstream`(或者右键单击`Repositories View`上的存储库并选择`Fetch from Upstream`)。

右键单击项目并选择`Team -> Remote -> Configure Fetch from Upstream:`可以配置该提取

[![configure fetch](img/2095e29bdb82d91ef8b846e7e1fe7cbf.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/configure-fetch.png)

### 5.6。比较&同步

如果我们想要查看本地工作目录和提交的变更之间的变更，我们可以右键单击资源并选择`Compare With`。这将打开`Synchronize View`,允许我们浏览更改:

[![synch 1](img/e2dbccd246ad6d38aeb8d8656e3553d2.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/synch-1.png)

通过双击已更改的文件，将打开比较编辑器，允许我们比较更改。

如果我们想要比较两个提交，我们需要选择`Team -> Show in History.`

从历史视图中，我们将突出显示我们想要比较的两个提交，并选择`Compare with Each Other`选项:

[![compare each other](img/38e383763970dcf6d7ff219dd4bfa373.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/compare-each-other.png)

如果我们想比较工作目录和分支，我们可以使用`Team -> Synchronize`

### 5.7。合并

合并将一个分支或标签中的更改合并到当前检出的分支中。

我们可以通过点击`Team -> Merge`或者右键点击存储库视图中的存储库名称并选择`Merge`来进行合并:

[![merge](img/486b7f04916c6a55b597e145e150fd68.png)](/web/20220626202205/https://www.baeldung.com/wp-content/uploads/2017/09/merge.png)

现在我们可以选择我们想要与当前检出的分支合并的分支或标签。

## 6。结论

在本教程中，我们介绍了 eclipse 的 EGit 插件，如何安装和配置它，以及如何在我们的日常开发中使用它。

关于 EGit 的更多细节，请查看其官方文档[这里](https://web.archive.org/web/20220626202205/https://wiki.eclipse.org/EGit/User_Guide)。