<!-- markdownlint-disable MD002 MD041 -->

在此部分中，您将添加在用户日历上创建事件的能力。

1. 将以下方法添加到 **./tutorial/graph_helper.py** 以创建新事件。

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="CreateEventSnippet":::

## <a name="create-a-new-event-form"></a>创建新的事件表单

1. 在 **./tutorial/templates/tutorial** 目录中创建一个名为的新文件 `newevent.html` ，并添加以下代码。

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/newevent.html" id="NewEventSnippet":::

1. 将以下视图添加到 **./tutorial/views.py**。

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="NewEventViewSnippet":::

1. 打开 **./tutorial/urls.py，** 然后为 `path` 视图添加 `newevent` 语句。

    ```python
    path('calendar/new', views.newevent, name='newevent'),
    ```

1. 保存更改并刷新应用。 在 **"日历** "页上，选择 **"新建事件"** 按钮。 填写表单，然后选择" **创建** "以创建事件。

    ![新事件表单的屏幕截图](images/create-event-01.png)
