<!-- markdownlint-disable MD002 MD041 -->

在本练习中, 你将扩展上一练习中的应用程序, 以支持 Azure AD 的身份验证。 若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph, 这是必需的。 在此步骤中, 将[flask-oauthlib](https://requests-oauthlib.readthedocs.io/en/latest/)库集成到应用程序中。

在名为`oauth_settings.yml`的项目的根目录中创建一个新文件, 并添加以下内容。

```text
app_id: YOUR_APP_ID_HERE
app_secret: YOUR_APP_PASSWORD_HERE
redirect: http://localhost:8000/tutorial/callback
scopes: openid profile offline_access user.read calendars.read
authority: https://login.microsoftonline.com/common
authorize_endpoint: /oauth2/v2.0/authorize
token_endpoint: /oauth2/v2.0/token
```

将`YOUR_APP_ID_HERE`替换为应用程序注册门户中的应用程序 ID, `YOUR_APP_SECRET_HERE`并将替换为您生成的密码。

> [!IMPORTANT]
> 如果您使用的是源代码管理 (如 git), 现在可以从源代码管理中排除`oauth_settings.yml`该文件, 以避免无意中泄漏您的应用程序 ID 和密码。

## <a name="implement-sign-in"></a>实施登录

在名为`./tutorial` `auth_helper.py`的目录中创建一个新文件, 并添加以下代码。

```python
import yaml
from requests_oauthlib import OAuth2Session
import os
import time

# This is necessary for testing with non-HTTPS localhost
# Remove this if deploying to production
os.environ['OAUTHLIB_INSECURE_TRANSPORT'] = '1'

# This is necessary because Azure does not guarantee
# to return scopes in the same case and order as requested
os.environ['OAUTHLIB_RELAX_TOKEN_SCOPE'] = '1'
os.environ['OAUTHLIB_IGNORE_SCOPE_CHANGE'] = '1'

# Load the oauth_settings.yml file
stream = open('oauth_settings.yml', 'r')
settings = yaml.load(stream, yaml.SafeLoader)
authorize_url = '{0}{1}'.format(settings['authority'], settings['authorize_endpoint'])
token_url = '{0}{1}'.format(settings['authority'], settings['token_endpoint'])

# Method to generate a sign-in url
def get_sign_in_url():
  # Initialize the OAuth client
  aad_auth = OAuth2Session(settings['app_id'],
    scope=settings['scopes'],
    redirect_uri=settings['redirect'])

  sign_in_url, state = aad_auth.authorization_url(authorize_url, prompt='login')

  return sign_in_url, state

# Method to exchange auth code for access token
def get_token_from_code(callback_url, expected_state):
  # Initialize the OAuth client
  aad_auth = OAuth2Session(settings['app_id'],
    state=expected_state,
    scope=settings['scopes'],
    redirect_uri=settings['redirect'])

  token = aad_auth.fetch_token(token_url,
    client_secret = settings['app_secret'],
    authorization_response=callback_url)

  return token
```

此文件将包含所有与身份验证相关的方法。 将`get_sign_in_url`生成一个授权 URL, 并且该`get_token_from_code`方法将交换访问令牌的授权响应。

将以下`import`语句添加到的顶部`./tutorial/views.py`。

```python
from django.urls import reverse
from tutorial.auth_helper import get_sign_in_url, get_token_from_code
```

现在, `./tutorial/views.py`在文件中添加两个新视图。

```python
def sign_in(request):
  # Get the sign-in URL
  sign_in_url, state = get_sign_in_url()
  # Save the expected state so we can validate in the callback
  request.session['auth_state'] = state
  # Redirect to the Azure sign-in page
  return HttpResponseRedirect(sign_in_url)

def callback(request):
  # Get the state saved in session
  expected_state = request.session.pop('auth_state', '')
  # Make the token request
  token = get_token_from_code(request.get_full_path(), expected_state)
  # Temporary! Save the response in an error so it's displayed
  request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(token) }
  return HttpResponseRedirect(reverse('home'))
```

这将定义两个新`signin`视图`callback`: 和。

该`signin`操作将生成 Azure AD 登录 URL, 保存 OAuth `state`客户端生成的值, 然后将浏览器重定向到 Azure AD 登录页。

`callback`操作是 Azure 在登录完成后重定向的位置。 该操作确保`state`值与保存的值相匹配, 然后使用 Azure 发送的授权代码来请求访问令牌。 然后, 它使用临时错误值中的访问令牌重定向回主页。 在继续操作之前, 你将使用此操作来验证我们的登录是否正常工作。 在测试之前, 您需要将视图添加到`./tutorial/urls.py`。

```python
path('signin', views.sign_in, name='signin'),
path('callback', views.callback, name='callback'),
```

将中`<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` `./tutorial/templates/tutorial/home.html`的行替换为以下代码行。

```html
<a href="{% url 'signin' %}" class="btn btn-primary btn-large">Click here to sign in</a>
```

将中`<a href="#" class="nav-link">Sign In</a>` `./tutorial/templates/tutorial/layout.html`的行替换为以下代码行。

```html
<a href="{% url 'signin' %}" class="nav-link">Sign In</a>
```

启动服务器并浏览到`https://localhost:8000/tutorial`。 单击 "登录" 按钮, 您应会被重定向`https://login.microsoftonline.com`到。 使用你的 Microsoft 帐户登录, 并同意请求的权限。 浏览器重定向到应用程序, 并显示令牌。

### <a name="get-user-details"></a>获取用户详细信息

在名为`./tutorial` `graph_helper.py`的目录中创建一个新文件, 并添加以下代码。

```python
from requests_oauthlib import OAuth2Session

graph_url = 'https://graph.microsoft.com/v1.0'

def get_user(token):
  graph_client = OAuth2Session(token=token)
  # Send GET to /me
  user = graph_client.get('{0}/me'.format(graph_url))
  # Return the JSON result
  return user.json()
```

该`get_user`方法通过使用您之前获取的访问令牌`/me` , 向 Microsoft GRAPH 终结点发出 get 请求, 以获取用户的配置文件。

更新中`callback` `./tutorial/views.py`的方法以从 Microsoft Graph 获取用户的配置文件。

首先, 将以下`import`语句添加到文件顶部。

```python
from tutorial.graph_helper import get_user
```

将`callback`方法替换为以下代码。

```python
def callback(request):
  # Get the state saved in session
  expected_state = request.session.pop('auth_state', '')
  # Make the token request
  token = get_token_from_code(request.get_full_path(), expected_state)

  # Get the user's profile
  user = get_user(token)
  # Temporary! Save the response in an error so it's displayed
  request.session['flash_error'] = { 'message': 'Token retrieved',
    'debug': 'User: {0}\nToken: {1}'.format(user, token) }
  return HttpResponseRedirect(reverse('home'))
```

新代码调用`get_user`方法来请求用户的配置文件。 它会将用户对象添加到临时输出中进行测试。

## <a name="storing-the-tokens"></a>存储令牌

现在, 您可以获取令牌, 现在是时候实现将它们存储在应用程序中的方法了。 由于这是一个示例应用程序, 因此为简单起见, 你将把它们存储在会话中。 实际应用程序将使用更可靠的安全存储解决方案, 就像数据库一样。

将以下新方法添加到`./tutorial/auth_helper.py`。

```python
def store_token(request, token):
  request.session['oauth_token'] = token

def store_user(request, user):
  request.session['user'] = {
    'is_authenticated': True,
    'name': user['displayName'],
    'email': user['mail'] if (user['mail'] != None) else user['userPrincipalName']
  }

def get_token(request):
  token = request.session['oauth_token']
  return token

def remove_user_and_token(request):
  if 'oauth_token' in request.session:
    del request.session['oauth_token']

  if 'user' in request.session:
    del request.session['user']
```

然后, 更新中`callback` `./tutorial/views.py`的函数以将标记存储在会话中, 并重定向回主页面。 将`from tutorial.auth_helper import get_sign_in_url, get_token_from_code`行替换为以下代码行。

```python
from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
```

将`callback`方法替换为以下方法。

```python
def callback(request):
  # Get the state saved in session
  expected_state = request.session.pop('auth_state', '')
  # Make the token request
  token = get_token_from_code(request.get_full_path(), expected_state)

  # Get the user's profile
  user = get_user(token)

  # Save token and user
  store_token(request, token)
  store_user(request, user)

  return HttpResponseRedirect(reverse('home'))
```

## <a name="implement-sign-out"></a>实现注销

在测试此新功能之前, 请添加一种注销方式。在中`./tutorial/views.py`添加`sign_out`新视图。

```python
def sign_out(request):
  # Clear out the user and token
  remove_user_and_token(request)

  return HttpResponseRedirect(reverse('home'))
```

现在将此视图添加`./tutorial/urls.py`到。

```python
path('signout', views.sign_out, name='signout'),
```

更新中**** `./tutorial/templates/tutorial/layout.html`的注销链接以使用此新视图。 将`<a href="#" class="dropdown-item">Sign Out</a>`行替换为以下代码行。

```html
<a href="{% url 'signout' %}" class="dropdown-item">Sign Out</a>
```

重新启动服务器并完成登录过程。 您应该最后返回到主页, 但 UI 应更改以指示您已登录。

![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

单击右上角的用户头像以访问 "**注销**" 链接。 单击 "**注销**" 重置会话并返回到主页。

![带有 "注销" 链接的下拉菜单的屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>刷新令牌

此时, 您的应用程序具有访问令牌, 该令牌是在 API `Authorization`调用的标头中发送的。 这是允许应用代表用户访问 Microsoft Graph 的令牌。

但是, 此令牌的生存期较短。 令牌在发出后会过期一小时。 这就是刷新令牌变得有用的地方。 刷新令牌允许应用在不要求用户重新登录的情况下请求新的访问令牌。 更新令牌管理代码以实现令牌刷新。

将中的`get_token`现有方法`./tutorial/auth_helper.py`替换为以下项。

```python
def get_token(request):
  token = request.session['oauth_token']
  if token != None:
    # Check expiration
    now = time.time()
    # Subtract 5 minutes from expiration to account for clock skew
    expire_time = token['expires_at'] - 300
    if now >= expire_time:
      # Refresh the token
      aad_auth = OAuth2Session(settings['app_id'],
        token = token,
        scope=settings['scopes'],
        redirect_uri=settings['redirect'])

      refresh_params = {
        'client_id': settings['app_id'],
        'client_secret': settings['app_secret'],
      }
      new_token = aad_auth.refresh_token(token_url, **refresh_params)

      # Save new token
      store_token(request, new_token)

      # Return new access token
      return new_token

    else:
      # Token still valid, just return it
      return token
```

此方法首先检查访问令牌是否已过期或接近即将过期。 如果是, 则它使用刷新令牌获取新令牌, 然后更新缓存并返回新的访问令牌。
