# Spec: Profile Page Design

## Overview
Implement the `/profile` page so that logged-in users land on a personalised dashboard after signing in. The page displays the user's name and email, a summary card showing total spending and expense count, and a table of their most recent expenses. It also adds a login-required guard so unauthenticated visitors are redirected to `/login`. This step converts the raw-string stub at `GET /profile` into a real rendered page and is the first screen users see after authentication — it anchors the visual identity of the app.

## Depends on
- Step 1 — Database setup (`get_db()`, `users` and `expenses` tables)
- Step 2 — Registration (`users` rows exist)
- Step 3 — Login and Logout (`session['user_id']` and `session['user_name']` set on login)

## Routes
- `GET /profile` — fetch user + expense summary, render profile template — logged-in only (redirect to `/login` if no session)

## Database changes
No new tables or columns. Two new helper functions are added to `database/db.py`:

- `get_user_by_id(user_id)` — returns a single user row by primary key
- `get_expense_summary(user_id)` — returns a dict with:
  - `total_spent` — sum of all expense amounts for the user (REAL, 0.0 if none)
  - `expense_count` — total number of expense rows for the user
  - `recent` — list of the 5 most recent expenses ordered by `date DESC` (id, amount, category, date, description)

## Templates
- **Create:** `templates/profile.html`
  - Extends `base.html`
  - Header section: greeting ("Hello, {name}!") and the user's email
  - Summary card row: two stat cards — "Total Spent" (formatted as currency) and "Expenses Logged"
  - Recent expenses table: columns — Date, Category, Description, Amount — limited to 5 rows
  - Empty-state message when the user has no expenses yet
  - "Add Expense" button linking to `url_for('add_expense')` (stub route, link only)

## Files to change
- `app.py` — replace the stub `profile()` function with a real implementation: check session, call DB helpers, pass data to template
- `database/db.py` — add `get_user_by_id(user_id)` and `get_expense_summary(user_id)`

## Files to create
- `templates/profile.html` — profile page template
- `static/css/profile.css` — page-specific styles (imported via `base.html` block or `<link>` in the template)

## New dependencies
No new pip packages.

## Rules for implementation
- No SQLAlchemy or ORMs — raw `sqlite3` only
- Parameterised queries only — never f-strings in SQL
- Passwords hashed with werkzeug (no change needed here, just maintain the pattern)
- Use CSS variables — never hardcode hex values in `profile.css`
- All templates extend `base.html`
- Authentication guard: if `session.get('user_id')` is falsy, call `redirect(url_for('login'))` at the top of the route — do not use a decorator
- `get_user_by_id()` and `get_expense_summary()` must live in `database/db.py`, never inline in the route
- The `profile()` route function must do exactly: guard → fetch → render. Nothing else.
- `total_spent` must default to `0.0` when the user has no expenses (use `COALESCE` in SQL or a Python fallback)
- Amount values must be formatted as currency in the template (e.g. `"${:.2f}".format(amount)` or a Jinja2 filter)
- The "Add Expense" link must use `url_for('add_expense')` — never a hardcoded path

## Definition of done
- [ ] `GET /profile` without a session redirects to `/login`
- [ ] `GET /profile` with a valid session renders `profile.html` (no 500)
- [ ] The page displays the logged-in user's name and email
- [ ] "Total Spent" shows the correct sum of all user expenses formatted as currency
- [ ] "Expenses Logged" shows the correct count of user expenses
- [ ] The recent expenses table shows up to 5 rows ordered newest-first
- [ ] When the user has no expenses, an empty-state message appears instead of an empty table
- [ ] The "Add Expense" button is present and links to `/expenses/add`
- [ ] No DB logic exists inside the `profile()` route function in `app.py`
- [ ] Styles use CSS variables only — no hardcoded hex values in `profile.css`
