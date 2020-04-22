<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="46cda-101">本教程向您介绍如何构建使用 Microsoft Graph API 检索用户的日历信息的 Python Django web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="46cda-101">This tutorial teaches you how to build a Python Django web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="46cda-102">如果您只想下载已完成的教程，可以通过两种方式下载它。</span><span class="sxs-lookup"><span data-stu-id="46cda-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="46cda-103">下载[Python 快速入门](https://developer.microsoft.com/graph/quick-start?platform=option-Python)，以分钟为单位获取工作代码。</span><span class="sxs-lookup"><span data-stu-id="46cda-103">Download the [Python quick start](https://developer.microsoft.com/graph/quick-start?platform=option-Python) to get working code in minutes.</span></span>
> - <span data-ttu-id="46cda-104">下载或克隆[GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp)。</span><span class="sxs-lookup"><span data-stu-id="46cda-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="46cda-105">先决条件</span><span class="sxs-lookup"><span data-stu-id="46cda-105">Prerequisites</span></span>

<span data-ttu-id="46cda-106">在开始本教程之前，您应在开发计算机上安装[Python](https://www.python.org/) （带有[pip](https://pypi.org/project/pip/)）。</span><span class="sxs-lookup"><span data-stu-id="46cda-106">Before you start this tutorial, you should have [Python](https://www.python.org/) (with [pip](https://pypi.org/project/pip/)) installed on your development machine.</span></span> <span data-ttu-id="46cda-107">如果您没有 Python，请访问 "下载选项" 的上一个链接。</span><span class="sxs-lookup"><span data-stu-id="46cda-107">If you do not have Python, visit the previous link for download options.</span></span>

<span data-ttu-id="46cda-108">您还应具有一个个人的 Microsoft 帐户，其中包含 Outlook.com 上的邮箱，或 Microsoft 工作或学校帐户。</span><span class="sxs-lookup"><span data-stu-id="46cda-108">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="46cda-109">如果你没有 Microsoft 帐户，可以使用以下几种方法获取免费帐户：</span><span class="sxs-lookup"><span data-stu-id="46cda-109">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="46cda-110">你可以[注册新的个人 Microsoft 帐户](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。</span><span class="sxs-lookup"><span data-stu-id="46cda-110">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="46cda-111">你可以[注册 office 365 开发人员计划](https://developer.microsoft.com/office/dev-program)以获取免费的 office 365 订阅。</span><span class="sxs-lookup"><span data-stu-id="46cda-111">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="46cda-112">本教程是使用 Python version 3.8.2 和 Django version 3.0.4 编写的。</span><span class="sxs-lookup"><span data-stu-id="46cda-112">This tutorial was written with Python version 3.8.2 and Django version 3.0.4.</span></span> <span data-ttu-id="46cda-113">本指南中的步骤可能适用于其他版本，但尚未经过测试。</span><span class="sxs-lookup"><span data-stu-id="46cda-113">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="46cda-114">反馈</span><span class="sxs-lookup"><span data-stu-id="46cda-114">Feedback</span></span>

<span data-ttu-id="46cda-115">请在[GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp)中提供有关本教程的任何反馈。</span><span class="sxs-lookup"><span data-stu-id="46cda-115">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp).</span></span>
