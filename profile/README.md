# Lyndrix Platform

**Lyndrix** is an open-source, self-hosted application platform for teams that want full control over their internal tooling — without building everything from scratch.

The runtime, **[lyndrix-core](https://github.com/lyndrix-platform/lyndrix-core)**, gives you authentication, permissions, secrets management, a database, an event bus, a REST API and a web UI — all pre-wired. Every feature on top of it is a plugin. You focus on your business logic, not the plumbing.

## Architecture

```
            Browser / Android app
                     │
              Load balancer
        ┌────────────┼────────────┐
   lyndrix-core  lyndrix-core  lyndrix-core      ← N identical instances (or just one)
   api · leader      api         runner          ← instance roles
        └────────────┼────────────┘
          MariaDB (state)   Vault (secrets)
```

| Building block | What it does |
|---|---|
| **Core** | Python / FastAPI runtime: event-driven boot, plugin lifecycle (install, upgrade, rollback by git tag), IAM, REST API, server-sent events, i18n, theming. |
| **Plugins** | Every feature is a plugin with the same anatomy — `model · logic · api · ui` — and API-first: UIs only talk to the plugin's own REST API. Plugins import a stable, versioned surface (`core.api`), never core internals. |
| **Frontends** | [lyndrix-ui](https://github.com/lyndrix-platform/lyndrix-ui) is the React shell; plugins ship federated React bundles that the shell loads at runtime. [lyndrix-app](https://github.com/lyndrix-platform/lyndrix-app) wraps it for Android. |
| **Event bus** | Components and plugins communicate over a global bus (fire-and-forget plus request/response) — no plugin may *need* another plugin to be loaded. |
| **MariaDB** | Shared state: users, permissions, plugin state, notifications, cluster coordination. |
| **HashiCorp Vault** | Secrets and runtime configuration, with automatic bootstrap/unseal and per-plugin isolation. |
| **Auth** | Local, LDAP and OIDC provider chain, role- and permission-based access control, scoped API keys. |

### High availability

The core runs as a single instance or as a **cluster** of N instances behind a load balancer, sharing one MariaDB and one Vault. Plugins write one code path for both modes.

- **Leader & leases** — one instance holds the leader lease; plugins claim named leases for work that must run exactly once (probers, schedulers, nightly jobs). Leases are fenced, so an instance that lost one stops its work before a peer takes over.
- **Instance roles** — `api`, `leader`, `runner`. Plugins declare a placement (`any`, `singleton`, `pinned:<role>`); the `runner` role hosts workloads that need a specific machine, such as the IaC job runner.
- **Cluster-wide operations** — a plugin install/upgrade is one tracked operation across all instances; events are bridged between instances; rolling restarts go one instance at a time, leader last.
- **Readiness & drain** — `GET /api/ready` for load balancers, graceful drain on SIGTERM.

Details: [HA documentation](https://docs.lyndrix.eu/ha/) (tiers, Docker Compose and Kubernetes guides, operations).

## Versions

> **Status (November 2026):** A full audit of the core and all official plugins (security, cluster behaviour, bugs, tests) has been completed. The resulting releases are marked **upcoming** below — they are not tagged yet. Until they are, the *released* column is what you can install.

| Component | Released | Upcoming |
|---|---|---|
| [lyndrix-core](https://github.com/lyndrix-platform/lyndrix-core) | 1.11.1 (plugin API 1.13) | **1.12.0** (plugin API 1.14) |
| [lyndrix-ui](https://github.com/lyndrix-platform/lyndrix-ui) | 1.8.0 | — |
| [lyndrix-app](https://github.com/lyndrix-platform/lyndrix-app) | 0.2.1 | — |

The core is published as `ghcr.io/lyndrix-platform/lyndrix-core:<version>`. Plugins are installed by the core straight from their repository at a git tag. Core 1.12.0 tightens several defaults — read the upgrade notes in the [core changelog](https://github.com/lyndrix-platform/lyndrix-core/blob/main/CHANGELOG.md) before rolling it out.

## Official plugins

| Plugin | What it does | Released | Upcoming |
|---|---|---|---|
| [iac-orchestrator](https://github.com/lyndrix-platform/lyndrix-plugin-iac-orchestrator) | GitOps controller that runs Terraform/OpenTofu and Ansible pipelines on a dedicated runner instance | 1.6.1 | 1.7.0 |
| [monitoring](https://github.com/lyndrix-platform/lyndrix-plugin-monitoring) | Native state monitoring for infrastructure and services, with outages and maintenance windows | 1.7.0 | 1.8.0 |
| [server-manager](https://github.com/lyndrix-platform/lyndrix-plugin-server-manager) | Central server inventory with configurable hardware catalogs and push-in ingest from other systems | 1.4.0 | 1.5.0 |
| [entity-registry](https://github.com/lyndrix-platform/lyndrix-plugin-entity-registry) | One stable UUID (CI id) per host, hypervisor, service and record; reconciles identities across plugins, never auto-merges | 0.3.0 | 0.4.0 |
| [itsm](https://github.com/lyndrix-platform/lyndrix-plugin-itsm) | Incidents, changes, problems, requests and tasks with a YAML process catalog, audit journal and CI links | 0.3.0 | 0.4.0 |
| [docker-manager](https://github.com/lyndrix-platform/lyndrix-plugin-docker-manager) | Docker host overview with runtime controls (start/stop/restart, logs) | 1.2.0 | 1.3.0 |
| [external-services](https://github.com/lyndrix-platform/lyndrix-plugin-external-services) | Embeds external web services (Grafana, Home Assistant, …) as sidebar entries | 1.4.0 | 1.5.0 |
| [discord-notifier](https://github.com/lyndrix-platform/lyndrix-plugin-discord-notifier) | Two-way Discord integration through the core messaging gateway | 1.2.0 | 1.3.0 |
| [ai-assistant](https://github.com/lyndrix-platform/lyndrix-plugin-ai-assistant) | Claude Code as an in-app assistant: sandboxed runner, chat panel, tool approvals | 0.3.0 | 0.4.0 |
| [meeting-bingo](https://github.com/lyndrix-platform/lyndrix-plugin-meeting-bingo) | Multiplayer meeting bingo — any use case is a valid use case | 1.2.0 | 1.3.0 |

[lyndrix-plugin-collection](https://github.com/lyndrix-platform/lyndrix-plugin-collection) is the plugin index the core's plugin page reads.

## Security

The November 2026 audit covered authentication and authorization, supply chain and secrets handling, cluster behaviour and general robustness. Its fixes ship with the upcoming releases above. Highlights:

- **Identity & sessions** — external (LDAP/OIDC) identity linking fails closed; login sessions expire; changing a password signs out other sessions; deleting a user removes their keys and sessions.
- **Brute-force protection** — every password check goes through one throttled path with per-account and per-address budgets.
- **Least privilege** — scoped API keys are a hard boundary; administrative endpoints, API docs and event streams are permission-gated; disabled plugins lose their routes. Plugins gained finer-grained permissions for sensitive actions (logs, runtime control, settings, credentials).
- **Secrets** — stricter file modes for key material, broader secret masking in logs, secrets no longer travel to the browser or across the cluster event bridge; plugins only pass on the data their consumers need.
- **Plugin supply chain** — stricter validation of plugin sources and dependency manifests; dependency installs run isolated from the core's environment; archive size limits.
- **Cluster** — fenced leases, bounded preflight checks, readiness tied to live coordinator state, loss-free ordered event bridging.
- **Tests** — the core test suite grew from 240 to about 1,200 tests, including multi-instance failure scenarios (leader change, database flaps, runner loss, rolling restart).

We recommend upgrading to the audited releases as soon as they are tagged.

**Reporting a vulnerability:** please report privately, never in a public issue — see the [security policy](https://github.com/lyndrix-platform/lyndrix-core/blob/main/SECURITY.md).

## Get started

```bash
git clone https://github.com/lyndrix-platform/lyndrix-core.git
cd lyndrix-core
docker compose -f docker/docker-compose.dev.yml up -d --build
```

- **App:** `http://localhost:8081` · **Vault UI:** `http://localhost:8200` · **Docs preview:** `http://localhost:8000`

### Documentation

| Topic | Link |
|---|---|
| Overview | [docs.lyndrix.eu](https://docs.lyndrix.eu/) |
| Architecture | [docs.lyndrix.eu/architecture](https://docs.lyndrix.eu/architecture/) |
| Deployment | [docs.lyndrix.eu/deployment](https://docs.lyndrix.eu/deployment/) |
| High availability | [docs.lyndrix.eu/ha](https://docs.lyndrix.eu/ha/) |
| Plugin development | [docs.lyndrix.eu/plugins](https://docs.lyndrix.eu/plugins/) |
| Security | [docs.lyndrix.eu/security](https://docs.lyndrix.eu/security/) |

## Other repositories

| Repository | Purpose |
|---|---|
| [lyndrix-homepage](https://github.com/lyndrix-platform/lyndrix-homepage) | Public website (Astro) |
| [aac-template-engine](https://github.com/lyndrix-platform/aac-template-engine) | Application-as-Code template engine for declarative service deployment |
| [ansible-ci-image](https://github.com/lyndrix-platform/ansible-ci-image) | Container image for Ansible-based CI pipelines |
| [documentation-framework](https://github.com/lyndrix-platform/documentation-framework) | Shared documentation tooling (modular Markdown, unified MkDocs setup) |

## Contributing

Issues and pull requests are welcome in the individual repositories.

- Code, comments and commit messages are in English.
- Plugins follow the canonical anatomy and import from `core.api` only — see [plugin development](https://docs.lyndrix.eu/plugins/).
- Releases are git-tag driven; a plugin's manifest version must match its tag.
- Please follow the [code of conduct](https://github.com/lyndrix-platform/lyndrix-core/blob/main/CODE_OF_CONDUCT.md), and report security issues privately as described in the [security policy](https://github.com/lyndrix-platform/lyndrix-core/blob/main/SECURITY.md).

## License

The core and the official plugins are released under the **Apache License 2.0**; see the `LICENSE` file in each repository.
