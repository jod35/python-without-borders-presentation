# HTML Forms and HTTP Methods

!!! note "Trainer snapshot"
    Duration: 60 min. Prerequisites: Ch3 list view. Files: `tasks/tasks/views.py` (search lines). Prepare: rows with distinct names for search demo. Start: plain list. End: safe GET filter understood, POST/CSRF rules stated.

## What we are learning

1. How the request/response cycle works.
2. When to use GET versus POST.
3. What CSRF protection does.
4. How validation errors should be shown (preview of ModelForms).

## Why this matters

Every write in the next two chapters depends on choosing the right method. Wrong method means lost data, scary warnings, or security holes.

## A simple explanation before code

The browser sends a request, Django returns a response. GET asks for data and must never change it: search, filter, pagination. POST changes data: create, update, delete. Django demands a `{% csrf_token %}` inside every POST form so strangers cannot forge submissions from other sites.

Important terms: request, response, GET, POST, CSRF, validation.

## Step-by-step example

### 1. The cycle (10 min)

Browser → URL pattern → view → (database) → template → response. Say it for the search form: submit reloads `/` with `?q=milk`; the view reads `request.GET`; the response is the same list, filtered. Nothing was stored.

### 2. Safe GET filter (20 min)

Already in `tasks/tasks/views.py`:

```py title="GET filter (already in place)"
def index(request):
    query = request.GET.get("q", "").strip()
    tasks = Task.objects.all()
    if query:
        tasks = tasks.filter(name__icontains=query)
    return render(request, "index.html", {"tasks": tasks, "query": query})
```

And in the template:

```html title="search form (already in place)"
<form method="get" action="{% url 'home_page' %}">
    <input type="search" name="q" placeholder="Search tasks" value="{{ query }}">
    <button type="submit">Search</button>
</form>
```

`method="get"` puts the term in the address bar, so results are bookmarkable and refresh-safe. No `csrf_token` is needed for GET.

### 3. POST rules (15 min)

State the rules before Ch6 implements them: POST forms need `{% csrf_token %}`; the view checks `request.method == "POST"`; invalid input re-renders the form with errors instead of saving; delete confirmations render on GET but delete only on POST. Never change data through GET: a crawler or refresh could wipe rows.

## Let's build it: follow-along practical (5 min)

Search `milk`, bookmark the URL, reload. Confirm the address holds `?q=milk` and no data changed in the admin.

## What the code is doing

`request.GET.get("q", "")` reads the query string safely with a default. `.strip()` ignores accidental spaces. `icontains` matches case-insensitively. The template echoes `{{ query }}` so the term survives reloads.

## Common mistakes and fixes

- Using POST for search — works but breaks bookmarks and back-button; switch to GET.
- Using GET for delete — dangerous; Ch6 uses POST-only delete.
- 403 CSRF failure on POST — missing `{% csrf_token %}` inside the form.
- Empty search returning everything — that is the designed default (`all()` when no `q`).

## Try it yourself

Guided: filter done-only tasks via a second GET parameter. Independent: explain in two sentences why the search URL is shareable but a POST submission is not.

## Check your understanding

1. What is the request/response cycle?
2. When do you use GET?
3. When do you use POST?
4. What does `{% csrf_token %}` protect against?
5. Why must deletes never use GET?

## Summary

GET reads and is safe to repeat. POST writes and needs CSRF. Validation errors belong back in the form.

## Next lesson

We build ModelForms that create, update, and delete tasks through POST.

!!! note "Trainer guide"
    Timed plan (adds to 60): cycle 10, GET live 20, POST rules 15, guided 5, review + exit 10.
    Live-coding order: submit search → show `?q=` URL → read `request.GET` line → clear box → show unfiltered → state POST rules on board without coding them. Ask "did anything change in the admin?" after each search.
    Core idea: safe methods read, unsafe methods write with a token.
    Misconceptions: CSRF needed for GET (only POST), POST data in URL (only GET), validation is optional (re-render with errors is required).
    Exit ticket: is "delete this task" a GET or POST, and why?
    Support: keep the cycle diagram visible. Extension: add `?done=1` filter.
    Troubleshooting: 403 → token; lost search term → echo `{{ query }}`; everything matches → `icontains` + empty default.
