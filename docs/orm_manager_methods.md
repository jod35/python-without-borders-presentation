# Working with Models and Manager Methods

!!! note "Trainer snapshot"
    Duration: 90 min. Prerequisites: database models chapter, migrated project. Files: `tasks/tasks/models.py`, `tasks/tasks/migrations/0001_initial.py`. Prepare: clean `db.sqlite3` backup, shell commands copied. Start checkpoint: `Task` model exists, no `0001_initial.py`. End checkpoint: migration applied, `__str__` + ordering added, shell CRUD verified.

## What we are learning

1. The difference between a model class and a model instance.
2. How to create, read, update, and delete records in the Django shell.
3. How to use manager methods to query records.
4. How to handle common query errors.

## Why this matters

Everything in this course reads and writes `Task` rows. If you can drive the model from the shell, views and forms later are just the same calls wrapped in HTTP.

## A simple explanation before code

A model class is the recipe. A model instance is one meal made from it. The manager (`Task.objects`) is the kitchen counter where you ask for meals. A QuerySet is the tray you get back: a list-like group of instances you can filter and count.

Important terms: model class, instance, manager, QuerySet, migration.

## Step-by-step example

### 1. Recap and checkpoint (15 min)

Verify the project runs and the `Task` table exists. On Windows:

```title="checkpoint commands"
env\Scripts\activate
python manage.py check
python manage.py makemigrations tasks
python manage.py migrate
python manage.py shell
```

On macOS/Linux replace the first line with `source env/bin/activate`. `makemigrations` creates `tasks/migrations/0001_initial.py`. `migrate` creates the table. If `makemigrations` says "No changes detected", the migration already exists and you can move on.

### 2. Class versus instance (10 min)

The class lives in `tasks/tasks/models.py`:

```py title="the Task model class"
# inside tasks/models.py
class Task(models.Model):
    name = models.CharField(max_length=150)
    # ...
```

An instance is one row. Create one in the shell:

```py title="creating an instance"
from tasks.models import Task
t = Task(name="Write lesson", description="ORM chapter", priority="HIGH")
t.save()
t.pk  # now has an id, for example 1
```

`Task` (capital) is the class. `t` (lowercase) is the instance. `t.save()` writes the row.

### 3. Manager methods (25 min)

The manager is `Task.objects`. It returns QuerySets. Try each line and read the result:

```py title="common manager methods"
Task.objects.all()                        # every task
Task.objects.filter(priority="LOW")       # matching tasks
Task.objects.exclude(done=True)           # everything not done
Task.objects.order_by("name")             # sorted tray
Task.objects.count()                      # how many rows
Task.objects.create(name="Buy milk", description="2 litres", priority="LOW")
Task.objects.get_or_create(name="Buy milk", defaults={"description": "2 litres"})
```

`filter` returns a QuerySet (possibly empty). `get` returns exactly one instance. `create` builds and saves in one step. `get_or_create` returns a `(task, created)` pair: `True` if it just made the row.

### 4. get versus filter (10 min)

```py title="get returns one instance"
one = Task.objects.get(pk=1)
```

```py title="filter returns a QuerySet"
many = Task.objects.filter(priority="HIGH")
```

Two common exceptions: `Task.DoesNotExist` when `get` finds nothing, and `Task.MultipleObjectsReturned` when `get` matches more than one row. `filter` never raises these; it returns an empty QuerySet instead. In views we will use `get_object_or_404` so a missing row becomes a 404 page instead of a crash.

### 5. Update and delete (10 min)

```py title="update and delete"
t = Task.objects.get(pk=1)
t.done = True
t.save()
t.delete()
```

## Let's build it: follow-along practical (10 min)

Create two tasks, list high-priority ones ordered by name, mark one done, count what is left undone. Read every return value out loud before moving on.

## What the code is doing

`tasks/tasks/models.py` also gained two small additions:

```py title="ordering and display name"
# inside tasks/models.py
class Task(models.Model):
    # ... fields unchanged ...
    class Meta:
        ordering = ["-created_at"]

    def __str__(self):
        return self.name
```

`Meta.ordering` sorts newest first everywhere. `__str__` controls the display name in the shell and admin. Field names are unchanged so earlier lessons still match.

## Common mistakes and fixes

- `NameError: Task` — you forgot `from tasks.models import Task` in the shell.
- `OperationalError: no such table` — you skipped `migrate`. Run it, then retry.
- `DoesNotExist` on `get` — the pk is wrong or the row was deleted. List `all()` to see real ids.
- Confusing `filter` output `<QuerySet []>` with an error — it is not an error, it means zero matches.

## Try it yourself

Guided: create three tasks with different priorities, then write one `filter`, one `exclude`, and one `order_by` that each return a non-empty result. Independent: use `get_or_create` twice with the same name and explain why `created` is `True` then `False`.

## Check your understanding

1. What is the difference between `Task` and `t = Task(...)`?
2. What does `Task.objects` return calls on, and what do those calls return?
3. When does `get` raise `DoesNotExist`?
4. What does `get_or_create` return?
5. Why do we run `makemigrations` before `migrate`?

## Summary

Model class is the recipe, instance is one row, manager takes requests, QuerySet is the tray of results. `filter` is safe for zero-or-many; `get` demands exactly one.

## Next lesson

We register `Task` in the admin so we can inspect these same rows in the browser.

!!! note "Trainer guide"
    Timed plan (adds to 90): recap/checkpoint 15, class-vs-instance 10, live-coding managers 25, get-vs-filter 10, update/delete 10, guided practice 10, review + exit ticket 10.
    Live-coding order: `check` → `makemigrations` (show the new file) → `migrate` → `shell` → create → all/filter/exclude/order/count → get vs filter + both exceptions → update → delete. Pause after each return value and ask "what type is this: instance, QuerySet, or boolean?".
    Core idea: manager in, QuerySet out; `get` is strict, `filter` is forgiving.
    Misconceptions: "saving the class" (only instances save), "`filter` failed because it printed empty" (empty is valid), "migration edits data" (it edits structure).
    Exit ticket: write one `get` and one `filter` for a task named "Demo"; predict the return type of each.
    Support: give the exact import line. Standard: guided queries above. Extension: chain `filter(priority="HIGH").exclude(done=True).order_by("name")` and explain the SQL-free reading.
    Troubleshooting: no-table error → `migrate`; shell import error → check venv + `cd tasks` project root; stale rows confuse counts → `Task.objects.all().values_list("name", flat=True)` to see reality.
