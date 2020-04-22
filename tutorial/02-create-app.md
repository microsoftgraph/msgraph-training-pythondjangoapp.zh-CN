<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6e010-101">在本练习中，将使用[Django](https://www.djangoproject.com/)生成 web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="6e010-101">In this exercise you will use [Django](https://www.djangoproject.com/) to build a web app.</span></span>

1. <span data-ttu-id="6e010-102">如果尚未安装 Django，则可以使用以下命令从命令行界面（CLI）安装它。</span><span class="sxs-lookup"><span data-stu-id="6e010-102">If you don't already have Django installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    pip install Django==3.0.4
    ```

1. <span data-ttu-id="6e010-103">打开您的 CLI，导航到您有权创建文件的目录，并运行以下命令以创建新的 Django 应用程序。</span><span class="sxs-lookup"><span data-stu-id="6e010-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Django app.</span></span>

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. <span data-ttu-id="6e010-104">导航到**graph_tutorial**目录，然后输入以下命令以启动本地 web 服务器。</span><span class="sxs-lookup"><span data-stu-id="6e010-104">Navigate to the **graph_tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    python manage.py runserver
    ```

1. <span data-ttu-id="6e010-105">打开浏览器，并导航到 `http://localhost:8000`。</span><span class="sxs-lookup"><span data-stu-id="6e010-105">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="6e010-106">如果一切正常，你将看到一个 Django 欢迎页面。</span><span class="sxs-lookup"><span data-stu-id="6e010-106">If everything is working, you will see a Django welcome page.</span></span> <span data-ttu-id="6e010-107">如果看不到此，请查看[Django 入门指南](https://www.djangoproject.com/start/)。</span><span class="sxs-lookup"><span data-stu-id="6e010-107">If you don't see that, check the [Django getting started guide](https://www.djangoproject.com/start/).</span></span>

1. <span data-ttu-id="6e010-108">将应用程序添加到项目中。</span><span class="sxs-lookup"><span data-stu-id="6e010-108">Add an app to the project.</span></span> <span data-ttu-id="6e010-109">在 CLI 中运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="6e010-109">Run the following command in your CLI.</span></span>

    ```Shell
    python manage.py startapp tutorial
    ```

1. <span data-ttu-id="6e010-110">打开 **。/graph_tutorial/settings.py** ，并将新`tutorial`应用程序添加`INSTALLED_APPS`到设置。</span><span class="sxs-lookup"><span data-stu-id="6e010-110">Open **./graph_tutorial/settings.py** and add the new `tutorial` app to the `INSTALLED_APPS` setting.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. <span data-ttu-id="6e010-111">在 CLI 中，运行以下命令来初始化项目的数据库。</span><span class="sxs-lookup"><span data-stu-id="6e010-111">In your CLI, run the following command to initialize the database for the project.</span></span>

    ```Shell
    python manage.py migrate
    ```

1. <span data-ttu-id="6e010-112">为`tutorial`应用程序添加[URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) 。</span><span class="sxs-lookup"><span data-stu-id="6e010-112">Add a [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) for the `tutorial` app.</span></span> <span data-ttu-id="6e010-113">在 **/tutorial**目录中创建一个名为`urls.py`的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="6e010-113">Create a new file in the **./tutorial** directory named `urls.py` and add the following code.</span></span>

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

1. <span data-ttu-id="6e010-114">更新项目 URLconf 以导入此项目。</span><span class="sxs-lookup"><span data-stu-id="6e010-114">Update the project URLconf to import this one.</span></span> <span data-ttu-id="6e010-115">打开 **。/graph_tutorial/urls.py** ，并将内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="6e010-115">Open **./graph_tutorial/urls.py** and replace the contents with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. <span data-ttu-id="6e010-116">将临时视图添加到`tutorials`应用程序以验证 URL 路由是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="6e010-116">Add a temporary view to the `tutorials` app to verify that URL routing is working.</span></span> <span data-ttu-id="6e010-117">打开 **/tutorial/views.py** ，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="6e010-117">Open **./tutorial/views.py** and add the following code.</span></span>

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. <span data-ttu-id="6e010-118">保存所有更改，然后重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="6e010-118">Save all of your changes and restart the server.</span></span> <span data-ttu-id="6e010-119">浏览到`http://localhost:8000`。</span><span class="sxs-lookup"><span data-stu-id="6e010-119">Browse to `http://localhost:8000`.</span></span> <span data-ttu-id="6e010-120">您应该会看到`Welcome to the tutorial.`</span><span class="sxs-lookup"><span data-stu-id="6e010-120">You should see `Welcome to the tutorial.`</span></span>

## <a name="install-libraries"></a><span data-ttu-id="6e010-121">安装库</span><span class="sxs-lookup"><span data-stu-id="6e010-121">Install libraries</span></span>

<span data-ttu-id="6e010-122">在继续操作之前，请先安装其他一些库，稍后将使用这些库：</span><span class="sxs-lookup"><span data-stu-id="6e010-122">Before moving on, install some additional libraries that you will use later:</span></span>

- <span data-ttu-id="6e010-123">[请求-flask-oauthlib：](https://requests-oauthlib.readthedocs.io/en/latest/)用于处理登录和 OAuth 令牌流以及调用 Microsoft Graph 的 OAuth。</span><span class="sxs-lookup"><span data-stu-id="6e010-123">[Requests-OAuthlib: OAuth for Humans](https://requests-oauthlib.readthedocs.io/en/latest/) for handling sign-in and OAuth token flows, and for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="6e010-124">用于从 YAML 文件加载配置的[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) 。</span><span class="sxs-lookup"><span data-stu-id="6e010-124">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) for loading configuration from a YAML file.</span></span>
- <span data-ttu-id="6e010-125">[python-dateutil](https://pypi.org/project/python-dateutil/)用于分析从 Microsoft Graph 返回的 ISO 8601 日期字符串。</span><span class="sxs-lookup"><span data-stu-id="6e010-125">[python-dateutil](https://pypi.org/project/python-dateutil/) for parsing ISO 8601 date strings returned from Microsoft Graph.</span></span>

1. <span data-ttu-id="6e010-126">在 CLI 中运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="6e010-126">Run the following command in your CLI.</span></span>

    ```Shell
    pip install requests_oauthlib==1.3.0
    pip install pyyaml==5.3.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a><span data-ttu-id="6e010-127">设计应用程序</span><span class="sxs-lookup"><span data-stu-id="6e010-127">Design the app</span></span>

1. <span data-ttu-id="6e010-128">在 **./tutorial**目录中创建一个名为`templates`的新目录。</span><span class="sxs-lookup"><span data-stu-id="6e010-128">Create a new directory in the **./tutorial** directory named `templates`.</span></span>

1. <span data-ttu-id="6e010-129">在 **./tutorial/templates**目录中，创建一个名为`tutorial`的新目录。</span><span class="sxs-lookup"><span data-stu-id="6e010-129">In the **./tutorial/templates** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="6e010-130">在 **./tutorial/templates/tutorial**目录中，创建一个名为`layout.html`的新文件。</span><span class="sxs-lookup"><span data-stu-id="6e010-130">In the **./tutorial/templates/tutorial** directory, create a new file named `layout.html`.</span></span> <span data-ttu-id="6e010-131">将以下代码添加到该文件中。</span><span class="sxs-lookup"><span data-stu-id="6e010-131">Add the following code in that file.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    <span data-ttu-id="6e010-132">此代码添加简单样式的[引导](http://getbootstrap.com/)，并添加一些简单图标的[字体](https://fontawesome.com/)。</span><span class="sxs-lookup"><span data-stu-id="6e010-132">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="6e010-133">它还定义具有导航栏的全局布局。</span><span class="sxs-lookup"><span data-stu-id="6e010-133">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="6e010-134">在 **./tutorial**目录中创建一个名为`static`的新目录。</span><span class="sxs-lookup"><span data-stu-id="6e010-134">Create a new directory in the **./tutorial** directory named `static`.</span></span>

1. <span data-ttu-id="6e010-135">在 **./tutorial/static**目录中，创建一个名为`tutorial`的新目录。</span><span class="sxs-lookup"><span data-stu-id="6e010-135">In the **./tutorial/static** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="6e010-136">在 **./tutorial/static/tutorial**目录中，创建一个名为`app.css`的新文件。</span><span class="sxs-lookup"><span data-stu-id="6e010-136">In the **./tutorial/static/tutorial** directory, create a new file named `app.css`.</span></span> <span data-ttu-id="6e010-137">将以下代码添加到该文件中。</span><span class="sxs-lookup"><span data-stu-id="6e010-137">Add the following code in that file.</span></span>

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. <span data-ttu-id="6e010-138">为使用该布局的主页创建模板。</span><span class="sxs-lookup"><span data-stu-id="6e010-138">Create a template for the home page that uses the layout.</span></span> <span data-ttu-id="6e010-139">在 **/tutorial/templates/tutorial**目录中创建一个名为`home.html`的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="6e010-139">Create a new file in the **./tutorial/templates/tutorial** directory named `home.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. <span data-ttu-id="6e010-140">打开`./tutorial/views.py`文件并添加以下新函数。</span><span class="sxs-lookup"><span data-stu-id="6e010-140">Open the `./tutorial/views.py` file and add the following new function.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. <span data-ttu-id="6e010-141">将现有`home`视图替换为以下项。</span><span class="sxs-lookup"><span data-stu-id="6e010-141">Replace the existing `home` view with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. <span data-ttu-id="6e010-142">保存所有更改，然后重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="6e010-142">Save all of your changes and restart the server.</span></span> <span data-ttu-id="6e010-143">现在，应用程序看起来应非常不同。</span><span class="sxs-lookup"><span data-stu-id="6e010-143">Now, the app should look very different.</span></span>

    ![重新设计的主页的屏幕截图](./images/create-app-01.png)
