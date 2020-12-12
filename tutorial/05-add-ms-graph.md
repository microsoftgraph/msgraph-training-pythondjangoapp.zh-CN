<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f4616-101">在此练习中，你将 Microsoft Graph 合并到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="f4616-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="f4616-102">从 Outlook 获取日历事件</span><span class="sxs-lookup"><span data-stu-id="f4616-102">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="f4616-103">首先，将方法添加到 **./tutorial/graph_helper.py，** 获取两个日期之间的日历视图。</span><span class="sxs-lookup"><span data-stu-id="f4616-103">Start by adding a method to **./tutorial/graph_helper.py** to fetch a view of the calendar between two dates.</span></span> <span data-ttu-id="f4616-104">添加以下方法。</span><span class="sxs-lookup"><span data-stu-id="f4616-104">Add the following method.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="GetCalendarSnippet":::

    <span data-ttu-id="f4616-105">考虑此代码正在做什么。</span><span class="sxs-lookup"><span data-stu-id="f4616-105">Consider what this code is doing.</span></span>

    - <span data-ttu-id="f4616-106">将调用的 URL 为 `/v1.0/me/calendarview` 。</span><span class="sxs-lookup"><span data-stu-id="f4616-106">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="f4616-107">标头使结果中的开始时间和结束 `Prefer: outlook.timezone` 时间调整为用户的时区。</span><span class="sxs-lookup"><span data-stu-id="f4616-107">The `Prefer: outlook.timezone` header causes the start and end times in the results to be adjusted to the user's time zone.</span></span>
        - <span data-ttu-id="f4616-108">and `startDateTime` `endDateTime` 参数设置视图的开始和结束。</span><span class="sxs-lookup"><span data-stu-id="f4616-108">The `startDateTime` and `endDateTime` parameters set the start and end of the view.</span></span>
        - <span data-ttu-id="f4616-109">该 `$select` 参数将每个事件返回的字段限制为仅视图将实际使用的字段。</span><span class="sxs-lookup"><span data-stu-id="f4616-109">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
        - <span data-ttu-id="f4616-110">此 `$orderby` 参数按开始时间对结果进行排序。</span><span class="sxs-lookup"><span data-stu-id="f4616-110">The `$orderby` parameter sorts the results by start time.</span></span>
        - <span data-ttu-id="f4616-111">此 `$top` 参数将结果限制为 50 个事件。</span><span class="sxs-lookup"><span data-stu-id="f4616-111">The `$top` parameter limits the results to 50 events.</span></span>

1. <span data-ttu-id="f4616-112">将以下代码添加到 **./tutorial/graph_helper.py** 以查找基于 Windows 时区名称的 [IANA](https://www.iana.org/time-zones) 时区标识符。</span><span class="sxs-lookup"><span data-stu-id="f4616-112">Add the following code to **./tutorial/graph_helper.py** to lookup an [IANA time zone identifier](https://www.iana.org/time-zones) based on a Windows time zone name.</span></span> <span data-ttu-id="f4616-113">这是必需的，因为 Microsoft Graph 可以将时区作为 Windows 时区名称返回，而 Python **日期** 时间库需要 IANA 时区标识符。</span><span class="sxs-lookup"><span data-stu-id="f4616-113">This is necessary because Microsoft Graph can return time zones as Windows time zone names, and the Python **datetime** library requires IANA time zone identifiers.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="ZoneMappingsSnippet":::

1. <span data-ttu-id="f4616-114">将以下视图添加到 **./tutorial/views.py**。</span><span class="sxs-lookup"><span data-stu-id="f4616-114">Add the following view to **./tutorial/views.py**.</span></span>

    ```python
    def calendar(request):
      context = initialize_context(request)
      user = context['user']

      # Load the user's time zone
      # Microsoft Graph can return the user's time zone as either
      # a Windows time zone name or an IANA time zone identifier
      # Python datetime requires IANA, so convert Windows to IANA
      time_zone = get_iana_from_windows(user['timeZone'])
      tz_info = tz.gettz(time_zone)

      # Get midnight today in user's time zone
      today = datetime.now(tz_info).replace(
        hour=0,
        minute=0,
        second=0,
        microsecond=0)

      # Based on today, get the start of the week (Sunday)
      if (today.weekday() != 6):
        start = today - timedelta(days=today.isoweekday())
      else:
        start = today

      end = start + timedelta(days=7)

      token = get_token(request)

      events = get_calendar_events(
        token,
        start.isoformat(timespec='seconds'),
        end.isoformat(timespec='seconds'),
        user['timeZone'])

      context['errors'] = [
        { 'message': 'Events', 'debug': format(events)}
      ]

      return render(request, 'tutorial/home.html', context)
    ```

1. <span data-ttu-id="f4616-115">打开 **./tutorial/urls.py，** 将现有 `path` 语句 `calendar` 替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="f4616-115">Open **./tutorial/urls.py** and replace the existing `path` statements for `calendar` with the following.</span></span>

    ```python
    path('calendar', views.calendar, name='calendar'),
    ```

1. <span data-ttu-id="f4616-116">登录并单击导航 **栏中** 的"日历"链接。</span><span class="sxs-lookup"><span data-stu-id="f4616-116">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="f4616-117">如果一切正常，应在用户日历上看到事件的 JSON 转储。</span><span class="sxs-lookup"><span data-stu-id="f4616-117">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="f4616-118">显示结果</span><span class="sxs-lookup"><span data-stu-id="f4616-118">Display the results</span></span>

<span data-ttu-id="f4616-119">现在，您可以添加一个模板，以更用户友好的方式显示结果。</span><span class="sxs-lookup"><span data-stu-id="f4616-119">Now you can add a template to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="f4616-120">在 **./tutorial/templates/tutorial** 目录中创建一个名为的新文件 `calendar.html` ，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="f4616-120">Create a new file in the **./tutorial/templates/tutorial** directory named `calendar.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/calendar.html" id="CalendarSnippet":::

    <span data-ttu-id="f4616-121">这将循环访问事件集合，并针对每个事件添加一个表格行。</span><span class="sxs-lookup"><span data-stu-id="f4616-121">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="f4616-122">将 `calendar` **./tutorial/views.py 中的视图替换为** 以下代码。</span><span class="sxs-lookup"><span data-stu-id="f4616-122">Replace the `calendar` view in **./tutorial/views.py** with the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CalendarViewSnippet":::

1. <span data-ttu-id="f4616-123">刷新页面，应用现在应呈现事件表。</span><span class="sxs-lookup"><span data-stu-id="f4616-123">Refresh the page and the app should now render a table of events.</span></span>

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
