<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="7eb68-101">在本练习中，将把 Microsoft Graph 合并到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="7eb68-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="7eb68-102">对于此应用程序，您将使用[flask-oauthlib](https://requests-oauthlib.readthedocs.io/en/latest/)库对 Microsoft Graph 进行调用。</span><span class="sxs-lookup"><span data-stu-id="7eb68-102">For this application, you will use the [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="7eb68-103">从 Outlook 获取日历事件</span><span class="sxs-lookup"><span data-stu-id="7eb68-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="7eb68-104">首先，将方法添加到 **./tutorial/graph_helper py**以提取日历事件。</span><span class="sxs-lookup"><span data-stu-id="7eb68-104">Start by adding a method to **./tutorial/graph_helper.py** to fetch the calendar events.</span></span> <span data-ttu-id="7eb68-105">添加以下方法。</span><span class="sxs-lookup"><span data-stu-id="7eb68-105">Add the following method.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="GetCalendarSnippet":::

    <span data-ttu-id="7eb68-106">考虑此代码执行的操作。</span><span class="sxs-lookup"><span data-stu-id="7eb68-106">Consider what this code is doing.</span></span>

    - <span data-ttu-id="7eb68-107">将调用的 URL 为`/v1.0/me/events`。</span><span class="sxs-lookup"><span data-stu-id="7eb68-107">The URL that will be called is `/v1.0/me/events`.</span></span>
    - <span data-ttu-id="7eb68-108">`$select`参数将为每个事件返回的字段限制为仅显示视图实际使用的字段。</span><span class="sxs-lookup"><span data-stu-id="7eb68-108">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="7eb68-109">`$orderby`参数按其创建日期和时间对结果进行排序，最新项目最先开始。</span><span class="sxs-lookup"><span data-stu-id="7eb68-109">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="7eb68-110">在 **/tutorial/views.py**中，将`from tutorial.graph_helper import get_user`行更改为以下代码行。</span><span class="sxs-lookup"><span data-stu-id="7eb68-110">In **./tutorial/views.py**, change the `from tutorial.graph_helper import get_user` line to the following.</span></span>

    ```python
    from tutorial.graph_helper import get_user, get_calendar_events
    ```

1. <span data-ttu-id="7eb68-111">将以下视图添加到 **/tutorial/views.py**。</span><span class="sxs-lookup"><span data-stu-id="7eb68-111">Add the following view to **./tutorial/views.py**.</span></span>

    ```python
    def calendar(request):
      context = initialize_context(request)

      token = get_token(request)

      events = get_calendar_events(token)

      context['errors'] = [
        { 'message': 'Events', 'debug': format(events)}
      ]

      return render(request, 'tutorial/home.html', context)
    ```

1. <span data-ttu-id="7eb68-112">打开 **/tutorial/urls.py** ，并将现有`path`语句替换为`calendar`以下项。</span><span class="sxs-lookup"><span data-stu-id="7eb68-112">Open **./tutorial/urls.py** and replace the existing `path` statements for `calendar` with the following.</span></span>

    ```python
    path('calendar', views.calendar, name='calendar'),
    ```

1. <span data-ttu-id="7eb68-113">登录并单击导航栏中的 "**日历**" 链接。</span><span class="sxs-lookup"><span data-stu-id="7eb68-113">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="7eb68-114">如果一切正常，应在用户的日历上看到一个 JSON 转储的事件。</span><span class="sxs-lookup"><span data-stu-id="7eb68-114">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="7eb68-115">显示结果</span><span class="sxs-lookup"><span data-stu-id="7eb68-115">Display the results</span></span>

<span data-ttu-id="7eb68-116">现在，您可以添加一个模板，以对用户更友好的方式显示结果。</span><span class="sxs-lookup"><span data-stu-id="7eb68-116">Now you can add a template to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="7eb68-117">在 **/tutorial/templates/tutorial**目录中创建一个名为`calendar.html`的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="7eb68-117">Create a new file in the **./tutorial/templates/tutorial** directory named `calendar.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/calendar.html" id="CalendarSnippet":::

    <span data-ttu-id="7eb68-118">这将遍历一组事件并为每个事件添加一个表行。</span><span class="sxs-lookup"><span data-stu-id="7eb68-118">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="7eb68-119">将以下`import`语句添加到 **/tutorials/views.py**文件的顶部。</span><span class="sxs-lookup"><span data-stu-id="7eb68-119">Add the following `import` statement to the top of the **./tutorials/views.py** file.</span></span>

    ```python
    import dateutil.parser
    ```

1. <span data-ttu-id="7eb68-120">将/tutorial/views.py `calendar`中的 **./tutorial/views.py**视图替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="7eb68-120">Replace the `calendar` view in **./tutorial/views.py** with the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CalendarViewSnippet":::

1. <span data-ttu-id="7eb68-121">刷新页面，应用现在应呈现一个事件表。</span><span class="sxs-lookup"><span data-stu-id="7eb68-121">Refresh the page and the app should now render a table of events.</span></span>

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
