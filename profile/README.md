# z4j

**Open-source control plane for Python task queues.**

One dashboard, one API and one agent per layer of your stack, for the
task engines you already run: Celery, RQ, Dramatiq, Huey, arq and taskiq.
Self-hosted. No telemetry. Nothing to rewrite.

[![PyPI](https://img.shields.io/pypi/v/z4j?label=z4j&color=blue)](https://pypi.org/project/z4j/)
[![Python](https://img.shields.io/pypi/pyversions/z4j?color=blue)](https://pypi.org/project/z4j/)
[![License](https://img.shields.io/badge/license-AGPL--3.0%20%2F%20Apache--2.0-green)](#license)
[![Docs](https://img.shields.io/badge/docs-z4j.dev-orange)](https://z4j.dev)
[![Demo](https://img.shields.io/badge/demo-demo.z4j.dev-yellow)](https://demo.z4j.dev)

## Try the live demo (no install)

[**demo.z4j.dev**](https://demo.z4j.dev) is the dashboard running in your
browser against sample data. One click on the pre-filled login lands you
in a populated control plane with four sample projects, including a
Django + Celery project
with a live incident (a failing schedule, an alert firing, a worker
offline) and a mixed-engine project driven by z4j-scheduler.

It is a navigable preview, not a sandbox: mutations are blocked, no real
backend is connected, and a refresh resets it. Useful before you commit
to `pip install`.

## Install

```bash
pip install z4j
z4j serve
```

Open **http://localhost:7700**. First boot mints the secrets, runs the
migrations and prints a one-time setup URL for the first admin. SQLite
and the dashboard are bundled in the wheel; no database server and no
npm install. For Docker, and for PostgreSQL when you want replicas,
partitioned history and full-text search, see the
[install guide](https://z4j.dev/getting-started/install/). Switching the
database backend later does not move your data; the guide says exactly
what does and does not carry over.

## Where to go next

- **Run it locally**: start with [z4j](https://github.com/z4jdev/z4j)
- **Integrate into an existing app**: pick your task engine
  [below](#engines-we-support)
- **Read the docs**: <https://z4j.dev>
- **Project website**: <https://z4j.com>

## z4j (the control plane)

[**z4j**](https://github.com/z4jdev/z4j) is the main application:
server, dashboard, REST API, rule engine, audit log. One process per
environment. Agents connect outbound over an HMAC-signed WebSocket; the
dashboard surfaces every task, worker, queue and schedule event and
exposes the operator controls.

What an operator gets:

- **Capability-aware controls.** Retry, cancel, bulk retry, purge,
  requeue dead-letter, restart worker, schedule CRUD, manual trigger,
  each offered only where the connected adapter advertises it. Six
  engines do not have six identical feature sets, and the dashboard does
  not pretend they do.
- **Failure fingerprinting.** The same exception across runs, and across
  engines, collapses into one issue with an occurrence count and an
  open or recovered state.
- **Automation with the brakes built in.** Per-project rules that
  notify, retry or cancel on a task or scheduler trigger, each with a
  circuit breaker, a kill switch and a dry-run mode.
- **An audit log you can verify.** Every privileged action lands on an
  HMAC-chained log with issuer, target, source IP, timestamp and result;
  `z4j audit verify` checks the chain offline. The threat model, and the
  boundary the chain does not cover, are published in the docs.
- **RBAC and MFA.** Three project-scoped roles (viewer, operator,
  admin), opt-in TOTP MFA with recovery codes, Argon2id passwords, signed
  session cookies, CSRF tokens and scoped API keys.
- **Reconciliation.** A background worker checks task state against the
  engine's own view where the engine exposes one, so a worker killed
  mid-task does not leave rows stuck on "running".
- **Notifications.** Email, Slack, Microsoft Teams, PagerDuty, Discord,
  Telegram and webhooks, with severity-aware subscriptions, cooldown and
  mute.
- **Schedules**, with a per-schedule trigger and a *Sync now* that pulls
  a fresh inventory from any connected agent. See the
  [Schedulers section](#schedulers) for how schedule sources fit in.
- **Multi-engine by design.** One project can run Celery, RQ and arq
  side by side; z4j renders the right badges per task, routes each
  action to the right adapter and keeps the audit log uniform.

```bash
pip install z4j                  # SQLite, single process
pip install 'z4j[postgres]'      # PostgreSQL driver for a replicated install
z4j serve
```

z4j is **AGPL-3.0-or-later** because it is the service operators host.
Everything your application code imports is **Apache-2.0**.

## Engines we support

Six Python task engines, all first-class. Mix them within a project.

| Engine | Adapter | Notes |
|---|---|---|
| **Celery** | [z4j-celery](https://github.com/z4jdev/z4j-celery) | Widest coverage: pool restart through Celery's own control channel, broker-side rate limiting. |
| **RQ** | [z4j-rq](https://github.com/z4jdev/z4j-rq) | Redis-backed; Django and Flask both first-class. |
| **Dramatiq** | [z4j-dramatiq](https://github.com/z4jdev/z4j-dramatiq) | Middleware-based capture, no decorator changes to your actors. |
| **Huey** | [z4j-huey](https://github.com/z4jdev/z4j-huey) | Huey 2.4 and later. |
| **arq** | [z4j-arq](https://github.com/z4jdev/z4j-arq) | Async-native; the common pairing with FastAPI. |
| **taskiq** | [z4j-taskiq](https://github.com/z4jdev/z4j-taskiq) | Async-native; middleware hooks. |

Each adapter streams task lifecycle events to z4j and accepts operator
actions back over the same WebSocket. All Apache-2.0. The exact action
matrix per engine is on [z4j.com](https://z4j.com/engines).

## Schedulers

z4j surfaces schedules from your existing scheduler (celery-beat,
rq-scheduler, APScheduler and the others below) so you can see them on
the dashboard alongside tasks. Or you can run **z4j-scheduler** as the
one scheduler across mixed engines, which is what makes the project
genuinely different from viewer-grade tooling.

### Observation-only adapters

These wrap the engine's native scheduler and surface its existing
schedules in the dashboard without taking ownership. Use them when the
scheduler you have meets your needs and you want the schedules visible
alongside tasks.

| Engine | Scheduler companion |
|---|---|
| Celery | [z4j-celerybeat](https://github.com/z4jdev/z4j-celerybeat) |
| RQ | [z4j-rqscheduler](https://github.com/z4jdev/z4j-rqscheduler) |
| Huey | [z4j-hueyperiodic](https://github.com/z4jdev/z4j-hueyperiodic) |
| arq | [z4j-arqcron](https://github.com/z4jdev/z4j-arqcron) |
| taskiq | [z4j-taskiqscheduler](https://github.com/z4jdev/z4j-taskiqscheduler) |
| APScheduler | [z4j-apscheduler](https://github.com/z4jdev/z4j-apscheduler) |
| Dramatiq | (no upstream scheduler; use z4j-scheduler) |

### z4j-scheduler (engine-agnostic)

[**z4j-scheduler**](https://github.com/z4jdev/z4j-scheduler) is z4j's
own scheduler. Worth a closer look if you run more than one engine, want
to edit schedules without a daemon restart, or need a record of who
changed what.

What it does that the in-language schedulers do not:

- **Engine-agnostic.** One process drives all six engines. A project
  running Celery for older services and arq for a FastAPI rewrite uses
  one scheduler for both.
- **Live editing.** Schedules live in z4j's database. Create, edit,
  pause, resume and delete from the dashboard, declarative config or the
  REST API. No daemon restart.
- **Audited.** Every schedule change (who, what, when, from where) lands
  on the same HMAC-chained log as z4j's other privileged actions.
- **Leader election.** Multiple instances against one PostgreSQL
  database race for an advisory lock; only the leader ticks, followers
  stay warm. Takeover has no fixed response-time promise, and HA does
  not make task execution exactly-once.
- **Catch-up you choose.** After an outage each schedule decides for
  itself: skip what it missed, fire one, or fire every slot it owes.
- **Importers and advisory exports.** Import from celery-beat,
  django-celery-beat, rq-scheduler, APScheduler jobstores or system
  cron, with a diff you can verify before cutover. Exports for Celery,
  RQ, APScheduler and cron are advisory: you review and apply the
  generated file yourself.
- **Cron, interval, one-shot and solar triggers.** IANA zones are
  validated; during a fall-back an ambiguous wall-clock slot is two
  distinct instants and z4j fires once at each.

```bash
pip install z4j-scheduler
z4j-scheduler import --from celery --project myproject
# brain URL and token come from --brain-url / --api-token or the
# Z4J_SCHEDULER_BRAIN_* environment variables
```

Full comparison with celery-beat, django-celery-beat, rq-scheduler,
APScheduler and cron, including where those remain the right choice:
[z4j.dev/schedulers/z4j-scheduler/](https://z4j.dev/schedulers/z4j-scheduler/).

## Framework integrations

One package per web framework. Each adapter picks up whichever engine
adapter you installed alongside it; cross-stack combinations such as
Flask + RQ or FastAPI + arq are first-class.

- [**z4j-django**](https://github.com/z4jdev/z4j-django): an AppConfig
  integration; the agent starts when Django boots.
- [**z4j-flask**](https://github.com/z4jdev/z4j-flask): the Flask
  extension pattern, initialised on your app.
- [**z4j-fastapi**](https://github.com/z4jdev/z4j-fastapi): a lifespan
  integration for async stacks.
- [**z4j-bare**](https://github.com/z4jdev/z4j-bare): the framework-free
  agent runtime for plain scripts, worker processes and custom services.

Quickstarts for each: <https://z4j.dev/getting-started/install/>. All
Apache-2.0.

## Foundations

- [**z4j-core**](https://github.com/z4jdev/z4j-core): the shared SDK used
  by every agent and by the brain. Protocols, domain models, redaction,
  the signed envelope. No framework imports.
- [**z4j**](https://github.com/z4jdev/z4j): the flagship distribution.
  The brain, plus an extras catalogue for pulling in adapters:
  `pip install 'z4j[django,celery]'` resolves a coherent stack in one
  command, with version floors that keep every package on one line.

## License

Split on purpose, not by accident.

- **z4j** (the central process you run in your infrastructure) is
  [**AGPL-3.0-or-later**](https://www.gnu.org/licenses/agpl-3.0.html).
  You can self-host, modify and redistribute. If you run a modified copy
  as a network service, publish your modifications under the same
  license. If that is incompatible with your policy, a commercial license
  is available: `licensing@z4j.com`.
- **All agent and scheduler packages** (engine adapters, framework
  integrations, foundations and z4j-scheduler) are
  [**Apache-2.0**](https://www.apache.org/licenses/LICENSE-2.0).
  Integrating z4j into a proprietary application does **not** subject
  your application to the AGPL.

## Project status

The ecosystem ships 19 PyPI packages cross-versioned to one release
line, with floors in every package's pyproject so mixed installs stay
coherent. The badge at the top of this page shows the current release;
the release history is at
[z4j.dev/reference/changelog/](https://z4j.dev/reference/changelog/).
