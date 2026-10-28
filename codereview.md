# Code Review: Spendly (Expense_Tracker_Basic)

**Date:** 2026-10-28
**Scope:** Whole codebase (`app.py`, `database/`, `templates/`, `static/`, spec files, config)
**Stack:** Flask 3.1 · SQLite · Jinja2 · vanilla JS

## Summary

The scaffold is small and clean. Page shells, routing and styling are in place. The backend and database are still empty, and the forms can't be submitted yet. The one bug found during the review (hero dashboard hidden on narrow screens) has been fixed.

## Bug fixed during this review

- **Hero dashboard hidden below 900px.** `static/css/style.css` sets `.hero-visual { display: none }` at that width. `static/css/landing.css` did not override it, so the new dashboard mock would not show on narrow screens. Fixed by adding `display: block` to `.hero-visual` in `landing.css`. Not yet re-checked in a browser.

## Open issues

### High

| # | Issue | Location | Recommendation |
|---|-------|----------|----------------|
| 1 | No backend: `db.py` contains only comments (`get_db`, `init_db`, `seed_db` not written). No code uses a database. | `database/db.py` | Implement Step 1 (SQLite connection with `row_factory` and foreign keys, tables, seed data). |
| 2 | Forms cannot work: `login.html` and `register.html` POST to `/login` and `/register`, but the routes allow GET only, so submitting returns 405. | `app.py` | Accept `methods=["GET", "POST"]` and handle the form data. |
| 3 | Delete is a GET route, so data would be changed by a plain link. | `app.py` (`/expenses/<id>/delete`) | Use POST with CSRF protection when building Step 9. |
| 4 | No `secret_key` and no CSRF protection. | `app.py` | Set `app.secret_key` from an environment variable; add CSRF tokens to forms. |
| 5 | `debug=True` exposes the Werkzeug debugger. | `app.py` (last line) | Fine locally; drive it from an env var and disable before deploying. |

### Medium

| # | Issue | Recommendation |
|---|-------|----------------|
| 6 | `/profile` and expense routes have no login guard or ownership check. | Add an auth guard once sessions exist, and verify an expense belongs to the logged-in user before edit/delete. |
| 7 | Registration has no server-side validation. The form says "Min. 8 characters" but only the browser's `required` attribute is checked. | Validate name, email and password length on the server; hash passwords with `werkzeug.security`. |
| 8 | Spec files sit in the project root with inconsistent names (`Spec_02_Registration.prompt.md`, `Spec_05-backend-routes…`). `Spec_01_database_setup_prompt.md` and `Spec_genration_prompt.md` are the same size and may be duplicates; one name has a typo. | Move to `.claude/specs/` as Spec 1 asks, use one naming scheme, remove duplicates. |
| 9 | No tests. `pytest` and `pytest-flask` are in `requirements.txt` but there is no `tests/` folder. | Add `tests/` with route and database tests as features land. |

### Low

| # | Issue | Recommendation |
|---|-------|----------------|
| 10 | `README.md` is a single line. Setup was not obvious (Python launcher, installing dependencies). | Document Python version, `pip install -r requirements.txt`, and how to run. |
| 11 | Hero rules now exist in both `style.css` and `landing.css`. | Once settled, remove the dead hero rules from `style.css`. |
| 12 | Old `.mock-*` and `.btn-ghost` styles are no longer used by the landing page. | Remove if nothing else needs them. |
| 13 | Video modal: placeholder YouTube URL, and no focus trap (Tab can leave the open modal). Escape, close button and click-outside all work. | Replace `data-src` on `#video-modal-iframe`; add a focus trap if accessibility matters. |
| 14 | `static/js/main.js` is empty. | Fine for now. |

## Strengths

- Templates extend `base.html` and use `url_for`, with no hardcoded paths.
- Colours and fonts are CSS variables; the CSS is organised into labelled sections.
- The spec files give a clear, step-by-step build plan.
- `.gitignore` already covers the SQLite file and `*.pyc`.

## Recommended order of work

1. Build Step 1 (`db.py`), then registration and login (allow POST, hash passwords, set a secret key).
2. Add the login guard and ownership checks before any expense routes.
3. Make delete a POST.
4. Add a `tests/` folder and a real README.
