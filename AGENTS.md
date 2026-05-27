# AGENTS.md — AI Coding Agent Brief

> Read this file before proposing or writing any code for this repository.

## Stack

- **Framework**: Ruby on Rails 8 (sample todo app)
- **Database**: SQLite (development/test) — see `config/database.yml` for exact adapter
- **View layer**: ERB templates, Hotwire (Turbo + Stimulus), Bootstrap 5
- **Test framework**: RSpec (`spec/`) with FactoryBot and Capybara for system tests
- **Background jobs**: none currently (ActiveJob is available but no adapter configured)
- **Asset pipeline**: Propshaft + Importmap (no Webpack/Vite)

## Commands

```bash
bin/setup            # install gems, prepare DB, seed
bin/dev              # start Rails + CSS watcher (Foreman)
bundle exec rspec    # run full test suite
bundle exec rubocop  # lint Ruby files
bin/rails db:migrate # run pending migrations
bin/rails db:seed    # seed development data only
```

## Conventions

- Controllers respond with HTML by default; Turbo Stream responses use `format.turbo_stream` inside a `respond_to` block — never redirect after a Turbo Stream request.
- View partials live in `app/views/<resource>/` and are named with a leading underscore (e.g. `_todo.html.erb`). Turbo Stream views follow the pattern `<action>.turbo_stream.erb` in the same directory.
- Authorization lives in the controller via `before_action`; no policy objects yet.
- Shared/layout partials go in `app/views/shared/`.
- Use `bin/rails generate` for models, controllers, and migrations — never hand-write boilerplate.
- Strong parameters are defined in a private `<resource>_params` method in every controller.
- All routes are RESTful; avoid custom route names unless absolutely necessary.

## Don'ts

1. **No new gems without explicit approval** — check with the team before adding anything to `Gemfile`.
2. **No inline JavaScript in ERB** — use Stimulus controllers in `app/javascript/controllers/` instead.
3. **No `skip_before_action :verify_authenticity_token`** — CSRF protection must remain on for all non-API endpoints.
4. **Do not seed real user data** — use `db/seeds.rb` with fake/anonymous records only.
5. **Do not import models or migrations from other projects** — keep the schema scoped to this todo app.
6. **No string interpolation in SQL** — use ActiveRecord query methods or parameterized queries exclusively.
EOF