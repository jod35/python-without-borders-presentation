# Glossary

- Project: whole website (`mysite`). App: one functionality (`tasks`).
- Model class: table recipe. Instance: one row.
- Manager (`Task.objects`): takes queries. QuerySet: tray of results.
- URL pattern: address → view mapping. View: Python deciding data. Template: HTML deciding display. Context: data dict passed to template.
- HTML form: browser boxes. ModelForm: boxes wired to a model with validation + save.
- GET: reads, safe, bookmarkable. POST: writes, needs CSRF.
- Authentication: who you are. Authorization: what you may do. Hiding a link displays less; only the view decorator protects.
- Migration: file describing structure change (`makemigrations` writes, `migrate` applies).
