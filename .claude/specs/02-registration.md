# Spec: Registration

## Overview
Implement user registration so new visitors can create an account. This step wires up the `POST /register` route, adds a `create_user()` helper to the data layer, and turns the existing static `register.html` form into a working form that validates input, hashes the password, and redirects to the dashboard (or login) on success. It also sets `app.secret_key` — required for Flask sessions and flash messages — which all authenticated steps depend on.

## Depends on
- Step 1 — Database setup (`get_db()`, `init_db()`, `users` table must exist)

## Routes
- `POST /register` — accepts form data, validates, inserts user, redirects — public

## Database changes
No new tables or columns. The `users` table created in Step 1 already has all required columns (`name`, `email`, `password_hash`, `created_at`). A new helper function `create_user()` is added to `database/db.py`.

## Templates
- **Modify:** `templates/register.html`
  - Add `method="POST"` and `action="{{ url_for('register') }}"` to the `<form>`
  - Add `name` attributes to all inputs (`name`, `email`, `password`, `confirm_password`)
  - Display flashed error/success messages using `get_flashed_messages()`
  - Include CSRF-safe hidden field with `{{ csrf_token() }}` if available, otherwise omit (no new packages)

## Files to change
- `app.py` — add `secret_key`, import `session`/`flash`/`redirect`/`url_for` from Flask, implement `POST /register` route
- `database/db.py` — add `create_user(name, email, password_hash)` helper
- `templates/register.html` — wire up form as described above

## Files to create
None.

## New dependencies
No new pip packages. Uses `werkzeug.security.generate_password_hash` (already installed) and Flask's built-in `flash`, `session`, `redirect`.

## Rules for implementation
- No SQLAlchemy or ORMs — raw `sqlite3` only
- Parameterised queries only — never f-strings in SQL
- Passwords hashed with `werkzeug.security.generate_password_hash` before insert
- Use CSS variables — never hardcode hex values in any style changes
- All templates extend `base.html`
- `secret_key` must be set on `app` before any `session` or `flash` usage — use a hard-coded dev string for now (e.g. `"dev-secret-change-in-prod"`)
- `create_user()` lives in `database/db.py`, never inline in the route
- Duplicate email must be caught and shown as a user-facing flash error, not an unhandled 500
- After successful registration, redirect to `/login` using `url_for('login')` — do not render a template directly
- The `POST /register` route function must do exactly: validate → insert → flash → redirect. Nothing else.

## Definition of done
- [ ] Submitting the registration form with valid data creates a new row in `users`
- [ ] The stored password is a werkzeug hash, not plaintext
- [ ] Submitting with an already-registered email shows an error message on the page (no 500)
- [ ] Submitting with mismatched passwords shows a validation error on the page
- [ ] Submitting with any empty field shows a validation error on the page
- [ ] Successful registration redirects to `/login`
- [ ] The `GET /register` route still works and renders the form
- [ ] No DB logic exists inside the route function in `app.py`
