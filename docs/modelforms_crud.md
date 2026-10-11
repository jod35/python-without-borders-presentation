# ModelForms: Create, Update, Delete

!!! note "Trainer snapshot"
    Duration: 90 min. Prerequisites: Ch5 method rules. Files: `tasks/tasks/forms.py`, `tasks/tasks/views.py`, `tasks/tasks/urls.py`, `tasks/templates/task_form.html`, `tasks/templates/task_confirm_delete.html`. Prepare: logged-out + logged-in browsers side by side (protection lands in Ch7, but views already require login). Start: read-only CRUD. End: full POST cycle verified.

## What we are learning

1. What a ModelForm is versus a plain HTML form.
2. How to render fields and validation errors.
3. How to save valid data and redirect.
4. How to delete safely through a confirmation POST.

## Why this matters

This is the chapter where the site becomes editable. Same three-step dance every time: GET shows the form, POST validates, valid saves and redirects.

## A simple explanation before code

An HTML form is just boxes in the browser. A ModelForm connects those boxes to the `Task` model: it renders fields, checks input, shows errors, and saves. You define which fields, Django does the repetitive work.

Important terms: ModelForm, validation, `form.is_valid()`, redirect-after-POST.

## Step-by-step example

### 1. Define the form (15 min)

Create `tasks/tasks/forms.py`:

```py title="task form"
# inside tasks/forms.py (new file)
from django import forms

from .models import Task


class TaskForm(forms.ModelForm):
    class Meta:
        model = Task
        fields = ["name", "description", "priority", "done"]
```

`Meta.model` picks the table. `fields` whitelists what visitors may set (never include `created_at`).

### 2. Create and update views (25 min)

Extend `tasks/tasks/views.py` (add to the Ch3 file, keep `index`/`task_detail`):

```py title="create and update"
from django.contrib.auth.decorators import login_required
from django.shortcuts import get_object_or_404, redirect, render

from .forms import TaskForm


@login_required
def task_create(request):
    if request.method == "POST":
        form = TaskForm(request.POST)
        if form.is_valid():
            task = form.save()
            return redirect("task_detail", pk=task.pk)
    else:
        form = TaskForm()
    return render(request, "task_form.html", {"form": form, "action": "Create"})
```

`task_update` is identical except it passes `instance=task` so the form edits instead of creating. Invalid POST skips the redirect and re-renders with errors. `@login_required` already guards these (fully taught in Ch7).

### 3. Templates and delete flow (20 min)

`tasks/templates/task_form.html`:

```html title="shared form template"
{% extends "base.html" %}
{% block title %}{{ action }} task | TaskRat{% endblock %}
{% block content %}
<h1>{{ action }} task</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    {% if form.errors %}
        <p>Please fix the errors above and submit again.</p>
    {% endif %}
    <button type="submit">{{ action }}</button>
</form>
<p><a href="{% url 'home_page' %}">Back to tasks</a></p>
{% endblock %}
```

`tasks/templates/task_confirm_delete.html` renders on GET and deletes only on POST:

```html title="delete confirmation"
{% extends "base.html" %}
{% block title %}Delete {{ task.name }} | TaskRat{% endblock %}
{% block content %}
<h1>Delete "{{ task.name }}"?</h1>
<p>This cannot be undone. Deleting happens only when you press the button below (a POST request).</p>
<form method="post">
    {% csrf_token %}
    <button type="submit">Yes, delete</button>
</form>
<p><a href="{% url 'task_detail' task.pk %}">Cancel</a></p>
{% endblock %}
```

Routes in `tasks/tasks/urls.py`:

```py title="crud routes"
path("tasks/new/", views.task_create, name="task_create"),
path("tasks/<int:pk>/edit/", views.task_update, name="task_update"),
path("tasks/<int:pk>/delete/", views.task_delete, name="task_delete"),
```

Order matters: `new/` sits before `<int:pk>/` patterns in reading order so learners never confuse them (Django matches in order; `new` is not an int so it is safe either way, but the habit helps).

## Let's build it: follow-along practical (15 min)

Create a task (lands on its detail page), submit an empty name (same page + "This field is required"), fix and resubmit, edit it, then open delete (GET shows question), confirm (POST deletes, lands on list).

## What the code is doing

HTML form versus ModelForm: HTML collects strings; ModelForm validates against the model and saves. GET versus POST: GET shows blank/prefilled form; POST carries the data. `form.save()` writes the row. `redirect` after success stops double-submit on refresh.

## Common mistakes and fixes

- `form.save()` on invalid data — always gate on `form.is_valid()` first.
- Missing `{% csrf_token %}` — 403 on POST.
- Delete works via link alone — your view deletes on GET. Move deletion inside `if request.method == "POST"`.
- `task_create` 404/redirect loop for anonymous users — expected: `@login_required` sends them to login (Ch7 explains).

## Try it yourself

Guided: submit each required field empty once and read the exact error. Independent: add `help_text` to one model field and confirm it appears in the form.

## Check your understanding

1. What connects a ModelForm to a table?
2. What happens on invalid POST?
3. Why redirect after a valid save?
4. Why does delete need two steps (GET confirm + POST)?
5. What does `instance=task` change?

## Summary

Define fields once, render with CSRF, validate, save, redirect. Deletes confirm on GET and act on POST.

## Next lesson

We add accounts so only logged-in users reach these write views.

!!! note "Trainer guide"
    Timed plan (adds to 90): form class 15, views 25, templates+delete 20, guided cycle 15, review + exit 15.
    Live-coding order: `forms.py` → blank GET → valid POST → redirect → invalid POST (errors) → update with instance → delete GET (no delete) → delete POST (gone). Pause on the invalid submit and ask "save or re-render?".
    Core idea: GET shows, POST decides, valid redirects.
    Misconceptions: errors mean crash (re-render is correct), redirect is optional (prevents duplicates), GET link can delete (never).
    Exit ticket: which HTTP method must delete on, and where is the token?
    Support: keep the three-step mantra visible. Extension: order `fields` differently, add widgets.
    Troubleshooting: 403 → token; silent non-save → `is_valid` gate; anonymous redirect → login (Ch7); wrong row edited → check `pk` in URL.
