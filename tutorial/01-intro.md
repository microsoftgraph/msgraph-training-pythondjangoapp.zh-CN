<!-- markdownlint-disable MD002 MD041 -->

本教程指导你如何生成使用 Microsoft Graph API 检索用户的日历信息的 Python Django Web 应用。

> [!TIP]
> 如果你更喜欢仅下载已完成的教程，可以通过两种方式下载它。
>
> - 下载 [Python 快速入门](https://developer.microsoft.com/graph/quick-start?platform=option-Python) ，以在数分钟内获取工作代码。
> - 下载或克隆 [GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp)。

## <a name="prerequisites"></a>先决条件

在开始本教程之前，应在开发计算机上安装 Python [ (](https://www.python.org/)[管道](https://pypi.org/project/pip/)) 管道连接。 如果没有 Python，请访问上一链接，查看下载选项。

你还应该拥有具有邮箱的个人 Microsoft 帐户Outlook.com或 Microsoft 工作或学校帐户。 如果你没有 Microsoft 帐户，则有几个选项可以获取免费帐户：

- 你可以 [注册新的个人 Microsoft 帐户](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。
- 你可以 [注册 Office 365 开发人员计划](https://developer.microsoft.com/office/dev-program) ，获取免费的 Office 365 订阅。

> [!NOTE]
> 本教程使用 Python 版本 3.9.0 和 Django 版本 3.1.4 编写。 本指南中的步骤可能与其他版本一起运行，但尚未经过测试。

## <a name="feedback"></a>反馈

请在 GitHub 存储库中提供有关本教程 [的任何反馈](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp)。
