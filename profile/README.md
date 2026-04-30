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
- **Integrate into an existing app**: pick your task engine
  [below](#engines-we-support)
- **Read the docs**: <https://z4j.dev>
- **Project website**: <https://z4j.com>

## z4j-brain (the control plane)

[**z4j-brain**](https://github.com/z4jdev/z4j-brain) is the main
application. Server, dashboard, REST API, audit log. One process per
environment, agents connect over an authenticated WebSocket, the
dashboard surfaces every task / worker / queue / schedule event and
exposes the operator action surface.

What an operator gets:

- **Unified action surface across every Python task engine.** Retry,
  cancel, bulk retry, purge queue, requeue dead-letter, restart
  worker, schedule CRUD, manual trigger. Same workflow whether the
  task ran on Celery, RQ, Dramatiq, Huey, arq, or TaskIQ.
- **Real audit story.** HMAC-chained tamper-evident audit log of
  every privileged action, with the issuer, target, source IP,
  timestamp, and result. Exportable to CSV / JSON / xlsx for
  compliance reviews.
- **RBAC.** Project-scoped roles (Viewer / Operator / Admin / global
  brain Admin). Argon2id passwords, signed session cookies, CSRF
  tokens, per-project bearer-token API keys.
- **Reconciliation.** Background worker reconciles tasks against
  the engine's ground truth on a continuous cadence. No stale
  "running" rows after a worker SIGKILL, no orphaned "pending"
  tasks the broker already discarded.
- **Notifications.** Per-user subscriptions and per-project
  defaults across email / Slack / PagerDuty / Discord / Telegram /
  webhook, with cooldown, mute, priority filters, and a personal
  delivery log.
- **Schedules**, with per-schedule trigger and a *Sync now* button
  that pulls a fresh inventory from any connected agent. See the
  [Schedulers section](#schedulers) below for how schedule sources
  fit in.
- **First-class multi-engine.** A single project runs Celery + RQ +
  arq side by side; the brain renders the right badges per task,
  routes operator actions to the right adapter, and keeps the
  audit log uniform across them.

```bash
pip install z4j-brain                  # SQLite, single process
pip install 'z4j-brain[postgres]'      # production
z4j-brain serve
```

The brain is **AGPL v3** because it's the service operators host.
Everything your application code imports is **Apache-2.0**.

## Engines we support

Six Python task engines, all first-class. Mix and match within a
project; the brain renders them uniformly.

| Engine | Adapter | Notes |
|---|---|---|
| **Celery** | [z4j-celery](https://github.com/z4jdev/z4j-celery) | Widest feature coverage. Pool restart with zero task loss, broker-side rate limiting. |
| **RQ** | [z4j-rq](https://github.com/z4jdev/z4j-rq) | Redis-backed; Django and Flask both first-class. |
| **Dramatiq** | [z4j-dramatiq](https://github.com/z4jdev/z4j-dramatiq) | Middleware-based capture, no decorator changes to your actors. |
| **Huey** | [z4j-huey](https://github.com/z4jdev/z4j-huey) | Huey 2.x and 3.x, redis / sqlite / in-memory backends. |
| **arq** | [z4j-arq](https://github.com/z4jdev/z4j-arq) | Async-native; common pairing with FastAPI. |
| **TaskIQ** | [z4j-taskiq](https://github.com/z4jdev/z4j-taskiq) | Async-native; middleware hooks. |

Each adapter streams task lifecycle events to the brain and
accepts operator control actions back the same WebSocket. All
Apache-2.0.

## Schedulers

z4j surfaces schedules from your existing in-language scheduler
(celery-beat, rq-scheduler, APScheduler, etc.) so you can see them
on the dashboard alongside tasks. Or you can run **z4j-scheduler**
as the canonical scheduler across mixed engines, which is what
makes the project genuinely different from Flower / rq-dashboard /
viewer-grade tooling.

### Observation-only adapters

These wrap the engine's native scheduler and surface its existing
schedules in the dashboard without taking ownership. Use them when
the in-language scheduler already meets your needs and you just
want the schedules visible alongside tasks.

| Engine | Scheduler companion |
|---|---|
| Celery | [z4j-celerybeat](https://github.com/z4jdev/z4j-celerybeat) |
| RQ | [z4j-rqscheduler](https://github.com/z4jdev/z4j-rqscheduler) |
| Huey | [z4j-hueyperiodic](https://github.com/z4jdev/z4j-hueyperiodic) |
| arq | [z4j-arqcron](https://github.com/z4jdev/z4j-arqcron) |
| TaskIQ | [z4j-taskiqscheduler](https://github.com/z4jdev/z4j-taskiqscheduler) |
| APScheduler | [z4j-apscheduler](https://github.com/z4jdev/z4j-apscheduler) |
| Dramatiq | (no upstream scheduler, use z4j-scheduler) |

### z4j-scheduler (canonical, engine-agnostic)

[**z4j-scheduler**](https://github.com/z4jdev/z4j-scheduler) is
z4j's own dynamic scheduler. It's the genuinely differentiated piece
and worth a closer look if any of these are true: you run more than
one engine, you want to edit schedules live without daemon restarts,
or you need an audit trail of schedule changes.

Concretely, what z4j-scheduler does that the in-language schedulers
don't:

- **Engine-agnostic.** One service drives all six engines from one
  process. A project running Celery for legacy services and arq
  for a FastAPI rewrite uses the same scheduler for both.
- **Live editing.** Schedules live in z4j-brain's Postgres database.
  Create, edit, pause, resume, rename, delete from the dashboard
  or REST API. No daemon restart.
- **HMAC-chained audit log.** Every schedule mutation (who, what,
  when, from which IP) recorded alongside the brain's other audit
  rows. celery-beat keeps no record. django-celery-beat keeps a
  partial one only if django-auditlog is wired up.
- **HA-ready.** Multiple instances against one Postgres; advisory
  locks elect a leader; followers stay warm. Rolling restarts and
  failovers are seconds, not minutes.
- **Reversible.** Importers cover every native scheduler
  (celery-beat / django-celery-beat / rq-scheduler / APScheduler /
  Huey @periodic_task / arq cron / taskiq sources / system
  crontab). Exporters write back to those same formats. Round-trip
  integrity is pinned by tests; you can leave whenever you want.
- **Solar triggers + DST correctness.** Schedule kinds: cron,
  interval, one-shot, solar (sunrise / sunset / dawn / dusk /
  noon / midnight at a given lat / lon). IANA zones validated at
  the boundary; DST fall-back fold fixed (no double-fires);
  spring-forward gap handled per-schedule.

If you only run one engine, have no compliance pressure, and your
existing in-language scheduler meets your needs, the
observation-only adapter above is the simpler choice. z4j-scheduler
exists for the mixed-engine + audit + live-editing case, and as a
reversible migration path when those constraints change.

```bash
pip install z4j-scheduler
z4j-scheduler import --from celery --celery-app myapp:app \
  --project myproject --brain-url https://brain.example.com \
  --api-token "$Z4J_SCHEDULER_BRAIN_API_TOKEN" --dry-run
```

Migration walkthrough at
[z4j.dev/scheduler/migrating-from-celery-beat/](https://z4j.dev/scheduler/migrating-from-celery-beat/).

## Framework integrations

One-line install for the three most common Python web frameworks.
Each adapter auto-discovers whichever engine adapter you have
installed alongside; cross-stack combos like Flask + RQ or
FastAPI + arq are first-class supported.

- [**z4j-django**](https://github.com/z4jdev/z4j-django). Add
  `"z4j_django"` to `INSTALLED_APPS`; the agent starts when
  Django boots.
- [**z4j-flask**](https://github.com/z4jdev/z4j-flask). `Z4J(app)`
  initializer in your app factory.
- [**z4j-fastapi**](https://github.com/z4jdev/z4j-fastapi).
  `add_z4j(app)` call after constructing the FastAPI app.
- [**z4j-bare**](https://github.com/z4jdev/z4j-bare). Framework-free
  agent runtime for plain scripts, Celery / RQ / Dramatiq workers,
  or custom services that don't have a web framework.

All Apache-2.0.

## Foundations

- [**z4j-core**](https://github.com/z4jdev/z4j-core). Shared SDK
  used by every agent. Pure-Python, no framework imports, vendorable
  into any worker process.
- [**z4j**](https://github.com/z4jdev/z4j). Umbrella meta-package.
  `pip install z4j[django,celery]` resolves a coherent stack in one
  command; cross-versioning across all 20 packages stays in sync
  via the umbrella's floors.

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
