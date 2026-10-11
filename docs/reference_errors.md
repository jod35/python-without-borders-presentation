# Common Errors

- `NoReverseMatch` — symptom: page crashes naming a route. Cause: misspelled `name` or missing `pk`. Fix: compare `{% url %}` with `urls.py`.
- `DoesNotExist` — symptom: shell/view crash on `get`. Cause: wrong pk. Fix: list `all()`, or use `get_object_or_404` in views (404 page is correct).
- `OperationalError: no such table` — cause: skipped `migrate`. Fix: run `migrate`.
- `403 CSRF verification failed` — cause: POST form lacks `{% csrf_token %}`. Fix: add it inside the form.
- `405 on logout` — cause: GET logout. Fix: use the POST button (correct behavior since Django 5).
- CSS 404 — cause: wrong static path. Fix: `tasks/static/tasks/style.css` + `{% load static %}`.
- `|lowercase` invalid — cause: old lesson typo. Fix: use `|lower`.
- Anonymous redirected to login with `?next=` — cause: `@login_required`. Fix: log in (correct behavior).
