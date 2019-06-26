<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="33430-101">在本练习中, 将把 Microsoft Graph 合并到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="33430-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="33430-102">对于此应用程序, 您将使用[flask-oauthlib](https://requests-oauthlib.readthedocs.io/en/latest/)库对 Microsoft Graph 进行调用。</span><span class="sxs-lookup"><span data-stu-id="33430-102">For this application, you will use the [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="33430-103">从 Outlook 获取日历事件</span><span class="sxs-lookup"><span data-stu-id="33430-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="33430-104">首先添加方法以`./tutorial/graph_helper.py`提取日历事件。</span><span class="sxs-lookup"><span data-stu-id="33430-104">Start by adding a method to `./tutorial/graph_helper.py` to fetch the calendar events.</span></span> <span data-ttu-id="33430-105">添加以下方法。</span><span class="sxs-lookup"><span data-stu-id="33430-105">Add the following method.</span></span>

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

<span data-ttu-id="33430-106">考虑此代码执行的操作。</span><span class="sxs-lookup"><span data-stu-id="33430-106">Consider what this code is doing.</span></span>

- <span data-ttu-id="33430-107">将调用的 URL 为`/v1.0/me/events`。</span><span class="sxs-lookup"><span data-stu-id="33430-107">The URL that will be called is `/v1.0/me/events`.</span></span>
- <span data-ttu-id="33430-108">`$select`参数将为每个事件返回的字段限制为仅显示视图实际使用的字段。</span><span class="sxs-lookup"><span data-stu-id="33430-108">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
- <span data-ttu-id="33430-109">`$orderby`参数按其创建日期和时间对结果进行排序, 最新项目最先开始。</span><span class="sxs-lookup"><span data-stu-id="33430-109">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

<span data-ttu-id="33430-110">现在, 创建一个日历视图。</span><span class="sxs-lookup"><span data-stu-id="33430-110">Now create a calendar view.</span></span> <span data-ttu-id="33430-111">在`./tutorial/views.py`中, 首先将`from tutorial.graph_helper import get_user`行更改为以下代码行。</span><span class="sxs-lookup"><span data-stu-id="33430-111">In `./tutorial/views.py`, first change the `from tutorial.graph_helper import get_user` line to the following.</span></span>

```python
from tutorial.graph_helper import get_user, get_calendar_events
```

<span data-ttu-id="33430-112">然后, 将以下视图添加到`./tutorial/views.py`。</span><span class="sxs-lookup"><span data-stu-id="33430-112">Then, add the following view to `./tutorial/views.py`.</span></span>

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

<span data-ttu-id="33430-113">更新`./tutorial/urls.py`以添加此新视图。</span><span class="sxs-lookup"><span data-stu-id="33430-113">Update `./tutorial/urls.py` to add this new view.</span></span>

```python
path('calendar', views.calendar, name='calendar'),
```

<span data-ttu-id="33430-114">最后, 更新中\*\*\*\* `./tutorial/templates/tutorial/layout.html`的日历链接以链接到此视图。</span><span class="sxs-lookup"><span data-stu-id="33430-114">Finally, update  the **Calendar** link in `./tutorial/templates/tutorial/layout.html` to link to this view.</span></span> <span data-ttu-id="33430-115">将`<a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="#">Calendar</a>`行替换为以下代码行。</span><span class="sxs-lookup"><span data-stu-id="33430-115">Replace the `<a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="#">Calendar</a>` line with the following.</span></span>

```html
<a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="{% url 'calendar' %}">Calendar</a>
```

<span data-ttu-id="33430-116">现在, 您可以对此进行测试。</span><span class="sxs-lookup"><span data-stu-id="33430-116">Now you can test this.</span></span> <span data-ttu-id="33430-117">登录并单击导航栏中的 "**日历**" 链接。</span><span class="sxs-lookup"><span data-stu-id="33430-117">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="33430-118">如果一切正常, 应在用户的日历上看到一个 JSON 转储的事件。</span><span class="sxs-lookup"><span data-stu-id="33430-118">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="33430-119">显示结果</span><span class="sxs-lookup"><span data-stu-id="33430-119">Display the results</span></span>

<span data-ttu-id="33430-120">现在, 您可以添加一个模板, 以对用户更友好的方式显示结果。</span><span class="sxs-lookup"><span data-stu-id="33430-120">Now you can add a template to display the results in a more user-friendly manner.</span></span> <span data-ttu-id="33430-121">在名为`./tutorial/templates/tutorial` `calendar.html`的目录中创建一个新文件, 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="33430-121">Create a new file in the `./tutorial/templates/tutorial` directory named `calendar.html` and add the following code.</span></span>

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

<span data-ttu-id="33430-122">这将遍历一组事件并为每个事件添加一个表行。</span><span class="sxs-lookup"><span data-stu-id="33430-122">That will loop through a collection of events and add a table row for each one.</span></span> <span data-ttu-id="33430-123">将以下`import`语句添加到`./tutorials/views.py`文件顶部。</span><span class="sxs-lookup"><span data-stu-id="33430-123">Add the following `import` statement to the top of the `./tutorials/views.py` file.</span></span>

```python
import dateutil.parser
```

<span data-ttu-id="33430-124">将`calendar`视图替换`./tutorial/views.py`为以下代码。</span><span class="sxs-lookup"><span data-stu-id="33430-124">Replace the `calendar` view in `./tutorial/views.py` with the following code.</span></span>

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

<span data-ttu-id="33430-125">刷新页面, 应用现在应呈现一个事件表。</span><span class="sxs-lookup"><span data-stu-id="33430-125">Refresh the page and the app should now render a table of events.</span></span>

![事件表的屏幕截图](./images/add-msgraph-01.png)
