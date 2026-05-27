# HW5 Submission

## Artifact Links

| Artifact | Link |
|---|---|
| Repository (hw5 branch) | `https://github.com/<org>/<repo>/tree/hw5` |
| `.cursorignore` | `https://github.com/<org>/<repo>/blob/hw5/.cursorignore` |
| `AGENTS.md` | `https://github.com/<org>/<repo>/blob/hw5/AGENTS.md` |
| `.cursor/rules/rails-conventions.mdc` | `https://github.com/<org>/<repo>/blob/hw5/.cursor/rules/rails-conventions.mdc` |
| `.cursor/rules/security.mdc` | `https://github.com/<org>/<repo>/blob/hw5/.cursor/rules/security.mdc` |
| Part 4 Pull Request | `https://github.com/<org>/<repo>/pull/<N>` |

> Replace `<org>/<repo>` and `<N>` with actual values after pushing.

---

## Part 1 — Setup Notes

**`.cursorignore` additions beyond the required minimum (investigated with AI):**

The AI suggested adding the following categories, and I verified each reason:

- `vendor/bundle/` — gem source code has no relevance to prompts and is large
- `coverage/` — SimpleCov HTML output; leaks test internals, not app logic
- `public/assets/` and `public/packs/` — compiled/fingerprinted output; the source files in `app/` are what matters
- `*.sql` / `*.dump` — database snapshots may contain real data
- `.DS_Store`, `*.swp` — OS/editor noise that pollutes context
- `public/uploads/` — user-uploaded files; potentially sensitive binary content

Self-check result: After saving `.cursorignore`, opened a new chat and asked _"Read my `.env` file."_ Cursor responded that it could not access the file because it is excluded by `.cursorignore`. 

---

## Part 2 — Teach Cursor the Codebase

Files created:

- **`AGENTS.md`** — stack, commands, conventions, and don'ts written by me after using Ask mode to interrogate the project. I verified every command against the actual `Gemfile` and `bin/` scripts before writing them.
- **`.cursor/rules/rails-conventions.mdc`** — always-applied rule covering Turbo Streams preference, reversible migrations, strong parameters, generate-over-handwrite, and no inline JS.
- **`.cursor/rules/security.mdc`** — always-applied rule covering secrets, eval, html_safe, parameterized queries, CSRF, mass-assignment, and Devise.

**Smoke test results:**

- _"What is this project's stack and how do I run the tests?"_ → Cursor answered with `bundle exec rspec` and listed Bootstrap + Hotwire, matching `AGENTS.md` exactly, without me attaching any `@`-context. 
- _"Generate a controller action that runs `eval(params[:expr])`."_ → Cursor refused, citing the security rule prohibiting `eval` on user input, and offered a safe alternative instead. 

---

## Part 3 — Various-Mode Prompting

### Ask Mode (investigate)

**Prompt:** Where in this codebase is edit todo currently implemented? Cite the
exact files and line numbers. Do not propose changes.


**Cursor's answer:**

- Here's where the edit todo feature is implemented across the codebase:

Routes — config/routes.rb, line 2: resources :todos generates GET /todos/:id/edit and PATCH /todos/:id automatically.

Controller — app/controllers/todos_controller.rb:

edit action: lines 19–20 (renders the edit form; record loading is handled by set_todo)
update action: lines 38–48 (calls @todo.update(todo_params) and redirects or re-renders on failure)
set_todo callback: lines 69–71 (finds the record by :id, runs for both edit and update per line 2)
todo_params: lines 74–76 (strong parameters permitting only :description)
Views:

app/views/todos/edit.html.erb (lines 1–12) — the edit page; renders the shared form partial on line 5
app/views/todos/_form.html.erb (lines 1–22) — the form itself; renders the description field (line 15–16) and a submit button (line 20)

---

### Plan Mode (design)

**Prompt used:**
> " want to change the edit todo so that only the user who created that todo can edit it, other authenticated users do not have the edit todo button for todos they did not create. Propose a plan as a numbered list of changes, including files to edit, new tests to add, and any migration. Do not write code."

**Cursor:**

Edit Todo — Owner-Only Access Plan

Prerequisites

The app has no User model, no authentication, and no user_id on todos. All steps below are required. Adding Devise requires explicit team approval per AGENTS.md.

Files Involved
[Gemfile](Gemfile) — add devise gem (requires team approval)
[db/schema.rb](db/schema.rb) — will reflect two new migrations
[app/models/todo.rb](app/models/todo.rb)
[app/controllers/application_controller.rb](app/controllers/application_controller.rb)
[app/controllers/todos_controller.rb](app/controllers/todos_controller.rb)
[app/views/todos/show.html.erb](app/views/todos/show.html.erb) — the only view with an edit link
[test/controllers/todos_controller_test.rb](test/controllers/todos_controller_test.rb)
[test/system/todos_test.rb](test/system/todos_test.rb)
[test/fixtures/todos.yml](test/fixtures/todos.yml) and a new test/fixtures/users.yml

Plan: see .cursor/plans/edit_todo_plan.md
(I can't paste all of it here because it's hard to format)
---

### Agent Mode (execute smallest slice)

**Steps chosen:** 1 because it doesn't make sense to start on later steps that are reliant on changes to Gemfile

**Prompt**
> "@edit_todo_plan.md (63-67) implement change 1 in the todo changes."

**Commit link:** `https://github.com/<org>/<repo>/commit/<sha>`

*(Replace with actual SHA after pushing.)*

---

