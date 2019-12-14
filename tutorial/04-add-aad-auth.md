<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="5f170-101">在本练习中，你将扩展上一练习中的应用程序，以支持 Azure AD 的身份验证。</span><span class="sxs-lookup"><span data-stu-id="5f170-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="5f170-102">若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph，这是必需的。</span><span class="sxs-lookup"><span data-stu-id="5f170-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="5f170-103">在此步骤中，将[flask-oauthlib](https://requests-oauthlib.readthedocs.io/en/latest/)库集成到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="5f170-103">In this step you will integrate the [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) library into the application.</span></span>

<span data-ttu-id="5f170-104">在名为`oauth_settings.yml`的项目的根目录中创建一个新文件，并添加以下内容。</span><span class="sxs-lookup"><span data-stu-id="5f170-104">Create a new file in the root of the project named `oauth_settings.yml`, and add the following content.</span></span>

```text
app_id: "YOUR_APP_ID_HERE"
app_secret: "YOUR_APP_PASSWORD_HERE"
redirect: "http://localhost:8000/tutorial/callback"
scopes: "openid profile offline_access user.read calendars.read"
authority: "https://login.microsoftonline.com/common"
authorize_endpoint: "/oauth2/v2.0/authorize"
token_endpoint: "/oauth2/v2.0/token"
```

<span data-ttu-id="5f170-105">将`YOUR_APP_ID_HERE`替换为应用程序注册门户中的应用程序 ID， `YOUR_APP_SECRET_HERE`并将替换为您生成的密码。</span><span class="sxs-lookup"><span data-stu-id="5f170-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="5f170-106">如果您使用的是源代码管理（如 git），现在可以从源代码管理中排除`oauth_settings.yml`该文件，以避免无意中泄漏您的应用程序 ID 和密码。</span><span class="sxs-lookup"><span data-stu-id="5f170-106">If you're using source control such as git, now would be a good time to exclude the `oauth_settings.yml` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="5f170-107">实施登录</span><span class="sxs-lookup"><span data-stu-id="5f170-107">Implement sign-in</span></span>

<span data-ttu-id="5f170-108">在名为`./tutorial` `auth_helper.py`的目录中创建一个新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="5f170-108">Create a new file in the `./tutorial` directory named `auth_helper.py` and add the following code.</span></span>

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

<span data-ttu-id="5f170-109">此文件将包含所有与身份验证相关的方法。</span><span class="sxs-lookup"><span data-stu-id="5f170-109">This file will hold all of your authentication-related methods.</span></span> <span data-ttu-id="5f170-110">将`get_sign_in_url`生成一个授权 URL，并且该`get_token_from_code`方法将交换访问令牌的授权响应。</span><span class="sxs-lookup"><span data-stu-id="5f170-110">The `get_sign_in_url` generates an authorization URL, and the `get_token_from_code` method exchanges the authorization response for an access token.</span></span>

<span data-ttu-id="5f170-111">将以下`import`语句添加到的顶部`./tutorial/views.py`。</span><span class="sxs-lookup"><span data-stu-id="5f170-111">Add the following `import` statements to the top of `./tutorial/views.py`.</span></span>

```python
from django.urls import reverse
from tutorial.auth_helper import get_sign_in_url, get_token_from_code
```

<span data-ttu-id="5f170-112">现在， `./tutorial/views.py`在文件中添加两个新视图。</span><span class="sxs-lookup"><span data-stu-id="5f170-112">Now add a couple of new views in the `./tutorial/views.py` file.</span></span>

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

<span data-ttu-id="5f170-113">这将定义两个新`signin`视图`callback`：和。</span><span class="sxs-lookup"><span data-stu-id="5f170-113">This defines two new views: `signin` and `callback`.</span></span>

<span data-ttu-id="5f170-114">该`signin`操作将生成 Azure AD 登录 URL，保存 OAuth `state`客户端生成的值，然后将浏览器重定向到 Azure AD 登录页。</span><span class="sxs-lookup"><span data-stu-id="5f170-114">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

<span data-ttu-id="5f170-115">`callback`操作是 Azure 在登录完成后重定向的位置。</span><span class="sxs-lookup"><span data-stu-id="5f170-115">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="5f170-116">该操作确保`state`值与保存的值相匹配，然后使用 Azure 发送的授权代码来请求访问令牌。</span><span class="sxs-lookup"><span data-stu-id="5f170-116">That action makes sure the `state` value matches the saved value, then uses the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="5f170-117">然后，它使用临时错误值中的访问令牌重定向回主页。</span><span class="sxs-lookup"><span data-stu-id="5f170-117">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="5f170-118">在继续操作之前，你将使用此操作来验证我们的登录是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="5f170-118">You'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="5f170-119">在测试之前，您需要将视图添加到`./tutorial/urls.py`。</span><span class="sxs-lookup"><span data-stu-id="5f170-119">Before you test, you need to add the views to `./tutorial/urls.py`.</span></span>

```python
path('signin', views.sign_in, name='signin'),
path('callback', views.callback, name='callback'),
```

<span data-ttu-id="5f170-120">将中`<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` `./tutorial/templates/tutorial/home.html`的行替换为以下代码行。</span><span class="sxs-lookup"><span data-stu-id="5f170-120">Replace the `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` line in `./tutorial/templates/tutorial/home.html` with the following.</span></span>

```html
<a href="{% url 'signin' %}" class="btn btn-primary btn-large">Click here to sign in</a>
```

<span data-ttu-id="5f170-121">将中`<a href="#" class="nav-link">Sign In</a>` `./tutorial/templates/tutorial/layout.html`的行替换为以下代码行。</span><span class="sxs-lookup"><span data-stu-id="5f170-121">Replace the `<a href="#" class="nav-link">Sign In</a>` line in `./tutorial/templates/tutorial/layout.html` with the following.</span></span>

```html
<a href="{% url 'signin' %}" class="nav-link">Sign In</a>
```

<span data-ttu-id="5f170-122">启动服务器并浏览到`https://localhost:8000/tutorial`。</span><span class="sxs-lookup"><span data-stu-id="5f170-122">Start the server and browse to `https://localhost:8000/tutorial`.</span></span> <span data-ttu-id="5f170-123">单击 "登录" 按钮，您应会被重定向`https://login.microsoftonline.com`到。</span><span class="sxs-lookup"><span data-stu-id="5f170-123">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="5f170-124">使用你的 Microsoft 帐户登录，并同意请求的权限。</span><span class="sxs-lookup"><span data-stu-id="5f170-124">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="5f170-125">浏览器重定向到应用程序，并显示令牌。</span><span class="sxs-lookup"><span data-stu-id="5f170-125">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="5f170-126">获取用户详细信息</span><span class="sxs-lookup"><span data-stu-id="5f170-126">Get user details</span></span>

<span data-ttu-id="5f170-127">在名为`./tutorial` `graph_helper.py`的目录中创建一个新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="5f170-127">Create a new file in the `./tutorial` directory named `graph_helper.py` and add the following code.</span></span>

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

<span data-ttu-id="5f170-128">该`get_user`方法通过使用您之前获取的访问令牌`/me` ，向 Microsoft GRAPH 终结点发出 get 请求，以获取用户的配置文件。</span><span class="sxs-lookup"><span data-stu-id="5f170-128">The `get_user` method makes a GET request to the Microsoft Graph `/me` endpoint to get the user's profile, using the access token you acquired previously.</span></span>

<span data-ttu-id="5f170-129">更新中`callback` `./tutorial/views.py`的方法以从 Microsoft Graph 获取用户的配置文件。</span><span class="sxs-lookup"><span data-stu-id="5f170-129">Update the `callback` method in `./tutorial/views.py` to get the user's profile from Microsoft Graph.</span></span>

<span data-ttu-id="5f170-130">首先，将以下`import`语句添加到文件顶部。</span><span class="sxs-lookup"><span data-stu-id="5f170-130">First, add the following `import` statement to the top of the file.</span></span>

```python
from tutorial.graph_helper import get_user
```

<span data-ttu-id="5f170-131">将`callback`方法替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="5f170-131">Replace the `callback` method with the following code.</span></span>

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

<span data-ttu-id="5f170-132">新代码调用`get_user`方法来请求用户的配置文件。</span><span class="sxs-lookup"><span data-stu-id="5f170-132">The new code calls the `get_user` method to request the user's profile.</span></span> <span data-ttu-id="5f170-133">它会将用户对象添加到临时输出中进行测试。</span><span class="sxs-lookup"><span data-stu-id="5f170-133">It adds the user object to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="5f170-134">存储令牌</span><span class="sxs-lookup"><span data-stu-id="5f170-134">Storing the tokens</span></span>

<span data-ttu-id="5f170-135">现在，您可以获取令牌，现在是时候实现将它们存储在应用程序中的方法了。</span><span class="sxs-lookup"><span data-stu-id="5f170-135">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="5f170-136">由于这是一个示例应用程序，因此为简单起见，你将把它们存储在会话中。</span><span class="sxs-lookup"><span data-stu-id="5f170-136">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="5f170-137">实际应用程序将使用更可靠的安全存储解决方案，就像数据库一样。</span><span class="sxs-lookup"><span data-stu-id="5f170-137">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="5f170-138">将以下新方法添加到`./tutorial/auth_helper.py`。</span><span class="sxs-lookup"><span data-stu-id="5f170-138">Add the following new methods to `./tutorial/auth_helper.py`.</span></span>

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

<span data-ttu-id="5f170-139">然后，更新中`callback` `./tutorial/views.py`的函数以将标记存储在会话中，并重定向回主页面。</span><span class="sxs-lookup"><span data-stu-id="5f170-139">Then, update the `callback` function in `./tutorial/views.py` to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="5f170-140">将`from tutorial.auth_helper import get_sign_in_url, get_token_from_code`行替换为以下代码行。</span><span class="sxs-lookup"><span data-stu-id="5f170-140">Replace the `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` line with the following.</span></span>

```python
from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
```

<span data-ttu-id="5f170-141">将`callback`方法替换为以下方法。</span><span class="sxs-lookup"><span data-stu-id="5f170-141">Replace the `callback` method with the following.</span></span>

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

## <a name="implement-sign-out"></a><span data-ttu-id="5f170-142">实现注销</span><span class="sxs-lookup"><span data-stu-id="5f170-142">Implement sign-out</span></span>

<span data-ttu-id="5f170-143">在测试此新功能之前，请添加一种注销方式。在中`./tutorial/views.py`添加`sign_out`新视图。</span><span class="sxs-lookup"><span data-stu-id="5f170-143">Before you test this new feature, add a way to sign out. Add a new `sign_out` view in `./tutorial/views.py`.</span></span>

```python
def sign_out(request):
  # Clear out the user and token
  remove_user_and_token(request)

  return HttpResponseRedirect(reverse('home'))
```

<span data-ttu-id="5f170-144">现在将此视图添加`./tutorial/urls.py`到。</span><span class="sxs-lookup"><span data-stu-id="5f170-144">Now add this view to `./tutorial/urls.py`.</span></span>

```python
path('signout', views.sign_out, name='signout'),
```

<span data-ttu-id="5f170-145">更新中\*\*\*\* `./tutorial/templates/tutorial/layout.html`的注销链接以使用此新视图。</span><span class="sxs-lookup"><span data-stu-id="5f170-145">Update the **Sign Out** link in `./tutorial/templates/tutorial/layout.html` to use this new view.</span></span> <span data-ttu-id="5f170-146">将`<a href="#" class="dropdown-item">Sign Out</a>`行替换为以下代码行。</span><span class="sxs-lookup"><span data-stu-id="5f170-146">Replace the `<a href="#" class="dropdown-item">Sign Out</a>` line with the following.</span></span>

```html
<a href="{% url 'signout' %}" class="dropdown-item">Sign Out</a>
```

<span data-ttu-id="5f170-147">重新启动服务器并完成登录过程。</span><span class="sxs-lookup"><span data-stu-id="5f170-147">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="5f170-148">您应该最后返回到主页，但 UI 应更改以指示您已登录。</span><span class="sxs-lookup"><span data-stu-id="5f170-148">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

<span data-ttu-id="5f170-150">单击右上角的用户头像以访问 "**注销**" 链接。</span><span class="sxs-lookup"><span data-stu-id="5f170-150">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="5f170-151">单击 "**注销**" 重置会话并返回到主页。</span><span class="sxs-lookup"><span data-stu-id="5f170-151">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![带有 "注销" 链接的下拉菜单的屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="5f170-153">刷新令牌</span><span class="sxs-lookup"><span data-stu-id="5f170-153">Refreshing tokens</span></span>

<span data-ttu-id="5f170-154">此时，您的应用程序具有访问令牌，该令牌是在 API `Authorization`调用的标头中发送的。</span><span class="sxs-lookup"><span data-stu-id="5f170-154">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="5f170-155">这是允许应用代表用户访问 Microsoft Graph 的令牌。</span><span class="sxs-lookup"><span data-stu-id="5f170-155">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="5f170-156">但是，此令牌的生存期较短。</span><span class="sxs-lookup"><span data-stu-id="5f170-156">However, this token is short-lived.</span></span> <span data-ttu-id="5f170-157">令牌在发出后会过期一小时。</span><span class="sxs-lookup"><span data-stu-id="5f170-157">The token expires an hour after it is issued.</span></span> <span data-ttu-id="5f170-158">这就是刷新令牌变得有用的地方。</span><span class="sxs-lookup"><span data-stu-id="5f170-158">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="5f170-159">刷新令牌允许应用在不要求用户重新登录的情况下请求新的访问令牌。</span><span class="sxs-lookup"><span data-stu-id="5f170-159">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="5f170-160">更新令牌管理代码以实现令牌刷新。</span><span class="sxs-lookup"><span data-stu-id="5f170-160">Update the token management code to implement token refresh.</span></span>

<span data-ttu-id="5f170-161">将中的`get_token`现有方法`./tutorial/auth_helper.py`替换为以下项。</span><span class="sxs-lookup"><span data-stu-id="5f170-161">Replace the existing `get_token` method in `./tutorial/auth_helper.py` with the following.</span></span>

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

<span data-ttu-id="5f170-162">此方法首先检查访问令牌是否已过期或接近即将过期。</span><span class="sxs-lookup"><span data-stu-id="5f170-162">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="5f170-163">如果是，则它使用刷新令牌获取新令牌，然后更新缓存并返回新的访问令牌。</span><span class="sxs-lookup"><span data-stu-id="5f170-163">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>
