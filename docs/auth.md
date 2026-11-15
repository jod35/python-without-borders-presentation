# Authentication: Login, Logout, Signup


## What we are learning

1. How Django's built-in authentication works.
2. How to add login, logout, and signup with `UserCreationForm`.
3. How to protect views with `login_required`.
4. How to show login state, and what login alone does not do.

## Why this matters

Creates, edits, and deletes currently redirect anonymous users to login without explanation. This chapter wires that up on purpose and states its limits honestly.

## A simple explanation before code

Authentication answers "who are you". Authorization answers "what may you do". Django ships both the user table and the login machinery; we add three routes and two templates. Hiding a link never protects a view: only the decorator on the view does.

Important terms: authentication, authorization, `login_required`, `UserCreationForm`.

## Step-by-step example

### 1. Routes (20 min)

Replace `tasks/mysite/urls.py` with:

```py title="project urls with accounts"
# inside mysite/urls.py (replaces the two-route version)
from django.contrib import admin
from django.contrib.auth import views as auth_views
from django.urls import path, include

from tasks.auth_views import signup

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/login/', auth_views.LoginView.as_view(template_name='registration/login.html'), name='login'),
    path('accounts/logout/', auth_views.LogoutView.as_view(), name='logout'),
    path('accounts/signup/', signup, name='signup'),
    path('', include('tasks.urls'))
]
```

`LoginView`/`LogoutView` are built-in class-based views we reuse as-is. You do not need to master class-based views in this course; treat these two as prebuilt login machines. (Appendix note at the end says where `ListView`/`CreateView` would fit later.)

### 2. Signup view (15 min)

Create `tasks/tasks/auth_views.py`:

```py title="signup with UserCreationForm"
# inside tasks/auth_views.py (new file)
from django.contrib.auth.forms import UserCreationForm
from django.shortcuts import redirect, render


def signup(request):
    if request.method == "POST":
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("login")
    else:
        form = UserCreationForm()
    return render(request, "registration/signup.html", {"form": form})
```

Passwords are hashed by Django. Never store or compare plain-text passwords yourself.

### 3. Templates and settings (15 min)

`tasks/templates/registration/login.html` and `signup.html` each extend `base.html` and post with `{% csrf_token %}` plus `{{ form.as_p }}`. In `tasks/mysite/settings.py` add:

```py title="auth redirects"
LOGIN_URL = 'login'
LOGIN_REDIRECT_URL = 'home_page'
LOGOUT_REDIRECT_URL = 'home_page'
```

Logout in `base.html` is a small POST form (Django 5+ rejects GET logout), which is why the nav shows a button, not a link.

### 4. Protect and display state (15 min)

Ch6 views already carry `@login_required` on create/update/delete: anonymous visitors redirect to `/accounts/login/?next=...` and return after login. `base.html` shows "Hello, user + Add task + Log out button" when `{% if user.is_authenticated %}`, else "Log in + Sign up". Demo both windows side by side. Then state the limit: any logged-in user can edit any task. Login is not ownership or per-object permission.

## Let's build it: follow-along practical (10 min)

As anonymous: open `/tasks/new/` (lands on login). Sign up, log in, create a task. Log out via the button (POST), confirm the nav flips. Try GET on logout URL (rejected) to prove POST-only.

## What the code is doing

`LOGIN_URL` tells the decorator where to send strangers. `LOGIN_REDIRECT_URL` picks the landing page after login. `UserCreationForm` validates username + double password + hashes on save. The `user` variable in templates comes from the `auth` context processor (already installed).

## Common mistakes and fixes

- GET logout 405 — correct behavior; use the POST button.
- Redirect loop after login — `LOGIN_REDIRECT_URL` misspelled; compare with route names.
- Signup saves but cannot log in — password validators rejected it; read the form errors.
- "I hid the button so it is safe" — hiding the link changes nothing; the decorator protects the view.

## Try it yourself

Guided: protect a view, prove anonymous redirect keeps `?next=`. Independent: write two sentences explaining auth versus authz to a friend.

## Check your understanding

1. What is the difference between authentication and authorization?
2. What does `@login_required` do to anonymous users?
3. Why is logout a POST?
4. How does a template know the user?
5. Does login stop one user editing another user's task?

## Summary

Built-in auth for identity, decorator for protection, template check for display, honesty about limits: login alone is not ownership.

## Next lesson

We trace the whole flow and take the final assessment.

