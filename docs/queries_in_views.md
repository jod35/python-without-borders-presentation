# Queries in Views

!!! note "Trainer snapshot"
    Duration: 90 min. Prerequisites: Ch1–Ch2, rows exist. Files: `tasks/tasks/views.py`, `tasks/tasks/urls.py`, `tasks/templates/index.html`, `tasks/templates/detail.html`. Prepare: two tasks in DB, one bad pk to demo 404. Start: dummy-list `index`. End: ORM list + detail with 404 and empty state.

## What we are learning

1. How a URL pattern connects to a view.
2. How to query data inside a view.
3. How to pass QuerySets to templates.
4. How to handle missing objects and empty lists.

## Why this matters

This replaces the dummy list with the real database. Every later chapter builds on this exact pattern.

## A simple explanation before code

A URL pattern says "this address runs that function". The view function queries the database, puts the result in a dictionary called context, and renders a template. The template loops over the context. Missing rows should give a 404 page, not a crash.

Important terms: URL pattern, view, context, `get_object_or_404`, `empty` state.

## Step-by-step example

### 1. The old dummy (10 min)

The old `tasks/tasks/views.py` held a hardcoded list with fields like `due_date` and lowercase `priority`. The real `Task` model has no `due_date` and uses `LOW`/`HIGH`. We now delete the dummy list and query the table instead. That drift is intentional to discuss, not to hide.

### 2. List view with a real QuerySet (20 min)

Replace `tasks/tasks/views.py` with:

```py title="ORM views"
# inside tasks/views.py (replaces the dummy list version)
from django.shortcuts import get_object_or_404, render

from .models import Task


def index(request):
    query = request.GET.get("q", "").strip()
    tasks = Task.objects.all()
    if query:
        tasks = tasks.filter(name__icontains=query)
    return render(request, "index.html", {"tasks": tasks, "query": query})


def task_detail(request, pk):
    task = get_object_or_404(Task, pk=pk)
    return render(request, "detail.html", {"task": task})
```

`index` passes a QuerySet as `tasks`. `task_detail` fetches one row or raises a 404. The `q` filter is a small GET bonus fully taught in Ch5; for now just notice it does not change data.

### 3. URLs (10 min)

Replace `tasks/tasks/urls.py` with:

```py title="task routes"
# inside tasks/urls.py (replaces the single-route version)
from django.urls import path

from . import views

urlpatterns = [
    path("", views.index, name="home_page"),
    path("tasks/<int:pk>/", views.task_detail, name="task_detail"),
]
```

`<int:pk>` captures the row id and passes it as `pk`. `name` lets templates link with `{% url %}` instead of hardcoding paths.

### 4. Templates (20 min)

Replace `tasks/templates/index.html` with:

```html title="list template"
{% extends "base.html" %}
{% block title %}Tasks | TaskRat{% endblock %}
{% block content %}
<h1>Tasks</h1>
<form method="get" action="{% url 'home_page' %}">
    <input type="search" name="q" placeholder="Search tasks" value="{{ query }}">
    <button type="submit">Search</button>
</form>
{% for task in tasks %}
    <div>
        <h2><a href="{% url 'task_detail' task.pk %}">{{ task.name }}</a> <small>created on {{ task.created_at }}</small></h2>
        <p>{{ task.description }}</p>
        <p>Priority: {{ task.priority }} | {% if task.done %}Done{% else %}Not done{% endif %}</p>
    </div>
{% empty %}
    <p>No tasks yet. Add your first task.</p>
{% endfor %}
{% endblock %}
```

Add `tasks/templates/detail.html`:

```html title="detail template"
{% extends "base.html" %}
{% block title %}{{ task.name }} | TaskRat{% endblock %}
{% block content %}
<h1>{{ task.name }}</h1>
<p>{{ task.description }}</p>
<p>Priority: {{ task.priority }} | {% if task.done %}Done{% else %}Not done{% endif %}</p>
<p>Created on {{ task.created_at }} | Updated on {{ task.updated_at }}</p>
<p><a href="{% url 'home_page' %}">Back to tasks</a></p>
{% endblock %}
```

`{% empty %}` covers zero rows. `{% if task.done %}` shows state. `{% url 'task_detail' task.pk %}` builds links from route names. (`base.html` itself is taught in Ch4; until then these templates assume it exists at the end checkpoint.)

## Let's build it: follow-along practical (15 min)

Visit `/` (rows appear), visit `/tasks/1/` (one row), visit `/tasks/9999/` (404 page). Delete all rows in admin and reload `/` to see the empty message, then re-add one.

## What the code is doing

URL pattern versus view: pattern matches the address, view runs Python. View versus template: view decides what data, template decides what HTML. Context versus HTML: `{"tasks": tasks}` is data; the `for` loop turns it into HTML.

## Common mistakes and fixes

- `NoReverseMatch` — route name misspelled or `pk` missing in `{% url %}`. Compare with `urls.py`.
- 404 on a row you just made — wrong pk in the address. Click the link instead of typing.
- Old dummy fields referenced (`due_date`) — the model has no such field; use `priority/done/created_at`.
- `VariableDoesNotExist` — context key and template name disagree (`tasks` vs `task_list`).

## Try it yourself

Guided: add `{% if %}` around the search box result count. Independent: order the list by `-created_at` explicitly in the view and explain why it matches `Meta.ordering`.

## Check your understanding

1. What does a URL pattern do?
2. What is context?
3. What happens when `get_object_or_404` finds nothing?
4. What does `{% empty %}` do?
5. Why do we use `{% url %}` instead of writing `/tasks/1/` by hand?

## Summary

Pattern runs view, view queries and passes context, template loops and links. Missing row gives 404, empty table gives the empty message.

## Next lesson

We share page structure and styling with a base template and static CSS.

!!! note "Trainer guide"
    Timed plan (adds to 90): dummy recap 10, list view 20, URLs 10, templates 20, guided browsing 15, review + exit 15.
    Live-coding order: show dummy list → delete it → write ORM `index` → reload → add route → 404 demo with bad pk → write templates → empty-state demo (delete rows, reload, re-add). Ask "where does data come from now?" after each reload.
    Core idea: view decides what, template decides how it looks.
    Misconceptions: template queries the DB (view does), 404 is a bug (it is correct for missing rows), route name equals address (name is for reversing).
    Exit ticket: what renders when the table is empty, and which tag produces it?
    Support: keep admin open to prove data vs display bugs. Extension: filter done-only tasks in the view.
    Troubleshooting: NoReverseMatch → names; blank page → context key mismatch; stale dummy fields → model has no `due_date`.
