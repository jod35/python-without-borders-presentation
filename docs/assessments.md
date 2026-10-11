# Assessments

## A1. ORM and Admin Checkpoint (45 min, after Ch2)

Starting checkpoint: Ch1 migration applied. Instructions: in the shell create 4 tasks (mixed priorities, one done), then in the admin find them via search + priority filter. Acceptance: `filter/exclude/get/count` each demonstrated; admin list shows columns; search returns the right row; `get` on a bad pk raises `DoesNotExist` (screenshot or pasted traceback). Rubric (10): shell CRUD 4, filter/exclude 2, get-vs-filter explanation 2, admin register/customize 2.

!!! note "Trainer key A1"
    Accept any sensible task names. `created` flag from `get_or_create` must be explained. Remediation: repeat Ch1 guided queries with `values_list` inspection.

## A2. CRUD with ModelForms (60 min, after Ch6)

Starting checkpoint: Ch5 code. Instructions: build `TaskForm`, wire create/update/confirm-delete per Ch6 (may copy structure, not blind-paste). Acceptance: valid create redirects to detail; empty name re-renders with "This field is required"; edit persists; GET on delete URL does not delete; POST deletes and redirects to list; CSRF token present. Rubric (15): form class 3, create+redirect 3, invalid re-render 3, update 2, delete two-step 3, CSRF 1.

!!! note "Trainer key A2"
    Alternatives accepted: function names may differ if routes/templates agree; `fields` order flexible. Remediation: Ch6 live-coding replay focusing on `is_valid` gate + method branch.

## A3. Final Project (90 min, after Ch8)

Starting checkpoint: full Ch7 code. Instructions: from a fresh temp DB (`migrate`), demonstrate: seeded list with search + empty state, detail + 404, styled shared layout, admin filter/search, form create/update with error case, POST delete, signup/login/logout, protected writes (anonymous redirect keeps `next`), auth-aware nav. Acceptance: every item above works; learner narrates the request→response hops for one create and one delete; states the login-vs-ownership limit. Rubric (25): display 5, forms/CRUD 8, admin 3, auth 6, narration + limits 3.

!!! note "Trainer key A3"
    Alternatives: CSS values free; route names free if consistent. Remediation: `reference/checklist.md` drill + Ch8 re-trace. Extension: propose (not build) an `owner` FK + queryset scoping.
