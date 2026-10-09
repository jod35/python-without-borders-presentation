Here is a shorter, cleaned-up version of your lesson. I have kept the beginner-friendly tone, improved the explanations, removed repetition, and corrected the English while preserving the important concepts.

Working with Django Models

# Working with Django Models

Things are about to get interesting. Let us look at models, Django's way of working with databases.

## What is a database model?

A database model describes the structure of the data an application needs to store. In Django, models are Python classes that define the fields of a database table and provide methods for creating, retrieving, updating, and deleting records without writing SQL queries manually.

## Creating a database model

Every Django app has a `models.py` file where you define its database models. In `tasks/models.py`, let us create our first model.

### CharField vs TextField

```
from django.db import models


class Task(models.Model):
    name = models.CharField(max_length=150)
    description = models.TextField()
```

Our `Task` class inherits from `models.Model`, the base class for Django models. Each field is defined as a class attribute.

* `CharField` stores short text with a specified maximum length. Here, `name` can contain up to 150 characters.

* `TextField` stores longer text, such as a task description. It does not require a `max_length` argument.

### Using TextChoices

Tasks can have either a low or high priority. Django's `TextChoices` lets us define these predefined values.

```
class Task(models.Model):

    class TaskPriority(models.TextChoices):
        LOW = "LOW"
        HIGH = "HIGH"

    # Other fields go here

    priority = models.CharField(
        max_length=10,
        choices=TaskPriority.choices,
        default=TaskPriority.HIGH,
    )
```

We define `TaskPriority` inside the model and use it to specify the allowed priority choices. The `default` argument sets a task's priority to `HIGH` unless another value is provided.

### Using BooleanField

We also need to know whether a task has been completed. A `BooleanField` stores either `True` or `False`.

```
done = models.BooleanField(default=False)
```

By default, a new task is marked as incomplete.

### Tracking creation and updates

Django provides `DateTimeField` for storing dates and times. We can use it to track when a task was created and last updated.

```
created_at = models.DateTimeField(auto_now_add=True)
updated_at = models.DateTimeField(auto_now=True)
```

* `auto_now_add=True` sets the creation timestamp when the object is first created.

* `auto_now=True` updates the timestamp whenever the object is saved.

### The complete Task model

Putting everything together, our `tasks/models.py` should look like this:

```
from django.db import models


class Task(models.Model):

    class TaskPriority(models.TextChoices):
        LOW = "LOW"
        HIGH = "HIGH"

    name = models.CharField(max_length=150)
    description = models.TextField()
    priority = models.CharField(
        max_length=10,
        choices=TaskPriority.choices,
        default=TaskPriority.HIGH,
    )
    done = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

## Applying database migrations

Creating a model does not automatically create its table in the database. We need to use **migrations** to apply our model changes.

Django's default database configuration is in `mysite/settings.py`. By default, Django uses SQLite and stores the database in a file called `db.sqlite3`.

```
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}
```

### 1. Apply the default migrations

When you create a Django project, it comes with built-in apps such as authentication, the admin interface, and sessions. Run the following command to apply their initial migrations:

```
python manage.py migrate
```

This creates the necessary database tables for the built-in apps.

A migration is a file that describes a change to your database structure, such as creating a table or adding a field.

### 2. Create migrations for our model

Now let us create a migration for the `Task` model we defined.

```
python manage.py makemigrations
```

Django detects the changes to your models and creates a migration file in `tasks/migrations/`. This file describes the database changes needed to create our `Task` table.

**Important:** `makemigrations` creates the migration file, but it does not apply the changes to the database. To create the table, run:

```
python manage.py migrate
```

Django will then apply the new migration and create the table for our `Task` model.
