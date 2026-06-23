# Website_meeting

A small internal Flask app for the **PauseIA** team to track its advocacy
outreach: the **people** it talks to (politicians / officials), the **meetings**
it has with them, and the **mails** it exchanges. The interface is in French;
the code and comments are in English.

## Features

- **People** — name, political group, stance on PauseAI, first-contact date, an
  optional follow-up ("relance") date, and free-form notes. Each person's page
  shows how many mails were sent to / received from them, how many meetings were
  held, and how many days remain until the next planned follow-up.
- **Meetings** — date, one or more people met, a short summary, and an optional
  attached full report (`.docx` / `.odt` / `.txt`). The meetings list is split
  into **upcoming** (soonest first) and **past** (most recent first).
- **Calendar** — a monthly view (`/calendar`) plotting upcoming meetings and
  follow-up ("relance") dates, with month-to-month navigation.
- **Mails** — date, one or more people, direction (sent / received), an
  importance flag, a short summary, and an optional attached document.
- Every record (person, meeting, mail) can be **edited** after creation via a
  "Modifier" button on its detail page; meeting/mail documents can be replaced
  or removed when editing.
- Full-text-ish search over each list, a single shared password gating the
  whole site, and French dates everywhere: they are **typed and displayed as
  DD/MM/YYYY** (`JJ/MM/AAAA`) and stored internally as ISO.

## Data model

Three entities, each with its own integer primary key, linked **many-to-many**
through join tables:

```
persons ──< meeting_persons >── meetings
persons ──< mail_persons    >── mails
```

- A meeting / mail involves **1..n** persons.
- A person appears in **0..n** meetings and **0..n** mails.
- Join tables use `ON DELETE CASCADE`, so deleting a person or a meeting/mail
  cleans up its links automatically. (`PRAGMA foreign_keys = ON` is set per
  connection.)

Tables:

| Table             | Key columns |
|-------------------|-------------|
| `persons`         | `id`, `name`, `political_group`, `stance`, `first_contacted`, `follow_up_date`, `notes`, `created_at` |
| `meetings`        | `id`, `meeting_date`, `summary`, `recorded_by`, `document_*`, `created_at` |
| `mails`           | `id`, `mail_date`, `direction` (`sent`/`received`), `important`, `summary`, `document_*`, `created_at` |
| `meeting_persons` | `(meeting_id, person_id)` |
| `mail_persons`    | `(mail_id, person_id)` |

The database (`meetings.db`, SQLite) and the `uploads/` folder are created
automatically on first run. `init_db()` also runs a lightweight migration that
adds the `follow_up_date` column to an older `persons` table if it is missing.

## Running

```bash
uv run flask --app app run --debug
```

Then open <http://127.0.0.1:5000> and sign in with the shared password.

## Configuration

Set these environment variables in production (defaults are for local dev only):

| Variable       | Default              | Purpose |
|----------------|----------------------|---------|
| `APP_PASSWORD` | `pauseia`            | Shared login password |
| `SECRET_KEY`   | `dev-secret-change-me` | Signs the session cookie |

The political-group dropdown (`POLITICAL_GROUPS`), the stance options
(`STANCES`) and the mail directions (`MAIL_DIRECTIONS`) are defined at the top
of `app.py` and are the single source of truth for the forms — edit them there.

## Project layout

```
app.py              Flask app: config, schema/migrations, routes
static/style.css    Stylesheet (no build step)
templates/          Jinja templates (base, login, people, meetings, mails)
meetings.db         SQLite database (created on first run)
uploads/            Attached documents (created on first run)
```
