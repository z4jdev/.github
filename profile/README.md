# z4j

**Open-source control plane for Python task infrastructure.**

One dashboard, one API, one agent SDK, for every Python task engine.
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
- **Migrate your scheduler**: see the
  [z4j-scheduler section](#z4j-scheduler-the-flagship) below
- **Read the docs**: <https://z4j.dev>
- **Project website**: <https://z4j.com>

## z4j-scheduler (the flagship)

z4j ships its own dynamic scheduler,
[**z4j-scheduler**](https://github.com/z4jdev/z4j-scheduler), as an
engine-agnostic alternative to celery-beat / rq-scheduler /
APScheduler.

It is the genuinely differentiated piece of the project. The other
adapters (z4j-celerybeat, z4j-rqscheduler, z4j-apscheduler, etc.)
are observation-only wrappers that surface existing schedules in
the dashboard. **z4j-scheduler is the schedule.**

### What makes it different

- **Engine-agnostic.** One scheduler service drives all six engines
  (Celery, RQ, Dramatiq, Huey, arq, TaskIQ) from one process. A
  project running Celery for legacy services and arq for a FastAPI
  rewrite uses the same scheduler for both, with one dashboard and
  one audit trail.
- **Live editing.** Schedules live in z4j-brain's Postgres database.
  Create, edit, pause, resume, rename, and delete from the dashboard
  or REST API without a daemon restart. celery-beat needs a
  beat-process restart for static-config edits; rq-scheduler stores
  schedules in Redis but has no editing UI; APScheduler edits are
  in-process only.
- **HMAC-chained audit log.** Every schedule mutation (who, what,
  when, from which IP) is recorded in a tamper-evident audit chain
  alongside the brain's other audit rows. celery-beat keeps no
  record. django-celery-beat keeps a partial one only if you wired
  up django-auditlog.
- **HA-ready.** Multiple scheduler instances run against the same
  Postgres database; advisory locks elect one leader to tick;
  followers stay warm. Rolling restarts and leader failovers are
  seconds, not minutes, with no missed-fire window beyond the
  per-schedule catch-up policy.
- **Reversible by design.** Importers cover celery-beat (static and
  django-celery-beat), rq-scheduler, APScheduler jobstores, Huey
  `@periodic_task`, arq cron, taskiq schedule sources, and system
  crontab. Exporters write any schedule back to those same formats.
  Round-trip integrity is pinned by tests so the schedule
  definitions stay yours; you can leave whenever you want.
- **Solar triggers + DST correctness.** Schedule kinds: cron,
  interval, one-shot, solar (sunrise / sunset / dawn / dusk / noon /
  midnight at a given lat / lon). IANA zones validated at the
  boundary; the DST fall-back fold is fixed (no double-fires);
  spring-forward gap handled per-schedule.

### When to choose it

You probably want it if you run more than one Python task engine
and want one schedule surface; if an auditor or security review
asks who paused the nightly billing job last Tuesday and you don't
have a clean answer; if you're tired of restarting celery-beat to
change a cron expression; or if you want HA scheduling without
standing up a second control plane.

You probably don't need it if you run a single engine, have no
compliance pressure, and the in-language scheduler already meets
your needs. The observation-only adapters
([z4j-celerybeat](https://github.com/z4jdev/z4j-celerybeat),
[z4j-rqscheduler](https://github.com/z4jdev/z4j-rqscheduler),
[z4j-apscheduler](https://github.com/z4jdev/z4j-apscheduler))
exist precisely for that case: surface the schedules in the z4j
dashboard without taking ownership.

```bash
pip install z4j-scheduler
z4j-scheduler import --from celery --celery-app myapp:app \
  --project myproject --brain-url https://brain.example.com \
  --api-token "$Z4J_SCHEDULER_BRAIN_API_TOKEN" --dry-run
```

The migration guide at
[z4j.dev/scheduler/migrating-from-celery-beat/](https://z4j.dev/scheduler/migrating-from-celery-beat/)
walks the importer + dashboard verification path step by step.

## Packages

### Brain (control plane)

- [**z4j-brain**](https://github.com/z4jdev/z4j-brain). FastAPI
  backend + React dashboard + Alembic migrations, bundled in one
  pip-installable wheel. **AGPL v3.**
- [z4j](https://github.com/z4jdev/z4j). Umbrella meta-package:
  `pip install z4j[django,celery]` in one command.

### Scheduler (engine-agnostic, dynamic)

- [**z4j-scheduler**](https://github.com/z4jdev/z4j-scheduler). The
  flagship dynamic scheduler. Drives every engine, edits live from
  the dashboard, HA-ready, audited, importer + exporter for every
  native scheduler. See [the dedicated section above](#z4j-scheduler-the-flagship).

### Engine adapters (Apache 2.0, safe for proprietary code)

| Engine | Adapter | Scheduler companion (observation-only) |
|---|---|---|
| Celery | [z4j-celery](https://github.com/z4jdev/z4j-celery) | [z4j-celerybeat](https://github.com/z4jdev/z4j-celerybeat) |
| RQ | [z4j-rq](https://github.com/z4jdev/z4j-rq) | [z4j-rqscheduler](https://github.com/z4jdev/z4j-rqscheduler) |
| Dramatiq | [z4j-dramatiq](https://github.com/z4jdev/z4j-dramatiq) | (use z4j-scheduler) |
| Huey | [z4j-huey](https://github.com/z4jdev/z4j-huey) | [z4j-hueyperiodic](https://github.com/z4jdev/z4j-hueyperiodic) |
| arq | [z4j-arq](https://github.com/z4jdev/z4j-arq) | [z4j-arqcron](https://github.com/z4jdev/z4j-arqcron) |
| TaskIQ | [z4j-taskiq](https://github.com/z4jdev/z4j-taskiq) | [z4j-taskiqscheduler](https://github.com/z4jdev/z4j-taskiqscheduler) |
| APScheduler | [z4j-apscheduler](https://github.com/z4jdev/z4j-apscheduler) | (engine-agnostic, runs in-process) |

The "scheduler companion" column lists the in-language scheduler
each engine ships with; the z4j adapter surfaces those schedules in
the dashboard without taking ownership. For Dramatiq (which has no
upstream scheduler) and for projects that want one canonical
scheduler across mixed engines, use
[z4j-scheduler](https://github.com/z4jdev/z4j-scheduler) instead.

### Framework integrations (Apache 2.0)

- [z4j-django](https://github.com/z4jdev/z4j-django). Django task
  telemetry; one-line install via `INSTALLED_APPS`.
- [z4j-flask](https://github.com/z4jdev/z4j-flask). Flask background
  tasks; one-line install via `Z4J(app)`.
- [z4j-fastapi](https://github.com/z4jdev/z4j-fastapi). FastAPI
  BackgroundTasks; one-line install via `add_z4j(app)`.

### Foundations (Apache 2.0)

- [z4j-core](https://github.com/z4jdev/z4j-core). Shared SDK used
  by every agent; pure-Python, no framework imports.
- [z4j-bare](https://github.com/z4jdev/z4j-bare). Framework-free
  agent runtime for plain scripts, Celery / RQ / Dramatiq workers,
  custom services.

## License

Split on purpose, not by accident.

- **Brain** (the server you run in your infrastructure) is
  [**AGPL v3**](https://www.gnu.org/licenses/agpl-3.0.html). You can
  self-host, modify, and redistribute. If you run a modified copy
  as a network service, publish your modifications under the same
  license. If that's incompatible with your policy, a commercial
  license is available: `licensing@z4j.com`.
- **All 18 agent + scheduler packages** (engine adapters, framework
  integrations, foundations, plus z4j-scheduler) are
  [**Apache 2.0**](https://www.apache.org/licenses/LICENSE-2.0).
  Integrating z4j into a proprietary application does **not**
  subject your application to the AGPL.

The split is deliberate: the brain is the service operators host,
protected by copyleft. Everything your application code imports is
permissive.

## Project status

z4j 1.3.4 (April 2026) is the current baseline. Earlier 1.x
versions on PyPI are yanked; `pip install` selects 1.3.x. The
ecosystem ships 20 PyPI packages cross-versioned to the same
release line, with floors enforced by the umbrella package so
mixed installs stay coherent.
