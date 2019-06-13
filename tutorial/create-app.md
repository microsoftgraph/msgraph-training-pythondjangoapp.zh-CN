<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="bab33-101">在本练习中, 将使用[Django](https://www.djangoproject.com/)生成 web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="bab33-101">In this exercise you will use [Django](https://www.djangoproject.com/) to build a web app.</span></span> <span data-ttu-id="bab33-102">如果尚未安装 Django, 则可以使用以下命令从命令行界面 (CLI) 安装它。</span><span class="sxs-lookup"><span data-stu-id="bab33-102">If you don't already have Django installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

```Shell
pip install Django=2.2.2
```

<span data-ttu-id="bab33-103">打开您的 CLI, 导航到您有权创建文件的目录, 并运行以下命令以创建新的 Django 应用程序。</span><span class="sxs-lookup"><span data-stu-id="bab33-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Django app.</span></span>

```Shell
django-admin startproject graph_tutorial
```

<span data-ttu-id="bab33-104">Django 创建一个名`graph_tutorial`为 Django web 应用程序的新目录, 并搭建基架。</span><span class="sxs-lookup"><span data-stu-id="bab33-104">Django creates a new directory called `graph_tutorial` and scaffolds a Django web app.</span></span> <span data-ttu-id="bab33-105">导航到此新目录, 然后输入以下命令以启动本地 web 服务器。</span><span class="sxs-lookup"><span data-stu-id="bab33-105">Navigate to this new directory and enter the following command to start a local web server.</span></span>

```Shell
python manage.py runserver
```

<span data-ttu-id="bab33-106">打开浏览器，并导航到 `http://localhost:8000`。</span><span class="sxs-lookup"><span data-stu-id="bab33-106">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="bab33-107">如果一切正常, 你将看到一个 Django 欢迎页面。</span><span class="sxs-lookup"><span data-stu-id="bab33-107">If everything is working, you will see a Django welcome page.</span></span> <span data-ttu-id="bab33-108">如果看不到此, 请查看[Django 入门指南](https://www.djangoproject.com/start/)。</span><span class="sxs-lookup"><span data-stu-id="bab33-108">If you don't see that, check the [Django getting started guide](https://www.djangoproject.com/start/).</span></span>

<span data-ttu-id="bab33-109">现在您已经验证了项目, 请将应用程序添加到项目中。</span><span class="sxs-lookup"><span data-stu-id="bab33-109">Now that you've verified the project, add an app to the project.</span></span> <span data-ttu-id="bab33-110">在 CLI 中运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="bab33-110">Run the following command in your CLI.</span></span>

```Shell
python manage.py startapp tutorial
```

<span data-ttu-id="bab33-111">这将`./tutorial`在目录中创建一个新的应用程序。</span><span class="sxs-lookup"><span data-stu-id="bab33-111">This creates a new app in the `./tutorial` directory.</span></span> <span data-ttu-id="bab33-112">打开`./graph_tutorial/settings.py` , 并将新`tutorial`应用程序添加到`INSTALLED_APPS`设置。</span><span class="sxs-lookup"><span data-stu-id="bab33-112">Open the `./graph_tutorial/settings.py` and add the new `tutorial` app to the `INSTALLED_APPS` setting.</span></span>

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

<span data-ttu-id="bab33-113">在 CLI 中, 运行以下命令来初始化项目的数据库。</span><span class="sxs-lookup"><span data-stu-id="bab33-113">In your CLI, run the following command to initialize the database for the project.</span></span>

```Shell
python manage.py migrate
```

<span data-ttu-id="bab33-114">为`tutorial`应用程序添加[URLconf](https://docs.djangoproject.com/en/2.1/topics/http/urls/) 。</span><span class="sxs-lookup"><span data-stu-id="bab33-114">Add a [URLconf](https://docs.djangoproject.com/en/2.1/topics/http/urls/) for the `tutorial` app.</span></span> <span data-ttu-id="bab33-115">在名为`./tutorial` `urls.py`的目录中创建一个新文件, 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="bab33-115">Create a new file in the `./tutorial` directory named `urls.py` and add the following code.</span></span>

```python
from django.urls import path

from . import views

urlpatterns = [
  # /tutorial
  path('', views.home, name='home'),
]
```

<span data-ttu-id="bab33-116">现在更新项目 URLconf 以导入此项目。</span><span class="sxs-lookup"><span data-stu-id="bab33-116">Now update the project URLconf to import this one.</span></span> <span data-ttu-id="bab33-117">打开`./graph_tutorial/urls.py`文件, 并将内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="bab33-117">Open the `./graph_tutorial/urls.py` file and replace the contents with the following.</span></span>

```python
from django.contrib import admin
from django.urls import path, include
from tutorial import views

urlpatterns = [
    path('tutorial/', include('tutorial.urls')),
    path('admin/', admin.site.urls),
]
```

<span data-ttu-id="bab33-118">最后将临时视图添加到`tutorials`应用程序以验证 URL 路由是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="bab33-118">Finally add a temporary view to the `tutorials` app to verify that URL routing is working.</span></span> <span data-ttu-id="bab33-119">打开 `./tutorial/views.py` 文件并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="bab33-119">Open the `./tutorial/views.py` file and add the following code.</span></span>

```python
from django.http import HttpResponse, HttpResponseRedirect

def home(request):
  # Temporary!
  return HttpResponse("Welcome to the tutorial.")
```

<span data-ttu-id="bab33-120">保存所有更改, 然后重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="bab33-120">Save all of your changes and restart the server.</span></span> <span data-ttu-id="bab33-121">浏览到`http://localhost:8000/tutorial`。</span><span class="sxs-lookup"><span data-stu-id="bab33-121">Browse to `http://localhost:8000/tutorial`.</span></span> <span data-ttu-id="bab33-122">您应该会看到`Welcome to the tutorial.`</span><span class="sxs-lookup"><span data-stu-id="bab33-122">You should see `Welcome to the tutorial.`</span></span>

<span data-ttu-id="bab33-123">在继续操作之前, 请先安装其他一些库, 稍后将使用这些库:</span><span class="sxs-lookup"><span data-stu-id="bab33-123">Before moving on, install some additional libraries that you will use later:</span></span>

- <span data-ttu-id="bab33-124">[请求-flask-oauthlib:](https://requests-oauthlib.readthedocs.io/en/latest/)用于处理登录和 OAuth 令牌流以及调用 Microsoft Graph 的 OAuth。</span><span class="sxs-lookup"><span data-stu-id="bab33-124">[Requests-OAuthlib: OAuth for Humans](https://requests-oauthlib.readthedocs.io/en/latest/) for handling sign-in and OAuth token flows, and for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="bab33-125">用于从 YAML 文件加载配置的[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) 。</span><span class="sxs-lookup"><span data-stu-id="bab33-125">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) for loading configuration from a YAML file.</span></span>
- <span data-ttu-id="bab33-126">[python-dateutil](https://pypi.org/project/python-dateutil/)用于分析从 Microsoft Graph 返回的 ISO 8601 日期字符串。</span><span class="sxs-lookup"><span data-stu-id="bab33-126">[python-dateutil](https://pypi.org/project/python-dateutil/) for parsing ISO 8601 date strings returned from Microsoft Graph.</span></span>

<span data-ttu-id="bab33-127">在 CLI 中运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="bab33-127">Run the following command in your CLI.</span></span>

```Shell
pip install requests_oauthlib==1.2.0
pip install pyyaml==5.1
pip install python-dateutil==2.8.0
```

## <a name="design-the-app"></a><span data-ttu-id="bab33-128">设计应用程序</span><span class="sxs-lookup"><span data-stu-id="bab33-128">Design the app</span></span>

<span data-ttu-id="bab33-129">首先创建模板目录, 并为应用程序定义全局布局。</span><span class="sxs-lookup"><span data-stu-id="bab33-129">Start by creating a templates directory and defining a global layout for the app.</span></span> <span data-ttu-id="bab33-130">在名为`./tutorial` `templates`的目录中创建一个新目录。</span><span class="sxs-lookup"><span data-stu-id="bab33-130">Create a new directory in the `./tutorial` directory named `templates`.</span></span> <span data-ttu-id="bab33-131">在`templates`目录中, 创建一个名为`tutorial`的新目录。</span><span class="sxs-lookup"><span data-stu-id="bab33-131">In the `templates` directory, create a new directory named `tutorial`.</span></span> <span data-ttu-id="bab33-132">最后, 在此目录中创建一个名为`layout.html`的新文件。</span><span class="sxs-lookup"><span data-stu-id="bab33-132">Finally, create a new file in this directory named `layout.html`.</span></span> <span data-ttu-id="bab33-133">项目的根目录的相对路径应为`./tutorial/templates/tutorial/layout.html`。</span><span class="sxs-lookup"><span data-stu-id="bab33-133">The relative path from the root of your project should be `./tutorial/templates/tutorial/layout.html`.</span></span> <span data-ttu-id="bab33-134">将以下代码添加到该文件中。</span><span class="sxs-lookup"><span data-stu-id="bab33-134">Add the following code in that file.</span></span>

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

<span data-ttu-id="bab33-135">此代码添加简单样式的[引导](http://getbootstrap.com/), 并添加一些简单图标的[字体](https://fontawesome.com/)。</span><span class="sxs-lookup"><span data-stu-id="bab33-135">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="bab33-136">它还定义具有导航栏的全局布局。</span><span class="sxs-lookup"><span data-stu-id="bab33-136">It also defines a global layout with a nav bar.</span></span>

<span data-ttu-id="bab33-137">现在, `./tutorial`在名为`static`的目录中创建一个新目录。</span><span class="sxs-lookup"><span data-stu-id="bab33-137">Now create a new directory in the `./tutorial` directory named `static`.</span></span> <span data-ttu-id="bab33-138">在`static`目录中, 创建一个名为`tutorial`的新目录。</span><span class="sxs-lookup"><span data-stu-id="bab33-138">In the `static` directory, create a new directory named `tutorial`.</span></span> <span data-ttu-id="bab33-139">最后, 在此目录中创建一个名为`app.css`的新文件。</span><span class="sxs-lookup"><span data-stu-id="bab33-139">Finally, create a new file in this directory named `app.css`.</span></span> <span data-ttu-id="bab33-140">项目的根目录的相对路径应为`./tutorial/static/tutorial/app.css`。</span><span class="sxs-lookup"><span data-stu-id="bab33-140">The relative path from the root of your project should be `./tutorial/static/tutorial/app.css`.</span></span> <span data-ttu-id="bab33-141">将以下代码添加到该文件中。</span><span class="sxs-lookup"><span data-stu-id="bab33-141">Add the following code in that file.</span></span>

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

<span data-ttu-id="bab33-142">接下来, 为使用该布局的主页创建一个模板。</span><span class="sxs-lookup"><span data-stu-id="bab33-142">Next, create a template for the home page that uses the layout.</span></span> <span data-ttu-id="bab33-143">在名为`./tutorial/templates/tutorial` `home.html`的目录中创建一个新文件, 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="bab33-143">Create a new file in the `./tutorial/templates/tutorial` directory named `home.html` and add the following code.</span></span>

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

<span data-ttu-id="bab33-144">更新`home`视图以使用此模板。</span><span class="sxs-lookup"><span data-stu-id="bab33-144">Update the `home` view to use this template.</span></span> <span data-ttu-id="bab33-145">打开`./tutorial/views.py`文件并添加以下新函数。</span><span class="sxs-lookup"><span data-stu-id="bab33-145">Open the `./tutorial/views.py` file and add the following new function.</span></span>

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

<span data-ttu-id="bab33-146">然后, 将现有`home`视图替换为以下项。</span><span class="sxs-lookup"><span data-stu-id="bab33-146">Then replace the existing `home` view with the following.</span></span>

```python
def home(request):
  context = initialize_context(request)

  return render(request, 'tutorial/home.html', context)
```

<span data-ttu-id="bab33-147">保存所有更改, 然后重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="bab33-147">Save all of your changes and restart the server.</span></span> <span data-ttu-id="bab33-148">现在, 应用程序看起来应非常不同。</span><span class="sxs-lookup"><span data-stu-id="bab33-148">Now, the app should look very different.</span></span>

![重新设计的主页的屏幕截图](./images/create-app-01.png)
