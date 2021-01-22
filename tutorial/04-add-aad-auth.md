<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将扩展上一练习中的应用程序，以支持使用 Azure AD 进行身份验证。 这是获取调用 Microsoft Graph 所需的 OAuth 访问令牌所必需的。 在此步骤中，你将 [MSAL for Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) 库集成到应用程序中。

1. 在名为项目根目录的项目中创建新文件 `oauth_settings.yml` ，并添加以下内容。

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. 替换为 `YOUR_APP_ID_HERE` 应用程序注册门户中的应用程序 ID，并 `YOUR_APP_SECRET_HERE` 替换为生成的密码。

> [!IMPORTANT]
> 如果你使用的是源代码管理（如 git），那么现在应该从源代码管理中排除 **oauth_settings.yml** 文件，以避免意外泄露应用 ID 和密码。

## <a name="implement-sign-in"></a>实现登录

1. 在 **./tutorial** 目录中创建一个名为的新文件， `auth_helper.py` 并添加以下代码。

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    此文件将保存所有与身份验证相关的方法。 该方法 `get_sign_in_flow` 将生成授权 URL，并 `get_token_from_code` 交换访问令牌的授权响应。

1. 将以下 `import` 语句添加到 **./tutorial/views.py 的顶部**。

    ```python
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code
    ```

1. 在 **./tutorial/views.py** 文件中添加登录视图。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. 在 **./tutorial/views.py** 文件中添加回调视图。

    ```python
    def callback(request):
      # Make the token request
      result = get_token_from_code(request)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(result) }
      return HttpResponseRedirect(reverse('home'))
    ```

    请考虑这些视图执行哪些功能：

    - 该操作将生成 Azure AD 登录 URL，保存 OAuth 客户端生成的流，然后将浏览器重定向到 `signin` Azure AD 登录页。

    - 此操作 `callback` 是 Azure 在登录完成后重定向的地方。 该操作使用已保存的流和 Azure 发送的查询字符串请求访问令牌。 然后，它将重定向回主页，并返回临时错误值中的响应。 在继续之前，你将使用此验证登录是否正常工作。

1. 打开 **./tutorial/urls.py，** 将现有 `path` 语句 `signin` 替换为以下内容。

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. 为视图 `path` 添加新 `callback` 视图。

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. 启动服务器并浏览到 `https://localhost:8000` 。 单击登录按钮，应重定向到 `https://login.microsoftonline.com` 。 使用 Microsoft 帐户登录，并同意请求的权限。 浏览器重定向到应用，显示响应，包括访问令牌。

### <a name="get-user-details"></a>获取用户详细信息

1. 在 **./tutorial** 目录中创建一个名为的新文件， `graph_helper.py` 并添加以下代码。

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    此方法使用你之前获取的访问令牌向 Microsoft Graph 终结点发送 GET 请求，获取 `get_user` `/me` 用户配置文件。

1. 更新 `callback` **./tutorial/views.py** 中的方法，从 Microsoft Graph 获取用户配置文件。 将以下 `import` 语句添加到文件顶部。

    ```python
    from tutorial.graph_helper import *
    ```

1. 将 `callback` 该方法替换为以下代码。

    ```python
    def callback(request):
      # Make the token request
      result = get_token_from_code(request)

      #Get the user's profile
      user = get_user(result['access_token'])
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': 'User: {0}\nToken: {1}'.format(user, result) }
      return HttpResponseRedirect(reverse('home'))
    ```

    新代码调用 `get_user` 此方法以请求用户配置文件。 它将用户对象添加到临时输出中进行测试。

1. 将以下新方法添加到 **./tutorial/auth_helper.py**。

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="SecondCodeSnippet":::

1. 更新 `callback` **./tutorial/views.py** 中的函数以在会话中存储用户并重定向回主页。 将 `from tutorial.auth_helper import get_sign_in_flow, get_token_from_code` 该行替换为以下内容。

    ```python
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code, store_user, remove_user_and_token, get_token
    ```

1. 将 `callback` 该方法替换为以下内容。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a>实现注销

1. 在 `sign_out` **./tutorial/views.py 中添加新视图**。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. 打开 **./tutorial/urls.py，** 将现有 `path` 语句 `signout` 替换为以下内容。

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. 重新启动服务器并完成登录过程。 你最终应返回主页，但 UI 应更改以指示你已登录。

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. 单击右上角的用户头像以访问 **"注销"** 链接。 单击 **"注销** "可重置会话，并返回到主页。

    ![包含"注销"链接的下拉菜单屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>刷新令牌

此时，应用程序具有访问令牌，该令牌在 API 调用 `Authorization` 标头中发送。 这是允许应用代表用户访问 Microsoft Graph 的令牌。

但是，此令牌是短期的。 令牌在颁发后一小时过期。 刷新令牌在这里变得有用。 刷新令牌允许应用请求新的访问令牌，而无需用户重新登录。

由于此示例使用 MSAL，因此不需要编写任何特定代码来刷新令牌。 MSAL `acquire_token_silent` 的方法可处理刷新令牌（如果需要）。
