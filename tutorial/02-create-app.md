<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="e87fb-101">在此练习中，你将使用 [Django](https://www.djangoproject.com/) 生成 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="e87fb-101">In this exercise you will use [Django](https://www.djangoproject.com/) to build a web app.</span></span>

1. <span data-ttu-id="e87fb-102">如果尚未安装 Django，可以通过以下命令从命令行界面 (CLI) 安装它。</span><span class="sxs-lookup"><span data-stu-id="e87fb-102">If you don't already have Django installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    pip install --user Django==3.1.4
    ```

1. <span data-ttu-id="e87fb-103">打开 CLI，导航到你拥有创建文件权限的目录，然后运行以下命令以创建新的 Django 应用。</span><span class="sxs-lookup"><span data-stu-id="e87fb-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Django app.</span></span>

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. <span data-ttu-id="e87fb-104">导航到 **graph_tutorial** 目录并输入以下命令以启动本地 Web 服务器。</span><span class="sxs-lookup"><span data-stu-id="e87fb-104">Navigate to the **graph_tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    python manage.py runserver
    ```

1. <span data-ttu-id="e87fb-105">打开浏览器，并导航到 `http://localhost:8000`。</span><span class="sxs-lookup"><span data-stu-id="e87fb-105">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="e87fb-106">如果一切正常，你将看到 Django 欢迎页面。</span><span class="sxs-lookup"><span data-stu-id="e87fb-106">If everything is working, you will see a Django welcome page.</span></span> <span data-ttu-id="e87fb-107">如果看不到，请查看 [Django 入门指南](https://www.djangoproject.com/start/)。</span><span class="sxs-lookup"><span data-stu-id="e87fb-107">If you don't see that, check the [Django getting started guide](https://www.djangoproject.com/start/).</span></span>

1. <span data-ttu-id="e87fb-108">向项目添加应用。</span><span class="sxs-lookup"><span data-stu-id="e87fb-108">Add an app to the project.</span></span> <span data-ttu-id="e87fb-109">在 CLI 中运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="e87fb-109">Run the following command in your CLI.</span></span>

    ```Shell
    python manage.py startapp tutorial
    ```

1. <span data-ttu-id="e87fb-110">打开 **./graph_tutorial/settings.py，** 将新 `tutorial` 应用添加到 `INSTALLED_APPS` 设置中。</span><span class="sxs-lookup"><span data-stu-id="e87fb-110">Open **./graph_tutorial/settings.py** and add the new `tutorial` app to the `INSTALLED_APPS` setting.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. <span data-ttu-id="e87fb-111">在 CLI 中，运行以下命令来初始化项目的数据库。</span><span class="sxs-lookup"><span data-stu-id="e87fb-111">In your CLI, run the following command to initialize the database for the project.</span></span>

    ```Shell
    python manage.py migrate
    ```

1. <span data-ttu-id="e87fb-112">为应用[添加 URLconf。](https://docs.djangoproject.com/en/3.0/topics/http/urls/) `tutorial`</span><span class="sxs-lookup"><span data-stu-id="e87fb-112">Add a [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) for the `tutorial` app.</span></span> <span data-ttu-id="e87fb-113">在 **./tutorial** 目录中创建一个名为的新文件， `urls.py` 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="e87fb-113">Create a new file in the **./tutorial** directory named `urls.py` and add the following code.</span></span>

    ```python
    from django.urls import path

    from . import views

    urlpatterns = [
      # /
      path('', views.home, name='home'),
      # TEMPORARY
      path('signin', views.home, name='signin'),
      path('signout', views.home, name='signout'),
      path('calendar', views.home, name='calendar'),
    ]
    ```

1. <span data-ttu-id="e87fb-114">更新项目 URLconf 以导入此项目。</span><span class="sxs-lookup"><span data-stu-id="e87fb-114">Update the project URLconf to import this one.</span></span> <span data-ttu-id="e87fb-115">打开 **./graph_tutorial/urls.py，** 将内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="e87fb-115">Open **./graph_tutorial/urls.py** and replace the contents with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. <span data-ttu-id="e87fb-116">向应用添加临时 `tutorials` 视图，以验证 URL 路由是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="e87fb-116">Add a temporary view to the `tutorials` app to verify that URL routing is working.</span></span> <span data-ttu-id="e87fb-117">打开 **./tutorial/views.py，** 并将其全部内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="e87fb-117">Open **./tutorial/views.py** and replace its entire contents with the following code.</span></span>

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect
    from django.urls import reverse
    from datetime import datetime, timedelta
    from dateutil import tz, parser

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. <span data-ttu-id="e87fb-118">保存所有更改并重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="e87fb-118">Save all of your changes and restart the server.</span></span> <span data-ttu-id="e87fb-119">浏览到 `http://localhost:8000` 。</span><span class="sxs-lookup"><span data-stu-id="e87fb-119">Browse to `http://localhost:8000`.</span></span> <span data-ttu-id="e87fb-120">你应该会看到 `Welcome to the tutorial.`</span><span class="sxs-lookup"><span data-stu-id="e87fb-120">You should see `Welcome to the tutorial.`</span></span>

## <a name="install-libraries"></a><span data-ttu-id="e87fb-121">安装库</span><span class="sxs-lookup"><span data-stu-id="e87fb-121">Install libraries</span></span>

<span data-ttu-id="e87fb-122">在继续之前，请安装一些稍后将使用的其他库：</span><span class="sxs-lookup"><span data-stu-id="e87fb-122">Before moving on, install some additional libraries that you will use later:</span></span>

- <span data-ttu-id="e87fb-123">[Microsoft 身份验证库 (适用于 Python) MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-python) 库，用于处理登录和 OAuth 令牌流。</span><span class="sxs-lookup"><span data-stu-id="e87fb-123">[Microsoft Authentication Library (MSAL) for Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="e87fb-124">[请求：用于调用](https://requests.readthedocs.io/en/master/) Microsoft Graph 的 HTTP for 用户。</span><span class="sxs-lookup"><span data-stu-id="e87fb-124">[Requests: HTTP for Humans](https://requests.readthedocs.io/en/master/) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="e87fb-125">用于从 YAML 文件加载配置的[PyYAML。](https://pyyaml.org/wiki/PyYAMLDocumentation)</span><span class="sxs-lookup"><span data-stu-id="e87fb-125">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) for loading configuration from a YAML file.</span></span>
- <span data-ttu-id="e87fb-126">[python-dateutil](https://pypi.org/project/python-dateutil/) 用于分析从 Microsoft Graph 返回的 ISO 8601 日期字符串。</span><span class="sxs-lookup"><span data-stu-id="e87fb-126">[python-dateutil](https://pypi.org/project/python-dateutil/) for parsing ISO 8601 date strings returned from Microsoft Graph.</span></span>

1. <span data-ttu-id="e87fb-127">在 CLI 中运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="e87fb-127">Run the following command in your CLI.</span></span>

    ```Shell
    pip install msal==1.7.0
    pip install requests==2.25.0
    pip install pyyaml==5.3.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a><span data-ttu-id="e87fb-128">设计应用</span><span class="sxs-lookup"><span data-stu-id="e87fb-128">Design the app</span></span>

1. <span data-ttu-id="e87fb-129">在名为 **./tutorial** 的目录中创建新目录 `templates` 。</span><span class="sxs-lookup"><span data-stu-id="e87fb-129">Create a new directory in the **./tutorial** directory named `templates`.</span></span>

1. <span data-ttu-id="e87fb-130">在 **./tutorial/templates** 目录中，创建一个名为 的新目录 `tutorial` 。</span><span class="sxs-lookup"><span data-stu-id="e87fb-130">In the **./tutorial/templates** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="e87fb-131">在 **./tutorial/templates/tutorial** 目录中，创建一个名为 ./tutorial 的新文件 `layout.html` 。</span><span class="sxs-lookup"><span data-stu-id="e87fb-131">In the **./tutorial/templates/tutorial** directory, create a new file named `layout.html`.</span></span> <span data-ttu-id="e87fb-132">在该文件中添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="e87fb-132">Add the following code in that file.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    <span data-ttu-id="e87fb-133">此代码为[简单样式设置添加 Bootstrap，](http://getbootstrap.com/)为一些简单图标添加[Fabric Core。](https://developer.microsoft.com/fluentui#/get-started#fabric-core)</span><span class="sxs-lookup"><span data-stu-id="e87fb-133">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) for some simple icons.</span></span> <span data-ttu-id="e87fb-134">它还定义具有导航栏的全局布局。</span><span class="sxs-lookup"><span data-stu-id="e87fb-134">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="e87fb-135">在名为 **./tutorial** 的目录中创建新目录 `static` 。</span><span class="sxs-lookup"><span data-stu-id="e87fb-135">Create a new directory in the **./tutorial** directory named `static`.</span></span>

1. <span data-ttu-id="e87fb-136">在 **./tutorial/static** 目录中，新建一个名为 `tutorial` .</span><span class="sxs-lookup"><span data-stu-id="e87fb-136">In the **./tutorial/static** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="e87fb-137">在 **./tutorial/static/tutorial** 目录中，创建一个名为 ./tutorial 的新文件 `app.css` 。</span><span class="sxs-lookup"><span data-stu-id="e87fb-137">In the **./tutorial/static/tutorial** directory, create a new file named `app.css`.</span></span> <span data-ttu-id="e87fb-138">在该文件中添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="e87fb-138">Add the following code in that file.</span></span>

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. <span data-ttu-id="e87fb-139">为使用布局的主页创建模板。</span><span class="sxs-lookup"><span data-stu-id="e87fb-139">Create a template for the home page that uses the layout.</span></span> <span data-ttu-id="e87fb-140">在 **./tutorial/templates/tutorial** 目录中创建一个名为的新文件 `home.html` ，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="e87fb-140">Create a new file in the **./tutorial/templates/tutorial** directory named `home.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. <span data-ttu-id="e87fb-141">打开 `./tutorial/views.py` 文件并添加以下新函数。</span><span class="sxs-lookup"><span data-stu-id="e87fb-141">Open the `./tutorial/views.py` file and add the following new function.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. <span data-ttu-id="e87fb-142">将现有 `home` 视图替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="e87fb-142">Replace the existing `home` view with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. <span data-ttu-id="e87fb-143">在 **./tutorial/static/tutorial** **no-profile-photo.png** 中添加名为no-profile-photo.png的 PNG 文件。</span><span class="sxs-lookup"><span data-stu-id="e87fb-143">Add a PNG file named **no-profile-photo.png** in the **./tutorial/static/tutorial** directory.</span></span>

1. <span data-ttu-id="e87fb-144">保存所有更改并重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="e87fb-144">Save all of your changes and restart the server.</span></span> <span data-ttu-id="e87fb-145">现在，应用看起来应该非常不同。</span><span class="sxs-lookup"><span data-stu-id="e87fb-145">Now, the app should look very different.</span></span>

    ![重新设计主页的屏幕截图](./images/create-app-01.png)
