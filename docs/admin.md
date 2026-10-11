# The Django Admin

!!! note "Trainer snapshot"
    Duration: 60 min. Prerequisites: Ch1 migration applied. Files: `tasks/tasks/admin.py`. Prepare: superuser credentials, `/admin/` open. Start: model unregistered. End: `TaskAdmin` with list/filter/search working.

## What we are learning

1. What the admin site is for.
2. How to create an admin user and log in.
3. How to register a model.
4. How to customize the list page.

## Why this matters

The admin is your fastest inspection tool. When a view looks wrong later, you check the admin first to see whether the data or the code is at fault.

## A simple explanation before code

Django ships with a staff-only site at `/admin/`. Your models do not appear there until you register them. Registration is one line; customization is a small class that says which columns, filters, and search box to show.

Important terms: superuser, registration, `ModelAdmin`, `list_display`.

## Step-by-step example

### 1. Create an admin user (10 min)

```title="create a superuser"
python manage.py createsuperuser
```

Pick a username and password you will remember for class. Log in at `http://127.0.0.1:8000/admin/`. You will see Users and Groups but no Tasks yet.

### 2. Register the model (15 min)

Open `tasks/tasks/admin.py`. Replace its content with:

```py title="register Task"
# inside tasks/admin.py (replaces the placeholder)
from django.contrib import admin

from .models import Task


@admin.register(Task)
class TaskAdmin(admin.ModelAdmin):
    list_display = ("name", "priority", "done", "created_at")
    list_filter = ("priority", "done")
    search_fields = ("name",)
    fields = ("name", "description", "priority", "done")
```

Refresh `/admin/`. Tasks now appears. `list_display` picks the columns. `list_filter` adds the right-hand filters. `search_fields` adds the search box. `fields` controls the edit form order.

### 3. Use it (15 min)

Add two tasks through the admin, then use search and the priority filter. Notice `__str__` from Ch1 is what each row is called. This is the same data your shell created.

Note: an earlier lesson wrote the filter as `|lowercase`. The correct Django filter is `|lower`. Use `|lower` in templates.

## Let's build it: follow-along practical (10 min)

Each learner adds one task in the admin, finds it with search, and toggles `done` on the edit form.

## What the code is doing

`@admin.register(Task)` connects the model to the admin. `TaskAdmin` does not change the database; it only changes how staff browse rows. `fields` omits `created_at/updated_at` because those are automatic.

## Common mistakes and fixes

- Model missing from admin — forgot to save `admin.py` or used the wrong app file. Restart `runserver` and refresh.
- `createsuperuser` password rejected — Django validates length and common passwords; choose a longer one for class.
- Edits in admin not visible on site — the site still shows dummy data until Ch3 swaps in the ORM. That is expected right now.

## Try it yourself

Guided: add a `list_filter` entry and prove it filters. Independent: add `search_fields` for description and test it, then revert if search gets noisy.

## Check your understanding

1. What URL serves the admin?
2. What command creates a superuser?
3. What line makes a model appear in the admin?
4. What do `list_display`, `list_filter`, and `search_fields` each do?

## Summary

Admin is for staff, not visitors. Register once, customize the list page, and use it to verify data through the rest of the course.

## Next lesson

We query this same data inside views and show it in templates.

!!! note "Trainer guide"
    Timed plan (adds to 60): recap 5, superuser 10, live register 15, guided use 15, review + exit 15.
    Live-coding order: show empty admin → open `admin.py` → write import + decorator + class → refresh → add row → filter → search → edit. Ask "staff tool or visitor page?" at each step.
    Core idea: register to reveal; customize to navigate.
    Misconceptions: admin is the website (it is staff-only), registration changes the database (it only changes browsing).
    Exit ticket: which attribute adds the search box?
    Support: pre-create the superuser and share credentials. Extension: add `date_hierarchy = "created_at"`.
    Troubleshooting: missing model → check file saved + server reloaded; login loop → check `AUTH` apps present; password error → lengthen it.
