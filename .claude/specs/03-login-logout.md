# Spec: Login and Logout

## Overview
This step wires up real authentication for Spendly. `GET /login` and `GET /register` already render their templates, and both forms already POST to `/login` and `/register` — but neither has a handler that actually checks credentials or establishes a session yet. This step implements the `POST /login` handler (verify email + password against the `users` table, start a Flask session) and the `GET /logout` handler (clear the session), plus reflects logged-in state in the navbar. This unlocks session-based access for later steps (`/profile` in Step 4, `/expenses/*` in Steps 7–9).

## Depends on
- Step 1 (database setup) — `users` table, `get_db()` with FK pragma. Complete.
- Step 2 (registration) — `GET /register` renders the form. Note: `POST /register` is not yet implemented in `app.py`; this step does not depend on it — login is tested against the seeded demo user (`demo@spendly.com` / `demo123`).

## Routes
- `POST /login` — validate email/password against `users`, create session on success, redirect to `/profile`; re-render `login.html` with an error on failure — public
- `GET /logout` — clear the session, redirect to `/` — logged-in (also safe to hit when not logged in)

## Database changes
No schema changes. Add one query helper to `database/db.py`:
- `get_user_by_email(email)` — returns the matching row from `users` (or `None`), used by the login handler to fetch `password_hash` for verification.

## Templates
- **Create:** none — `login.html` and `register.html` already exist and already post to the right endpoints.
- **Modify:** `templates/base.html` — nav currently always shows "Sign in" / "Get started". Update it to show "Profile" / "Sign out" (linking to `url_for('logout')`) when a session exists, using `session.get('user_id')` passed into the template context (Flask injects `session` into Jinja automatically, no extra plumbing needed).

## Files to change
- `app.py` — add `app.secret_key`, implement `POST /login`, implement `GET /logout`
- `database/db.py` — add `get_user_by_email(email)`
- `templates/base.html` — conditional nav links based on session state

## Files to create
None.

## New dependencies
No new dependencies — Flask's built-in `session` (signed cookie) covers this; only `app.secret_key` needs to be set in `app.py`.

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only
- Passwords verified with `werkzeug.security.check_password_hash` — never compare plaintext
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Session state read via Flask's `session` object only — no cookies/tokens hand-rolled
- `GET /login` (existing) stays untouched; only the `POST` method is new on that same route
- Do not touch `/profile` or `/expenses/*` stub bodies — only their eventual redirect target from a successful login

## Definition of done
- [ ] Submitting `demo@spendly.com` / `demo123` on `/login` redirects to `/profile`
- [ ] Submitting a wrong password re-renders `login.html` with an error message and no session is created
- [ ] Submitting an email that doesn't exist re-renders `login.html` with an error message (no user enumeration — same generic error as wrong password)
- [ ] After login, the navbar shows "Sign out" instead of "Sign in" / "Get started"
- [ ] Visiting `/logout` clears the session and redirects to `/`, and the navbar reverts to "Sign in" / "Get started"
- [ ] Restarting the app and reloading a page after login does not error (secret key is set, session cookie is valid)
- [ ] `app.py` still starts cleanly on port 5001 with `python app.py`
