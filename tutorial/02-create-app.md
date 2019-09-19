<!-- markdownlint-disable MD002 MD041 -->

在本练习中，将使用[Django](https://www.djangoproject.com/)生成 web 应用程序。 如果尚未安装 Django，则可以使用以下命令从命令行界面（CLI）安装它。

```Shell
pip install Django=2.2.5
```

打开您的 CLI，导航到您有权创建文件的目录，并运行以下命令以创建新的 Django 应用程序。

```Shell
django-admin startproject graph_tutorial
```

Django 创建一个名`graph_tutorial`为 Django web 应用程序的新目录，并搭建基架。 导航到此新目录，然后输入以下命令以启动本地 web 服务器。

```Shell
python manage.py runserver
```

打开浏览器，并导航到 `http://localhost:8000`。 如果一切正常，你将看到一个 Django 欢迎页面。 如果看不到此，请查看[Django 入门指南](https://www.djangoproject.com/start/)。

现在您已经验证了项目，请将应用程序添加到项目中。 在 CLI 中运行以下命令。

```Shell
python manage.py startapp tutorial
```

这将`./tutorial`在目录中创建一个新的应用程序。 打开`./graph_tutorial/settings.py` ，并将新`tutorial`应用程序添加到`INSTALLED_APPS`设置。

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'tutorial',
]
```

在 CLI 中，运行以下命令来初始化项目的数据库。

```Shell
python manage.py migrate
```

为`tutorial`应用程序添加[URLconf](https://docs.djangoproject.com/en/2.1/topics/http/urls/) 。 在名为`./tutorial` `urls.py`的目录中创建一个新文件，并添加以下代码。

```python
from django.urls import path

from . import views

urlpatterns = [
  # /tutorial
  path('', views.home, name='home'),
]
```

现在更新项目 URLconf 以导入此项目。 打开`./graph_tutorial/urls.py`文件，并将内容替换为以下内容。

```python
from django.contrib import admin
from django.urls import path, include
from tutorial import views

urlpatterns = [
    path('tutorial/', include('tutorial.urls')),
    path('admin/', admin.site.urls),
]
```

最后将临时视图添加到`tutorials`应用程序以验证 URL 路由是否正常工作。 打开 `./tutorial/views.py` 文件并添加以下代码。

```python
from django.http import HttpResponse, HttpResponseRedirect

def home(request):
  # Temporary!
  return HttpResponse("Welcome to the tutorial.")
```

保存所有更改，然后重新启动服务器。 浏览到`http://localhost:8000/tutorial`。 您应该会看到`Welcome to the tutorial.`

在继续操作之前，请先安装其他一些库，稍后将使用这些库：

- [请求-flask-oauthlib：](https://requests-oauthlib.readthedocs.io/en/latest/)用于处理登录和 OAuth 令牌流以及调用 Microsoft Graph 的 OAuth。
- 用于从 YAML 文件加载配置的[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) 。
- [python-dateutil](https://pypi.org/project/python-dateutil/)用于分析从 Microsoft Graph 返回的 ISO 8601 日期字符串。

在 CLI 中运行以下命令。

```Shell
pip install requests_oauthlib==1.2.0
pip install pyyaml==5.1
pip install python-dateutil==2.8.0
```

## <a name="design-the-app"></a>设计应用程序

首先创建模板目录，并为应用程序定义全局布局。 在名为`./tutorial` `templates`的目录中创建一个新目录。 在`templates`目录中，创建一个名为`tutorial`的新目录。 最后，在此目录中创建一个名为`layout.html`的新文件。 项目的根目录的相对路径应为`./tutorial/templates/tutorial/layout.html`。 将以下代码添加到该文件中。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Python Graph Tutorial</title>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
      integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
      integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    {% load static %}
    <link rel="stylesheet" href="{% static "tutorial/app.css" %}">
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
      integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
      integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <a href="{% url 'home' %}" class="navbar-brand">Python Graph Tutorial</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
          aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <a href="{% url 'home' %}" class="nav-link{% if request.resolver_match.view_name == 'home' %} active{% endif %}">Home</a>
            </li>
            {% if user.is_authenticated %}
              <li class="nav-item" data-turbolinks="false">
                <a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="#">Calendar</a>
              </li>
            {% endif %}
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            {% if user.is_authenticated %}
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">
                  {% if user.avatar %}
                    <img src="{{ user.avatar }}" class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  {% else %}
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  {% endif %}
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0">{{ user.name }}</h5>
                  <p class="dropdown-item-text text-muted mb-0">{{ user.email }}</p>
                  <div class="dropdown-divider"></div>
                  <a href="#" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            {% else %}
              <li class="nav-item">
                <a href="#" class="nav-link">Sign In</a>
              </li>
            {% endif %}
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      {% if errors %}
        {% for error in errors %}
          <div class="alert alert-danger" role="alert">
            <p class="mb-3">{{ error.message }}</p>
            {% if error.debug %}
              <pre class="alert-pre border bg-light p-2"><code>{{ error.debug }}</code></pre>
            {% endif %}
          </div>
        {% endfor %}
      {% endif %}
      {% block content %}{% endblock %}
    </main>
  </body>
</html>
```

此代码添加简单样式的[引导](http://getbootstrap.com/)，并添加一些简单图标的[字体](https://fontawesome.com/)。 它还定义具有导航栏的全局布局。

现在， `./tutorial`在名为`static`的目录中创建一个新目录。 在`static`目录中，创建一个名为`tutorial`的新目录。 最后，在此目录中创建一个名为`app.css`的新文件。 项目的根目录的相对路径应为`./tutorial/static/tutorial/app.css`。 将以下代码添加到该文件中。

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

接下来，为使用该布局的主页创建一个模板。 在名为`./tutorial/templates/tutorial` `home.html`的目录中创建一个新文件，并添加以下代码。

```html
{% extends "tutorial/layout.html" %}
{% block content %}
<div class="jumbotron">
  <h1>Python Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from Python</p>
  {% if user.is_authenticated %}
    <h4>Welcome {{ user.name }}!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  {% else %}
    <a href="#" class="btn btn-primary btn-large">Click here to sign in</a>
  {% endif %}
</div>
{% endblock %}
```

更新`home`视图以使用此模板。 打开`./tutorial/views.py`文件并添加以下新函数。

```python
def initialize_context(request):
  context = {}

  # Check for any errors in the session
  error = request.session.pop('flash_error', None)

  if error != None:
    context['errors'] = []
    context['errors'].append(error)

  # Check for user in the session
  context['user'] = request.session.get('user', {'is_authenticated': False})
  return context
```

然后，将现有`home`视图替换为以下项。

```python
def home(request):
  context = initialize_context(request)

  return render(request, 'tutorial/home.html', context)
```

保存所有更改，然后重新启动服务器。 现在，应用程序看起来应非常不同。

![重新设计的主页的屏幕截图](./images/create-app-01.png)
