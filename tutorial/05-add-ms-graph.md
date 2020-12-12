<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将 Microsoft Graph 合并到应用程序中。

## <a name="get-calendar-events-from-outlook"></a>从 Outlook 获取日历事件

1. 首先，将方法添加到 **./tutorial/graph_helper.py，** 获取两个日期之间的日历视图。 添加以下方法。

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="GetCalendarSnippet":::

    考虑此代码正在做什么。

    - 将调用的 URL 为 `/v1.0/me/calendarview` 。
        - 标头使结果中的开始时间和结束 `Prefer: outlook.timezone` 时间调整为用户的时区。
        - and `startDateTime` `endDateTime` 参数设置视图的开始和结束。
        - 该 `$select` 参数将每个事件返回的字段限制为仅视图将实际使用的字段。
        - 此 `$orderby` 参数按开始时间对结果进行排序。
        - 此 `$top` 参数将结果限制为 50 个事件。

1. 将以下代码添加到 **./tutorial/graph_helper.py** 以查找基于 Windows 时区名称的 [IANA](https://www.iana.org/time-zones) 时区标识符。 这是必需的，因为 Microsoft Graph 可以将时区作为 Windows 时区名称返回，而 Python **日期** 时间库需要 IANA 时区标识符。

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="ZoneMappingsSnippet":::

1. 将以下视图添加到 **./tutorial/views.py**。

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

1. 打开 **./tutorial/urls.py，** 将现有 `path` 语句 `calendar` 替换为以下内容。

    ```python
    path('calendar', views.calendar, name='calendar'),
    ```

1. 登录并单击导航 **栏中** 的"日历"链接。 如果一切正常，应在用户日历上看到事件的 JSON 转储。

## <a name="display-the-results"></a>显示结果

现在，您可以添加一个模板，以更用户友好的方式显示结果。

1. 在 **./tutorial/templates/tutorial** 目录中创建一个名为的新文件 `calendar.html` ，并添加以下代码。

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/calendar.html" id="CalendarSnippet":::

    这将循环访问事件集合，并针对每个事件添加一个表格行。

1. 将 `calendar` **./tutorial/views.py 中的视图替换为** 以下代码。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CalendarViewSnippet":::

1. 刷新页面，应用现在应呈现事件表。

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
