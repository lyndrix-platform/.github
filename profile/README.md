# Lyndrix Platform

**Lyndrix** is an open-source, self-hosted application platform built for teams that want full control over their internal tooling — without building everything from scratch.

## What We Build

At the core of the organization sits **[lyndrix-core](https://github.com/lyndrix-platform/lyndrix-core)**: a secure, extensible runtime framework built on FastAPI, NiceGUI, MariaDB, and HashiCorp Vault. It gives you authentication, secrets management, a live web UI, a database, and an event bus — all pre-wired. You focus on your business logic, not the plumbing.

## Key Concepts

| Concept | What it means |
|---|---|
| **Plugin-First** | Every feature is a plugin. Drop in `setup(ctx)` and the framework does the rest. |
| **Event-Driven** | Components and plugins communicate over a global event bus — no tight coupling. |
| **Vault-Backed Secrets** | HashiCorp Vault with automatic bootstrap, unseal, and per-plugin isolation. |
| **Built-in Auth** | Local, LDAP, and OIDC out of the box. Plugins can register custom providers at runtime. |
| **Managed Dependencies** | Each plugin gets its own `vendor/` directory — no dependency conflicts. |

## Repositories

| Repository | Purpose |
|---|---|
| [lyndrix-core](https://github.com/lyndrix-platform/lyndrix-core) | The main runtime — boot orchestration, plugin lifecycle, secrets, auth, UI |
| [lyndrix-homepage](https://github.com/lyndrix-platform/lyndrix-homepage) | Public-facing website (Astro + Cloudflare) |
| [aac-template-engine](https://github.com/lyndrix-platform/aac-template-engine) | Application as Code template engine for declarative app configuration |
| [ansible-ci-image](https://github.com/lyndrix-platform/ansible-ci-image) | Docker image used for Ansible-based CI pipelines |

## Official Plugins

| Plugin | What it does |
|---|---|
| [lyndrix-plugin-iac-orchestrator](https://github.com/lyndrix-platform/lyndrix-plugin-iac-orchestrator) | Terraform & Ansible IaC orchestration |
| [lyndrix-plugin-docker-manager](https://github.com/lyndrix-platform/lyndrix-plugin-docker-manager) | Docker container and image management |
| [lyndrix-plugin-monitoring](https://github.com/lyndrix-platform/lyndrix-plugin-monitoring) | System and service monitoring with alerting |
| [lyndrix-plugin-server-manager](https://github.com/lyndrix-platform/lyndrix-plugin-server-manager) | Server lifecycle and inventory management |
| [lyndrix-plugin-external-services](https://github.com/lyndrix-platform/lyndrix-plugin-external-services) | Third-party API integrations |
| [lyndrix-plugin-discord-notifier](https://github.com/lyndrix-platform/lyndrix-plugin-discord-notifier) | Discord notifications for alerts and events |
| [lyndrix-plugin-meeting-bingo](https://github.com/lyndrix-platform/lyndrix-plugin-meeting-bingo) | Meeting Bingo — any use case is a valid use case |
| [lyndrix-plugin-collection](https://github.com/lyndrix-platform/lyndrix-plugin-collection) | Official plugin collection index |

## Get Started

```bash
git clone https://github.com/lyndrix-platform/lyndrix-core.git
cd lyndrix-core
docker compose -f docker/docker-compose.dev.yml up -d --build
```

- **App:** `http://localhost:8081`
- **Docs:** `http://localhost:8000` · [lyndrix-platform.github.io/lyndrix-core](https://lyndrix-platform.github.io/lyndrix-core/)

## License

All repositories are released under the **MIT** or **Apache 2.0** license.
