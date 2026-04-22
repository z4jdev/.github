# z4j

**Open-source control plane for Python task infrastructure.**

One dashboard, one API, one agent SDK - for every Python task engine.
Celery, RQ, Dramatiq, Huey, arq, TaskIQ, APScheduler, or plain
scripts. Self-hosted. Self-contained. Zero external dependencies
beyond Python.

[![PyPI](https://img.shields.io/pypi/v/z4j-brain?label=z4j-brain&color=blue)](https://pypi.org/project/z4j-brain/)
[![Python](https://img.shields.io/pypi/pyversions/z4j-brain?color=blue)](https://pypi.org/project/z4j-brain/)
[![License](https://img.shields.io/badge/license-AGPL--3.0%20%2F%20Apache--2.0-green)](#license)
[![Docs](https://img.shields.io/badge/docs-z4j.dev-orange)](https://z4j.dev)

## Install in 30 seconds

```bash
pip install z4j
export Z4J_SECRET=$(python -c "import secrets; print(secrets.token_urlsafe(48))")
export Z4J_SESSION_SECRET=$(python -c "import secrets; print(secrets.token_urlsafe(48))")
z4j-brain migrate upgrade head
z4j-brain serve
```

Open **http://localhost:7700**. You land on the dashboard. SQLite and
the React SPA are bundled in the wheel, no database server or npm
install required.

## Where to go next

- **Run it locally**: start with [z4j-brain](https://github.com/z4jdev/z4j-brain)
- **Integrate into an existing app**: pick your task engine below
- **Read the docs**: <https://z4j.dev>
- **Project website**: <https://z4j.com>

## Packages

### Brain (control plane)

- [**z4j-brain**](https://github.com/z4jdev/z4j-brain) - FastAPI
  backend + React dashboard + Alembic migrations, bundled in one
  pip-installable wheel. **AGPL v3.**
- [z4j](https://github.com/z4jdev/z4j) - umbrella meta-package:
  `pip install z4j[django,celery]` in one command.

### Engine adapters (Apache 2.0, safe for proprietary code)

| Engine | Adapter | Scheduler companion |
|---|---|---|
| Celery | [z4j-celery](https://github.com/z4jdev/z4j-celery) | [z4j-celerybeat](https://github.com/z4jdev/z4j-celerybeat) |
| RQ | [z4j-rq](https://github.com/z4jdev/z4j-rq) | [z4j-rqscheduler](https://github.com/z4jdev/z4j-rqscheduler) |
| Dramatiq | [z4j-dramatiq](https://github.com/z4jdev/z4j-dramatiq) | - |
| Huey | [z4j-huey](https://github.com/z4jdev/z4j-huey) | [z4j-hueyperiodic](https://github.com/z4jdev/z4j-hueyperiodic) |
| arq | [z4j-arq](https://github.com/z4jdev/z4j-arq) | [z4j-arqcron](https://github.com/z4jdev/z4j-arqcron) |
| TaskIQ | [z4j-taskiq](https://github.com/z4jdev/z4j-taskiq) | [z4j-taskiqscheduler](https://github.com/z4jdev/z4j-taskiqscheduler) |
| APScheduler | [z4j-apscheduler](https://github.com/z4jdev/z4j-apscheduler) | - |

### Framework integrations (Apache 2.0)

- [z4j-django](https://github.com/z4jdev/z4j-django) - Django task telemetry
- [z4j-flask](https://github.com/z4jdev/z4j-flask) - Flask background tasks
- [z4j-fastapi](https://github.com/z4jdev/z4j-fastapi) - FastAPI BackgroundTasks

### Foundations (Apache 2.0)

- [z4j-core](https://github.com/z4jdev/z4j-core) - shared SDK used by every agent
- [z4j-bare](https://github.com/z4jdev/z4j-bare) - no-engine agents for scripts / crons / one-shot jobs

## License

Split on purpose, not by accident.

- **Brain** (the server you run in your infrastructure) is
  [**AGPL v3**](https://www.gnu.org/licenses/agpl-3.0.html). You can
  self-host, modify, and redistribute - if you run a modified copy
  as a network service, publish your modifications under the same
  license. If that's incompatible with your policy, a commercial
  license is available: `licensing@z4j.com`.
- **All 17 agents** (engine adapters, framework integrations,
  foundations) are
  [**Apache 2.0**](https://www.apache.org/licenses/LICENSE-2.0).
  Integrating z4j into a proprietary application does **not** subject
  your application to the AGPL.

The split is deliberate: the brain is the service operators host,
protected by copyleft. The agents are the client libraries
integrators embed, open under a permissive license.

## Project status

v1.0.0 shipped April 2026. Production-ready, 1000+ tests,
zero known CVEs across the Python and JavaScript dependency
graphs.
