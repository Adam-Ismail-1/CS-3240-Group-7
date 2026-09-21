# UVA Meal Exchange Sharing

CS 3240 - Fall 2026 - Group 7

A student application for offering and finding meals through unused meal exchanges. The team is investigating the difficulties students encounter in existing free-food group chats and using stakeholder feedback to define the app's requirements.

## Team

| Member | Role |
| --- | --- |
| Will | Scrum Lead |
| Adam | Requirements Lead |
| Sid | QA |
| Michael | Development Lead |
| Aidan | DevOps |

## Current scope

This is the Sprint 2 foundation: a Django homepage, environment-based settings, PostgreSQL CI, and Heroku deployment configuration. Meal listings, Google login, user profiles, moderation, messaging, and S3 uploads are not implemented yet. The repository configuration alone does not create a running Heroku app.

## Technology

- Python 3.12 and Django 6.1.1
- Django templates with plain CSS; no separate frontend API framework
- SQLite for optional local development
- PostgreSQL for GitHub Actions and production on Heroku
- GitHub Actions for checks and tests
- Gunicorn and WhiteNoise for the deployed app and its static assets
- Amazon S3 for future user-uploaded images and documents

## Local setup

Clone the repository and open its root folder in VS Code. If you already have a working virtual environment, activate it and start with the dependency installation step below.

### Windows PowerShell

```powershell
git clone https://github.com/Adam-Ismail-1/CS-3240-Group-7.git
cd CS-3240-Group-7
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
Copy-Item .env.example .env
python manage.py migrate
python manage.py runserver
```

Only copy `.env.example` if you do not already have a local `.env`; preserve your existing settings. If PowerShell blocks activation, use `.\.venv\Scripts\python.exe` in place of `python` without activating the environment.

### macOS / Linux

Use Python 3.12 for consistency with CI and Heroku.

```bash
git clone https://github.com/Adam-Ismail-1/CS-3240-Group-7.git
cd CS-3240-Group-7
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cp .env.example .env
python manage.py migrate
python manage.py runserver
```

Open <http://127.0.0.1:8000/>. To access Django's development administration site, create a local account with `python manage.py createsuperuser`, then visit `/admin/`.

`.env` is loaded automatically and excluded from Git. Existing environment variables take precedence over `.env`. With the example settings, local development uses SQLite. To use a local PostgreSQL database, set `DATABASE_URL` in your local `.env` and leave `DATABASE_SSL_REQUIRE=False` only for that local server.

Keep dependency versions pinned in `requirements.txt`. After deliberately adding a dependency in the project's virtual environment, update the file and review the diff. On Windows PowerShell, use `python -m pip freeze | Set-Content -Encoding utf8 requirements.txt` to avoid writing UTF-16 text.

## GitHub workflow

Start work from an up-to-date `main`, create a descriptive branch, and open a pull request when the change is ready for review. Have another team member review it and wait for CI before merging. Do not commit `.env`, database files, virtual environments, credentials, or survey response exports.

The workflow in `.github/workflows/django-ci.yml` runs on pull requests and pushes to `main`. It installs the pinned dependencies, starts a temporary PostgreSQL 16 database, verifies the selected database engine, checks and applies migrations, collects static files, runs Django tests, and checks the homepage with production settings. It does not use the Heroku database or require Heroku/AWS credentials.

No application test suite exists yet. The initial checks establish that the base project builds and starts; add behavior tests as app features are developed.

After the first successful run, the repository owner can configure a ruleset for `main` requiring a pull request, one approval, and the `Django checks and tests (PostgreSQL)` status check. Confirm all teammates have accepted their repository invitations.

## User stories and backlog

Use **Issues -> New issue -> User story** after this template is merged into the default branch. Each issue records the story, requirement source, supporting evidence, acceptance criteria, and priority. In the requirements document, link each finalized user story to its issue.

Use actual survey, interview, and artifact-analysis findings for elicited requirements. The following are already known course constraints and should be represented in the backlog as the team plans the work:

- Google login, with account linking if public password login is also offered.
- Student and higher-level moderator capabilities, including moderation of users and content.
- Profiles with profile pictures and relevant information.
- Account deletion and editing/deleting a user's own contributions.
- In-app messaging as the primary messaging channel.
- S3 storage for uploaded images and documents.

Specific meal-sharing features and their priorities should follow elicitation. Offering and receiving meals need not be mutually exclusive user roles.

## Heroku setup

Follow the course's Tech Resources instructions for the student benefit and team account setup. The team still needs to create a Heroku app, attach the course-approved PostgreSQL database, and deploy the reviewed code. Choose services through the course setup process; this repository does not provision billable resources.

Before the first deploy, configure these values in the Heroku app's **Settings -> Config Vars**:

| Variable | Value |
| --- | --- |
| `DJANGO_DEBUG` | `False` |
| `DJANGO_SECRET_KEY` | A newly generated secret used only for this app |
| `DJANGO_ALLOWED_HOSTS` | The app's actual hostname, without `https://` or a trailing slash; separate additional hostnames with commas |
| `DATABASE_URL` | Supplied by the attached Heroku PostgreSQL database |
| `DATABASE_SSL_REQUIRE` | `True` |

Use the exact app URL shown in the Heroku dashboard; newer URLs may include a generated suffix. Generate the secret locally with the following command, paste the result into Config Vars, and do not put it in an issue, commit, or shared document:

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

Connect the Heroku app to this repository and deploy `main` after the setup pull request has been reviewed and merged. Alternatively, with the Heroku CLI installed and authenticated, replace `YOUR_APP_NAME` below with the existing app's name:

```bash
heroku git:remote -a YOUR_APP_NAME
git push heroku main
heroku open -a YOUR_APP_NAME
```

The root `.python-version` pins the Python series. `requirements.txt` supplies build dependencies. Heroku collects static files during its Python build; the `Procfile` applies migrations in the release phase and starts Gunicorn. WhiteNoise serves bundled CSS and Django admin assets. PostgreSQL is required whenever debug mode is off and on Heroku/CI; an absent database configuration fails instead of silently using SQLite.

Verify the public homepage loads over HTTPS, then check deployment logs if it does not:

```bash
heroku logs --tail -a YOUR_APP_NAME
```

Before implementing any user uploads, configure an S3 bucket and the Django S3 storage backend. WhiteNoise and Heroku's local filesystem are not the upload storage solution for this project.

## Sprint 2 deliverables

- Conduct and document three elicitation techniques, including direct stakeholder input and analysis of existing systems/artifacts.
- Present and prioritize user stories in the requirements document and GitHub Issues.
- Deploy the base Django project to Heroku.
- Sprint Check 2 report: September 27, 2026, 12:00 PM ET.
- Requirements document: September 29, 2026, 11:59 PM ET.

## Setup references

- [GitHub Actions PostgreSQL services](https://docs.github.com/en/actions/tutorials/use-containerized-services/create-postgresql-service-containers)
- [Django on Heroku](https://devcenter.heroku.com/articles/django-app-configuration)
- [Heroku Python version selection](https://devcenter.heroku.com/articles/python-runtimes)
- [WhiteNoise with Django](https://whitenoise.readthedocs.io/en/stable/django.html)
