<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将使用 [Django](https://www.djangoproject.com/) 生成 Web 应用。

1. 如果尚未安装 Django，可以通过以下命令从命令行界面 (CLI) 安装它。

    ```Shell
    pip install --user Django==3.1.4
    ```

1. 打开 CLI，导航到你拥有创建文件权限的目录，然后运行以下命令以创建新的 Django 应用。

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. 导航到 **graph_tutorial** 目录并输入以下命令以启动本地 Web 服务器。

    ```Shell
    python manage.py runserver
    ```

1. 打开浏览器，并导航到 `http://localhost:8000`。 如果一切正常，你将看到 Django 欢迎页面。 如果看不到，请查看 [Django 入门指南](https://www.djangoproject.com/start/)。

1. 向项目添加应用。 在 CLI 中运行以下命令。

    ```Shell
    python manage.py startapp tutorial
    ```

1. 打开 **./graph_tutorial/settings.py，** 将新 `tutorial` 应用添加到 `INSTALLED_APPS` 设置中。

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. 在 CLI 中，运行以下命令来初始化项目的数据库。

    ```Shell
    python manage.py migrate
    ```

1. 为应用[添加 URLconf。](https://docs.djangoproject.com/en/3.0/topics/http/urls/) `tutorial` 在 **./tutorial** 目录中创建一个名为的新文件， `urls.py` 并添加以下代码。

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

1. 更新项目 URLconf 以导入此项目。 打开 **./graph_tutorial/urls.py，** 将内容替换为以下内容。

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. 向应用添加临时 `tutorials` 视图，以验证 URL 路由是否正常工作。 打开 **./tutorial/views.py，** 并将其全部内容替换为以下代码。

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

1. 保存所有更改并重新启动服务器。 浏览到 `http://localhost:8000` 。 你应该会看到 `Welcome to the tutorial.`

## <a name="install-libraries"></a>安装库

在继续之前，请安装一些稍后将使用的其他库：

- [Microsoft 身份验证库 (适用于 Python) MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-python) 库，用于处理登录和 OAuth 令牌流。
- [请求：用于调用](https://requests.readthedocs.io/en/master/) Microsoft Graph 的 HTTP for 用户。
- 用于从 YAML 文件加载配置的[PyYAML。](https://pyyaml.org/wiki/PyYAMLDocumentation)
- [python-dateutil](https://pypi.org/project/python-dateutil/) 用于分析从 Microsoft Graph 返回的 ISO 8601 日期字符串。

1. 在 CLI 中运行以下命令。

    ```Shell
    pip install msal==1.7.0
    pip install requests==2.25.0
    pip install pyyaml==5.3.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a>设计应用

1. 在名为 **./tutorial** 的目录中创建新目录 `templates` 。

1. 在 **./tutorial/templates** 目录中，创建一个名为 的新目录 `tutorial` 。

1. 在 **./tutorial/templates/tutorial** 目录中，创建一个名为 ./tutorial 的新文件 `layout.html` 。 在该文件中添加以下代码。

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    此代码为[简单样式设置添加 Bootstrap，](http://getbootstrap.com/)为一些简单图标添加[Fabric Core。](https://developer.microsoft.com/fluentui#/get-started#fabric-core) 它还定义具有导航栏的全局布局。

1. 在名为 **./tutorial** 的目录中创建新目录 `static` 。

1. 在 **./tutorial/static** 目录中，新建一个名为 `tutorial` .

1. 在 **./tutorial/static/tutorial** 目录中，创建一个名为 ./tutorial 的新文件 `app.css` 。 在该文件中添加以下代码。

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. 为使用布局的主页创建模板。 在 **./tutorial/templates/tutorial** 目录中创建一个名为的新文件 `home.html` ，并添加以下代码。

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. 打开 `./tutorial/views.py` 文件并添加以下新函数。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. 将现有 `home` 视图替换为以下内容。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. 在 **./tutorial/static/tutorial** **no-profile-photo.png** 中添加名为no-profile-photo.png的 PNG 文件。

1. 保存所有更改并重新启动服务器。 现在，应用看起来应该非常不同。

    ![重新设计主页的屏幕截图](./images/create-app-01.png)
