# Service Index (uv dev env) agent instructions

## Purpose and boundaries

This repository is the local uv-managed development environment for the ACCESS Operations
Service Index Django application. The application source lives under
`Operations_ServiceIndex_Django/`; this outer repository provides the uv environment
(`pyproject.toml`, `uv.lock`), local dev configuration, and helper assets for running the
app on localhost.

Application development happens here in the dev environment. Provisioning, deployed
configuration, service units, release selection, and production operations belong to
`../Operations_ServiceIndex_Infrastructure`. Do not add AWS or deployment logic here.

## Sources of truth and coupling

- `README.md` describes the local uv dev workflow.
- `pyproject.toml` and `uv.lock` define the supported stack (Django, DRF, allauth,
  gunicorn, psycopg2) and the optional `access-django-user-admin` dependency. Do not copy
  pinned versions into durable guidance.
- `Operations_ServiceIndex_Django/` holds the Django project (settings, URLs, apps,
  models, migrations, tests). If it is a nested repository, treat it as its own Git root
  and do not stage its changes from the outer repository.
- `migrate_schemas.sql` and any per-schema migration flow indicate multi-schema data;
  treat schema and migration changes as high-risk.
- `access_django_user_admin` (from `../ACCESS_Django_user_app_pypi`) is an external
  dependency; a change to its public interface or pinned version is a producer/consumer
  boundary.
- `../Operations_ServiceIndex_Infrastructure` selects the deployed app tag and uv
  environment; that tag plus the uv environment is the shared contract. See
  `multi_agent_plan.md` at the workspace root for the wiring.

## Safe inspection and validation

Always begin at the Git root, and inspect the nested app repository if present:

```bash
git status --short --untracked-files=all
git diff --check
```

Run Django and Python commands through uv, from the app project directory, with a
deliberately nonproduction config only:

```bash
uv run python Operations_ServiceIndex_Django/manage.py check
uv run python Operations_ServiceIndex_Django/manage.py test <affected_app>
uv run python Operations_ServiceIndex_Django/manage.py makemigrations --check --dry-run
```

Do not run checks or tests if the supplied config could reach a production database or
live OAuth. Record missing configuration as a skipped check rather than inventing values.

## Safety and change control

- A human must approve migrations, production data access, dependency changes,
  authentication changes, public route or schema changes, releases, pushes, merges, and
  tags.
- Never run `migrate`, an unbounded `makemigrations`, a schema migration, or a deployment;
  use `makemigrations --check --dry-run` for validation.
- Never commit `serviceindex-dev.conf.json`, `*.log` (`serviceindex-dev.log`),
  `bandit-report.json`, local databases, or the `.venv/`. Never hardcode credentials or
  secret keys.
- Treat settings, root URLs, migrations, authentication, and multi-schema data as
  high-risk shared interfaces.

## Worktree and multi-agent rules

Inspect the full dirty state, including any nested application repository, before editing
and preserve unrelated tracked and untracked files. Never reset, clean, or overwrite user
work. Use one writer per repository or isolated worktree; assign a single owner to
migrations, settings, URLs, and dependency metadata. Infrastructure and consumer reviewers
stay read-only until the interface is frozen.

Every delegated task must state the repository root and base commit, the role and allowed
write paths, prohibited paths and external actions, the frozen API/schema/auth interfaces
and dependent repositories, the required validation checks, and the expected handoff. The
handoff must list files changed, the final diff summary, checks run and their results,
failures or skipped checks, assumptions, and unresolved risks.

## Wiring

See `multi_agent_plan.md` at the workspace root. Coupled repositories:
`Operations_ServiceIndex_Infrastructure` (deploys this app) and
`ACCESS_Django_user_app_pypi` (`access_django_user_admin`, producer).
