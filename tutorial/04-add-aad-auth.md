<!-- markdownlint-disable MD002 MD041 -->

在本练习中，你将扩展上一练习中的应用程序，以支持 Azure AD 的身份验证。 若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph，这是必需的。 在此步骤中，将[flask-oauthlib](https://requests-oauthlib.readthedocs.io/en/latest/)库集成到应用程序中。

1. 在名为`oauth_settings.yml`的项目的根目录中创建一个新文件，并添加以下内容。

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. 将`YOUR_APP_ID_HERE`替换为应用程序注册门户中的应用程序 ID， `YOUR_APP_SECRET_HERE`并将替换为您生成的密码。

> [!IMPORTANT]
> 如果您使用的是源代码管理（如 git），现在是从源代码控制中**oauth_settings 排除 yml**文件以避免无意中泄漏您的应用程序 ID 和密码，这是一种很合适的时机。

## <a name="implement-sign-in"></a>实施登录

1. 在 **/tutorial**目录中创建一个名为`auth_helper.py`的新文件，并添加以下代码。

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    此文件将包含所有与身份验证相关的方法。 将`get_sign_in_url`生成一个授权 URL，并且该`get_token_from_code`方法将交换访问令牌的授权响应。

1. 将以下`import`语句添加到 **/tutorial/views.py**的顶部。

    ```python
    from django.urls import reverse
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code
    ```

1. 在 **./tutorial/views.py**文件中添加一个登录视图。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. 在 **./tutorial/views.py**文件中添加一个回调视图。

    ```python
    def callback(request):
      # Get the state saved in session
      expected_state = request.session.pop('auth_state', '')
      # Make the token request
      token = get_token_from_code(request.get_full_path(), expected_state)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(token) }
      return HttpResponseRedirect(reverse('home'))
    ```

    考虑这些视图的功能：

    - 该`signin`操作将生成 Azure AD 登录 URL，保存 OAuth `state`客户端生成的值，然后将浏览器重定向到 Azure AD 登录页。

    - `callback`操作是 Azure 在登录完成后重定向的位置。 该操作确保`state`值与保存的值相匹配，然后使用 Azure 发送的授权代码来请求访问令牌。 然后，它使用临时错误值中的访问令牌重定向回主页。 在继续操作之前，你将使用此操作来验证我们的登录是否正常工作。

1. 打开 **/tutorial/urls.py** ，并将现有`path`语句替换为`signin`以下项。

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. 为`callback`视图添加`path`新的。

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. 启动服务器并浏览到`https://localhost:8000`。 单击 "登录" 按钮，您应会被重定向`https://login.microsoftonline.com`到。 使用你的 Microsoft 帐户登录，并同意请求的权限。 浏览器重定向到应用程序，并显示令牌。

### <a name="get-user-details"></a>获取用户详细信息

1. 在 **/tutorial**目录中创建一个名为`graph_helper.py`的新文件，并添加以下代码。

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    该`get_user`方法通过使用您之前获取的访问令牌`/me` ，向 Microsoft GRAPH 终结点发出 get 请求，以获取用户的配置文件。

1. 更新/tutorial/views.py `callback`中的 **./tutorial/views.py**方法以从 Microsoft Graph 获取用户的配置文件。 将以下`import`语句添加到文件顶部。

    ```python
    from tutorial.graph_helper import get_user
    ```

1. 将`callback`方法替换为以下代码。

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

现在，您可以获取令牌，现在是时候实现将它们存储在应用程序中的方法了。 由于这是一个示例应用程序，因此为简单起见，你将把它们存储在会话中。 实际应用程序将使用更可靠的安全存储解决方案，就像数据库一样。

1. 将以下新方法添加到 **/tutorial/auth_helper py**。

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

1. 更新/tutorial/views.py `callback`中的 **./tutorial/views.py**函数以将令牌存储在会话中，并重定向回主页面。 将`from tutorial.auth_helper import get_sign_in_url, get_token_from_code`行替换为以下代码行。

    ```python
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
    ```

1. 将`callback`方法替换为以下方法。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a>实现注销

1. 在/tutorial/views.py 中`sign_out`添加新 **./tutorial/views.py**视图。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. 打开 **/tutorial/urls.py** ，并将现有`path`语句替换为`signout`以下项。

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. 重新启动服务器并完成登录过程。 您应该最后返回到主页，但 UI 应更改以指示您已登录。

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. 单击右上角的用户头像以访问 "**注销**" 链接。 单击 "**注销**" 重置会话并返回到主页。

    ![带有 "注销" 链接的下拉菜单的屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>刷新令牌

此时，您的应用程序具有访问令牌，该令牌是在 API `Authorization`调用的标头中发送的。 这是允许应用代表用户访问 Microsoft Graph 的令牌。

但是，此令牌的生存期较短。 令牌在发出后会过期一小时。 这就是刷新令牌变得有用的地方。 刷新令牌允许应用在不要求用户重新登录的情况下请求新的访问令牌。 更新令牌管理代码以实现令牌刷新。

1. 将`get_token` **/tutorial/auth_helper py**中的现有方法替换为以下项。

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="GetTokenSnippet":::

    此方法首先检查访问令牌是否已过期或接近即将过期。 如果是，则它使用刷新令牌获取新令牌，然后更新缓存并返回新的访问令牌。
