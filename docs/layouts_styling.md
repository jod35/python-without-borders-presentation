# Layouts and Styling

!!! note "Trainer snapshot"
    Duration: 60 min. Prerequisites: Ch3 list + detail. Files: `tasks/templates/base.html`, `tasks/static/tasks/style.css`. Prepare: unstyled pages open for before/after. Start: two standalone pages. End: shared layout + CSS loading.

## What we are learning

1. How template inheritance shares page structure.
2. How blocks and navigation work.
3. How static files serve CSS.
4. How to keep a readable, simple responsive layout.

## Why this matters

Without a base template every page repeats the same header and footer. One layout file fixes all pages at once.

## A simple explanation before code

`base.html` holds the skeleton with named `{% block %}` holes. Child pages `{% extends %}` it and fill the holes. CSS lives in `static/` and is linked with `{% static %}`. Django finds it because `staticfiles` is installed and `STATIC_URL` is set.

Important terms: inheritance, block, `{% load static %}`, static files.

## Step-by-step example

### 1. Base template (20 min)

Create `tasks/tasks/templates/base.html`:

```html title="shared layout"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}TaskRat{% endblock %}</title>
    {% load static %}
    <link rel="stylesheet" href="{% static 'tasks/style.css' %}">
</head>
<body>
    <header>
        <nav>
            <a href="{% url 'home_page' %}">TaskRat</a>
            {% if user.is_authenticated %}
                <span>Hello, {{ user.username }}</span>
                <a href="{% url 'task_create' %}">Add task</a>
                <form method="post" action="{% url 'logout' %}" style="display:inline;">
                    {% csrf_token %}
                    <button type="submit">Log out</button>
                </form>
            {% else %}
                <a href="{% url 'login' %}">Log in</a>
                <a href="{% url 'signup' %}">Sign up</a>
            {% endif %}
        </nav>
    </header>
    <main>
        {% block content %}{% endblock %}
    </main>
    <footer>
        <p>&copy; TaskRat {% now 'Y' %}</p>
    </footer>
</body>
</html>
```

Auth links render conditionally; the full auth wiring arrives in Ch7. For now anonymous learners see Log in / Sign up links that will work once Ch7 routes exist.

### 2. Static CSS (15 min)

Create `tasks/tasks/static/tasks/style.css`:

```css title="minimal readable styling"
body {
    font-family: system-ui, sans-serif;
    max-width: 40rem;
    margin: 0 auto;
    padding: 0 1rem;
    line-height: 1.5;
}

header nav {
    display: flex;
    gap: 1rem;
    align-items: center;
    padding: 1rem 0;
    border-bottom: 1px solid #ccc;
}

main div {
    border: 1px solid #ddd;
    padding: 0.75rem;
    margin: 0.75rem 0;
}

footer {
    text-align: center;
    color: #555;
    margin-top: 2rem;
}

@media (max-width: 30rem) {
    header nav {
        flex-wrap: wrap;
    }
}
```

Reload with devtools network tab: `style.css` returns 200. No build step, no framework.

### 3. Children extend it (10 min)

Ch3 templates already start with `{% extends "base.html" %}` and fill `title` + `content`. Show that deleting the nav from `base.html` removes it everywhere: one file, all pages.

## Let's build it: follow-along practical (5 min)

Change the footer text once, reload two pages, confirm both changed. Narrow the browser to 400px and confirm the nav wraps.

## What the code is doing

`{% block %}` names a hole with default content. `{% extends %}` says "use that skeleton". `{% load static %}` enables the `{% static %}` tag that builds the CSS URL. The responsive rule only wraps the nav; nothing else to learn.

## Common mistakes and fixes

- CSS 404 — file in wrong folder. Path must be `tasks/static/tasks/style.css` with the doubled `tasks` segment.
- CSS not refreshing — hard refresh (Ctrl+F5 / Cmd+Shift+R).
- `Invalid block tag: 'static'` — forgot `{% load static %}`.
- `NoReverseMatch` for `task_create/login` — expected until Ch6/Ch7 routes exist; teach nav now, wire auth later.

## Try it yourself

Guided: add a `{% block %}` sidebar. Independent: style the search input to full width on small screens.

## Check your understanding

1. What does `{% extends %}` do?
2. What is a block?
3. How does `{% static %}` build a URL?
4. Where must app CSS live?

## Summary

One base template plus one small stylesheet gives every page the same nav, layout, and footer.

## Next lesson

We handle user input: HTML forms, GET versus POST, and CSRF.

!!! note "Trainer guide"
    Timed plan (adds to 60): base 20, CSS 15, children 10, guided 5, review + exit 10.
    Live-coding order: show duplicated headers → create `base.html` → convert one page → convert second → add CSS → 200 check → resize demo. Ask "how many files must change to edit the nav?" before and after.
    Core idea: skeleton once, fill holes per page.
    Misconceptions: CSS edits need migrate (no), blocks replace whole file (only named holes), static works without the app path (needs `tasks/` prefix).
    Exit ticket: which tag loads CSS URLs and which line enables it?
    Support: give the exact static path. Extension: add an `extra_css` block.
    Troubleshooting: 404 CSS → path; stale style → hard refresh; reverse errors → Ch6/Ch7 pending, note and continue.
