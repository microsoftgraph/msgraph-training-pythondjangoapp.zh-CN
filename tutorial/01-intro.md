<!-- markdownlint-disable MD002 MD041 -->

本教程向您介绍如何构建使用 Microsoft Graph API 检索用户的日历信息的 Python Django web 应用程序。

> [!TIP]
> 如果您只想下载已完成的教程，可以通过两种方式下载它。
>
> - 下载[Python 快速入门](https://developer.microsoft.com/graph/quick-start?platform=option-Python)，以分钟为单位获取工作代码。
> - 下载或克隆[GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp)。

## <a name="prerequisites"></a>先决条件

在开始本教程之前，您应在开发计算机上安装[Python](https://www.python.org/) （带有[pip](https://pypi.org/project/pip/)）。 如果您没有 Python，请访问 "下载选项" 的上一个链接。

您还应具有一个个人的 Microsoft 帐户，其中包含 Outlook.com 上的邮箱，或 Microsoft 工作或学校帐户。 如果你没有 Microsoft 帐户，可以使用以下几种方法获取免费帐户：

- 你可以[注册新的个人 Microsoft 帐户](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。
- 你可以[注册 office 365 开发人员计划](https://developer.microsoft.com/office/dev-program)以获取免费的 office 365 订阅。

> [!NOTE]
> 本教程是使用 Python version 3.8.2 和 Django version 3.0.4 编写的。 本指南中的步骤可能适用于其他版本，但尚未经过测试。

## <a name="feedback"></a>反馈

请在[GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp)中提供有关本教程的任何反馈。
