<!-- markdownlint-disable MD002 MD041 -->

在本练习中, 将把 Microsoft Graph 合并到应用程序中。 对于此应用程序, 您将使用[flask-oauthlib](https://requests-oauthlib.readthedocs.io/en/latest/)库对 Microsoft Graph 进行调用。

## <a name="get-calendar-events-from-outlook"></a>从 Outlook 获取日历事件

首先添加方法以`./tutorial/graph_helper.py`提取日历事件。 添加以下方法。

```python
def get_calendar_events(token):
  graph_client = OAuth2Session(token=token)

  # Configure query parameters to
  # modify the results
  query_params = {
    '$select': 'subject,organizer,start,end',
    '$orderby': 'createdDateTime DESC'
  }

  # Send GET to /me/events
  events = graph_client.get('{0}/me/events'.format(graph_url), params=query_params)
  # Return the JSON result
  return events.json()
```

考虑此代码执行的操作。

- 将调用的 URL 为`/v1.0/me/events`。
- `$select`参数将为每个事件返回的字段限制为仅显示视图实际使用的字段。
- `$orderby`参数按其创建日期和时间对结果进行排序, 最新项目最先开始。

现在, 创建一个日历视图。 在`./tutorial/views.py`中, 首先将`from tutorial.graph_helper import get_user`行更改为以下代码行。

```python
from tutorial.graph_helper import get_user, get_calendar_events
```

然后, 将以下视图添加到`./tutorial/views.py`。

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

更新`./tutorial/urls.py`以添加此新视图。

```python
path('calendar', views.calendar, name='calendar'),
```

最后, 更新中**** `./tutorial/templates/tutorial/layout.html`的日历链接以链接到此视图。 将`<a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="#">Calendar</a>`行替换为以下代码行。

```html
<a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="{% url 'calendar' %}">Calendar</a>
```

现在, 您可以对此进行测试。 登录并单击导航栏中的 "**日历**" 链接。 如果一切正常, 应在用户的日历上看到一个 JSON 转储的事件。

## <a name="display-the-results"></a>显示结果

现在, 您可以添加一个模板, 以对用户更友好的方式显示结果。 在名为`./tutorial/templates/tutorial` `calendar.html`的目录中创建一个新文件, 并添加以下代码。

```html
{% extends "tutorial/layout.html" %}
{% block content %}
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    {% if events %}
      {% for event in events %}
        <tr>
          <td>{{ event.organizer.emailAddress.name }}</td>
          <td>{{ event.subject }}</td>
          <td>{{ event.start.dateTime|date:'SHORT_DATETIME_FORMAT' }}</td>
          <td>{{ event.end.dateTime|date:'SHORT_DATETIME_FORMAT' }}</td>
        </tr>
      {% endfor %}
    {% endif %}
  </tbody>
</table>
{% endblock %}
```

这将遍历一组事件并为每个事件添加一个表行。 将以下`import`语句添加到`./tutorials/views.py`文件顶部。

```python
import dateutil.parser
```

将`calendar`视图替换`./tutorial/views.py`为以下代码。

```python
def calendar(request):
  context = initialize_context(request)

  token = get_token(request)

  events = get_calendar_events(token)

  if events:
    # Convert the ISO 8601 date times to a datetime object
    # This allows the Django template to format the value nicely
    for event in events['value']:
      event['start']['dateTime'] = dateutil.parser.parse(event['start']['dateTime'])
      event['end']['dateTime'] = dateutil.parser.parse(event['end']['dateTime'])

    context['events'] = events['value']

  return render(request, 'tutorial/calendar.html', context)
```

刷新页面, 应用现在应呈现一个事件表。

![事件表的屏幕截图](./images/add-msgraph-01.png)
