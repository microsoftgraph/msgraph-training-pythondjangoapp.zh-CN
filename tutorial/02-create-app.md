<!-- markdownlint-disable MD002 MD041 -->

在本练习中，将使用[Django](https://www.djangoproject.com/)生成 web 应用程序。

1. 如果尚未安装 Django，则可以使用以下命令从命令行界面（CLI）安装它。

    ```Shell
    pip install Django==3.0.4
    ```

1. 打开您的 CLI，导航到您有权创建文件的目录，并运行以下命令以创建新的 Django 应用程序。

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. 导航到**graph_tutorial**目录，然后输入以下命令以启动本地 web 服务器。

    ```Shell
    python manage.py runserver
    ```

1. 打开浏览器，并导航到 `http://localhost:8000`。 如果一切正常，你将看到一个 Django 欢迎页面。 如果看不到此，请查看[Django 入门指南](https://www.djangoproject.com/start/)。

1. 将应用程序添加到项目中。 在 CLI 中运行以下命令。

    ```Shell
    python manage.py startapp tutorial
    ```

1. 打开 **。/graph_tutorial/settings.py** ，并将新`tutorial`应用程序添加`INSTALLED_APPS`到设置。

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. 在 CLI 中，运行以下命令来初始化项目的数据库。

    ```Shell
    python manage.py migrate
    ```

1. 为`tutorial`应用程序添加[URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) 。 在 **/tutorial**目录中创建一个名为`urls.py`的新文件，并添加以下代码。

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

1. 更新项目 URLconf 以导入此项目。 打开 **。/graph_tutorial/urls.py** ，并将内容替换为以下内容。

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. 将临时视图添加到`tutorials`应用程序以验证 URL 路由是否正常工作。 打开 **/tutorial/views.py** ，并添加以下代码。

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. 保存所有更改，然后重新启动服务器。 浏览到`http://localhost:8000`。 您应该会看到`Welcome to the tutorial.`

## <a name="install-libraries"></a>安装库

在继续操作之前，请先安装其他一些库，稍后将使用这些库：

- [请求-flask-oauthlib：](https://requests-oauthlib.readthedocs.io/en/latest/)用于处理登录和 OAuth 令牌流以及调用 Microsoft Graph 的 OAuth。
- 用于从 YAML 文件加载配置的[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) 。
- [python-dateutil](https://pypi.org/project/python-dateutil/)用于分析从 Microsoft Graph 返回的 ISO 8601 日期字符串。

1. 在 CLI 中运行以下命令。

    ```Shell
    pip install requests_oauthlib==1.3.0
    pip install pyyaml==5.3.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a>设计应用程序

1. 在 **./tutorial**目录中创建一个名为`templates`的新目录。

1. 在 **./tutorial/templates**目录中，创建一个名为`tutorial`的新目录。

1. 在 **./tutorial/templates/tutorial**目录中，创建一个名为`layout.html`的新文件。 将以下代码添加到该文件中。

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    此代码添加简单样式的[引导](http://getbootstrap.com/)，并添加一些简单图标的[字体](https://fontawesome.com/)。 它还定义具有导航栏的全局布局。

1. 在 **./tutorial**目录中创建一个名为`static`的新目录。

1. 在 **./tutorial/static**目录中，创建一个名为`tutorial`的新目录。

1. 在 **./tutorial/static/tutorial**目录中，创建一个名为`app.css`的新文件。 将以下代码添加到该文件中。

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. 为使用该布局的主页创建模板。 在 **/tutorial/templates/tutorial**目录中创建一个名为`home.html`的新文件，并添加以下代码。

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. 打开`./tutorial/views.py`文件并添加以下新函数。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. 将现有`home`视图替换为以下项。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. 保存所有更改，然后重新启动服务器。 现在，应用程序看起来应非常不同。

    ![重新设计的主页的屏幕截图](./images/create-app-01.png)
