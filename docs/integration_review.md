# Integration Project and Final Review

!!! note "Trainer snapshot"
    Duration: 60 min + 90 min final assessment. Prerequisites: Ch1–Ch7. Files: whole `tasks/`. Prepare: seeded DB, checklist printed. Start: all features built. End: flow traceable, gaps listed, assessment submitted.

## What we are learning

1. How a browser request becomes a response through URL, view, database, and template.
2. How all course features fit in one project.
3. What remains weak and how to fix it.

## Why this matters

Learners leave with one coherent story instead of seven disconnected tricks.

## The full flow (20 min)

Trace creating a task out loud: POST `/tasks/new/` → `tasks/urls.py` matches `task_create` → `@login_required` passes → `TaskForm(request.POST)` validates → `form.save()` writes the row → `redirect("task_detail")` → `task_detail` queries → `detail.html` renders → 302 then 200. Repeat for search (GET, no write), delete (GET confirm, POST act), and anonymous create (302 to login with `?next=`).

Acceptance for the course project: list with search + empty state, detail with 404, shared layout + CSS, admin with filter/search, ModelForm create/update with errors + redirect, POST-only delete confirm, signup/login/POST-logout, protected writes, auth-aware nav. All verified in validation.

## Guided review (20 min)

Run the `reference/checklist.md` drill: migrate from scratch on a copy DB, create superuser, add two tasks in admin, search, create via form, submit invalid, edit, delete confirm + act, sign up a second user, prove cross-editing is possible (the documented limit), log out.

## Try it yourself + assessments

Assessments live in `assessments.md`: A1 ORM+admin (45 min), A2 CRUD+ModelForms (60 min), A3 final project (90 min) with rubrics and trainer keys. Remediation for each is listed there.

## Check your understanding

1. Name every step from address bar to rendered page for a valid create.
2. Where does validation happen?
3. Why does delete need two requests?
4. What does login protect here, and what does it not protect?

## Summary

One project, one flow, honest limits. You can now build, verify, and explain a beginner Django CRUD site with auth.

!!! note "Trainer guide"
    Timed plan (adds to 60): flow trace 20, guided drill 20, assessment brief 10, exit 10. A3 runs separately (90).
    Core idea: say each hop before clicking. Misconceptions: template queries (view does), redirect optional (prevents resubmit), login equals ownership (no).
    Exit ticket: write the hop list for delete. Support: flow diagram on board. Extension: propose an ownership rule (`owner = ForeignKey(User)`) as future work without implementing.
    Troubleshooting: see `reference/errors.md` symptom index.
