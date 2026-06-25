# Spec: Login and Logout

## Overview
Implement session-based login and logout so registered users can authenticate and access protected areas of Spendly. This step wires up `POST /login` (look up user, verify password, set session), `GET /logout` (clear session, redirect), and updates `base.html`'s navbar to show a "Sign out" link when the user is logged in and the current "Sign in / Get started" links when they are not. It also adds a `get_user_by_email()` DB helper and imports `check_password_hash` — the complement to Step 2's `generate_password_hash`.

## Depends on
- Step 1 — Database setup (`get_db()`, `users` table)
- Step 2 — Registration (`create_user()`, `app.secret_key`, `session` already imported)

## Routes
- `POST /login` — accepts email + password, verifies credentials, sets session, redirects — public
- `GET /logout` — clears session, redirects to `/login` — public (safe to call even if not logged in)

## Database changes
No new tables or columns. A new helper `get_user_by_email(email)` is added to `database/db.py` to fetch a user row by email for credential verification.

## Templates
- **Modify:** `templates/login.html`
  - Fix `action="/login"` → `action="{{ url_for('login') }}"`
  - Replace `{% if error %}` block with `get_flashed_messages(with_categories=true)` pattern (same as `register.html`)
- **Modify:** `templates/base.html`
  - Navbar: when `session.get('user_id')` is set, show a "Sign out" link (`url_for('logout')`); otherwise show existing "Sign in" and "Get started" links

## Files to change
- `database/db.py` — add `get_user_by_email(email)`
- `app.py` — add `check_password_hash` import, rewrite `GET /login` → `GET+POST /login`, implement `GET /logout`
- `templates/login.html` — fix form action, replace `{{ error }}` with flash messages
- `templates/base.html` — conditional navbar based on session state

## Files to create
None.

## New dependencies
No new pip packages. Uses `werkzeug.security.check_password_hash` (already installed) and Flask's built-in `session` (already imported in `app.py`).

## Rules for implementation
- No SQLAlchemy or ORMs — raw `sqlite3` only
- Parameterised queries only — never f-strings in SQL
- Passwords verified with `werkzeug.security.check_password_hash` — never compare plaintext
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `get_user_by_email()` lives in `database/db.py`, never inline in the route
- On successful login: set `session['user_id']` and `session['user_name']`, then redirect to `url_for('profile')` (stub, implemented in Step 4)
- On failed login (wrong email or wrong password): show the same generic flash error "Invalid email or password." — do not distinguish between the two (prevents user enumeration)
- `GET /logout` must call `session.clear()` then redirect to `url_for('login')` — never render a template
- The `POST /login` route function must do exactly: validate → look up → verify → set session → redirect. Nothing else.

## Definition of done
- [ ] `GET /login` renders the login form
- [ ] `POST /login` with correct credentials sets a session and redirects to `/profile`
- [ ] `POST /login` with wrong password shows "Invalid email or password." (no 500, no user enumeration)
- [ ] `POST /login` with unknown email shows "Invalid email or password." (same message)
- [ ] `POST /login` with empty fields shows "All fields are required."
- [ ] `GET /logout` clears the session and redirects to `/login`
- [ ] After logout, revisiting `/logout` again redirects cleanly (session already empty — no crash)
- [ ] Navbar shows "Sign out" link when logged in, "Sign in / Get started" when not
- [ ] No DB logic exists inside the route functions in `app.py`
