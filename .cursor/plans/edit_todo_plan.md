---
name: Edit Todo Ownership
overview: Add user ownership to todos so only the creator can edit their own todo, hiding the edit button from other authenticated users. Requires introducing authentication (Devise) and associating todos with users.
todos:
  - id: devise-gem
    content: Add Devise gem and run devise:install + devise User generator
    status: pending
  - id: migration-user-ref
    content: Generate and run AddUserRefToTodos migration
    status: pending
  - id: model-association
    content: Add belongs_to :user to Todo model
    status: pending
  - id: app-controller-auth
    content: Add authenticate_user! before_action to ApplicationController
    status: pending
  - id: create-ownership
    content: Build todo from current_user.todos in create action
    status: pending
  - id: authorize-owner
    content: Add authorize_owner! guard to edit and update actions
    status: pending
  - id: view-conditional
    content: Wrap edit link in current_user == @todo.user conditional in show view
    status: pending
  - id: fixtures
    content: Add users.yml fixture; link todos fixture to alice
    status: pending
  - id: update-existing-tests
    content: Add Devise test helpers and sign_in to existing controller tests
    status: pending
  - id: new-auth-tests
    content: Add owner vs. non-owner request-level tests for edit/update
    status: pending
  - id: system-test
    content: Add system test asserting edit button is absent for non-owner
    status: pending
isProject: false
---

# Edit Todo — Owner-Only Access Plan

## Prerequisites

The app has no `User` model, no authentication, and no `user_id` on `todos`. All steps below are required. Adding Devise requires explicit team approval per `AGENTS.md`.

## Files Involved

- [`Gemfile`](Gemfile) — add `devise` gem (requires team approval)
- [`db/schema.rb`](db/schema.rb) — will reflect two new migrations
- [`app/models/todo.rb`](app/models/todo.rb)
- [`app/controllers/application_controller.rb`](app/controllers/application_controller.rb)
- [`app/controllers/todos_controller.rb`](app/controllers/todos_controller.rb)
- [`app/views/todos/show.html.erb`](app/views/todos/show.html.erb) — the only view with an edit link
- [`test/controllers/todos_controller_test.rb`](test/controllers/todos_controller_test.rb)
- [`test/system/todos_test.rb`](test/system/todos_test.rb)
- [`test/fixtures/todos.yml`](test/fixtures/todos.yml) and a new `test/fixtures/users.yml`

---

## Numbered Changes

1. **Add Devise** — Add `gem "devise"` to [`Gemfile`](Gemfile), run `bundle install`, then `bin/rails generate devise:install` and `bin/rails generate devise User`. This produces a `users` migration, the `User` model, and Devise routes. Run `bin/rails db:migrate`.

2. **Associate todos with users (migration)** — Run `bin/rails generate migration AddUserRefToTodos user:references`. This adds a `user_id` foreign key column to the `todos` table and a NOT NULL constraint. Run `bin/rails db:migrate`.

3. **Update the `Todo` model** — Add `belongs_to :user` to [`app/models/todo.rb`](app/models/todo.rb).

4. **Update `ApplicationController`** — Add `before_action :authenticate_user!` to [`app/controllers/application_controller.rb`](app/controllers/application_controller.rb) so all todo actions require a signed-in user.

5. **Stamp ownership on create** — In the `create` action of [`app/controllers/todos_controller.rb`](app/controllers/todos_controller.rb), build the todo from `current_user.todos.build(todo_params)` instead of `Todo.new(todo_params)` so the creator is recorded automatically.

6. **Guard `edit` and `update` in the controller** — Add a private `authorize_owner!` method to [`app/controllers/todos_controller.rb`](app/controllers/todos_controller.rb) that returns `head :forbidden` unless `@todo.user == current_user`. Call it via `before_action :authorize_owner!, only: [:edit, :update]`.

7. **Conditionally render the edit link in the view** — In [`app/views/todos/show.html.erb`](app/views/todos/show.html.erb), wrap `link_to "Edit this todo", edit_todo_path(@todo)` in `<% if current_user == @todo.user %>` so the link is invisible to non-owners.

8. **Update fixtures** — Add a `users.yml` fixture with at least two users (`alice`, `bob`). Update [`test/fixtures/todos.yml`](test/fixtures/todos.yml) to set `user: alice` on the existing fixtures so the foreign key is satisfied.

9. **Update existing controller tests** — [`test/controllers/todos_controller_test.rb`](test/controllers/todos_controller_test.rb) currently assumes no auth. Add Devise test helpers (`include Devise::Test::IntegrationHelpers`) and a `sign_in users(:alice)` call in `setup` so existing tests continue to pass with auth enabled.

10. **Add new authorization tests (controller/request level)** — In [`test/controllers/todos_controller_test.rb`](test/controllers/todos_controller_test.rb), add:
    - A test where `bob` (non-owner) sends `GET /todos/:id/edit` for Alice's todo → asserts `403 Forbidden`.
    - A test where `bob` sends `PATCH /todos/:id` for Alice's todo → asserts `403 Forbidden`.
    - A test where `alice` (owner) sends those same requests → asserts success/redirect.

11. **Add a system test for the hidden edit button** — In [`test/system/todos_test.rb`](test/system/todos_test.rb), add a test that signs in as `bob`, visits the show page for Alice's todo, and asserts `assert_no_selector` for the "Edit this todo" link.
