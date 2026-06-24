# Jarvis Homelab

## Overview

Jarvis is a self-hosted monitoring and automation platform for a multi-node homelab. It consolidates the health and status of every machine in the environment into a single dashboard, and exposes a small, token-gated set of operational actions so routine maintenance no longer requires logging into each system individually.

## Problem Statement

As a homelab grows beyond a single machine, operational visibility becomes fragmented. Checking on services means juggling multiple SSH sessions and remembering different commands for different hosts, and there's no single place to see whether everything is actually healthy. Jarvis addresses this by centralizing status collection and a curated set of safe, auditable actions behind one interface.

## Key Capabilities

- Infrastructure health monitoring across multiple hosts
- Docker container monitoring and alerting
- Proxmox virtual machine visibility
- Wake-on-LAN controls for powered-down machines
- Automated status reporting
- Local AI service monitoring (Ollama)
- Snapshot and maintenance operations
- Event logging and audit history

## Technology Stack

- Python / Flask
- Bash / SSH
- Docker
- Proxmox VE
- Ollama
- HTML / CSS (single-page dashboard, no frontend framework)

## Architecture

Jarvis runs as a single Flask application with three responsibilities:

1. **Status collection** — a bash script gathers local system metrics, Docker container state, and AI service status, and pulls equivalent data from remote hosts over SSH with short timeouts so an offline machine degrades gracefully instead of breaking the dashboard.
2. **Presentation** — the Flask app parses and caches that status, then renders it as a single dashboard page and exposes it as JSON over a `/status` endpoint.
3. **Action execution** — a set of token-gated endpoints perform maintenance operations (service restarts, VM snapshots, Wake-on-LAN). Higher-risk actions can be routed through an approve/reject queue rather than executing immediately, and every attempt is recorded in an event log.

See [docs/architecture.md](docs/architecture.md) for a detailed breakdown.

## Features

- Unified, real-time status dashboard across all hosts in the lab
- Per-host health detail: uptime, disk usage, service status, container health
- Container-level alerting for unhealthy or stopped services
- One-click maintenance actions (service restarts, snapshots, Wake-on-LAN)
- Approval queue for higher-risk actions
- Token-gated automation API for safe use from scripts and scheduled jobs
- Rolling event log for auditability

See [docs/features.md](docs/features.md) for the full list.

## Screenshots

Placeholders — see [docs/screenshots.md](docs/screenshots.md).

## Future Improvements

- Historical metrics and trend graphs instead of point-in-time snapshots
- Push notifications instead of dashboard polling
- Multi-approver support for the action queue
- Containerized deployment of Jarvis itself
- A proper frontend build replacing the current inline-HTML dashboard
- Extend monitoring to a Windows Server VM running core network services (Active Directory, DNS, DHCP)

## Project Docs

- [Architecture](docs/architecture.md)
- [Features](docs/features.md)
- [Security](docs/security.md)
- [Screenshots](docs/screenshots.md)

## Lessons Learned

Building Jarvis reinforced a few practical lessons:

- **Graceful degradation matters more than perfect uptime.** Designing every status check to fail independently (per host, per service) made the dashboard far more useful than one that breaks when a single machine is offline.
- **Gate destructive actions behind review, not just auth.** Token authentication alone wasn't enough peace of mind for actions like snapshots or restarts — adding an approve/reject queue made it safe to automate without losing control.
- **Keep the operational surface small.** Resisting the urge to expose every possible command kept the action API easy to reason about and secure.
- **Logging pays for itself immediately.** A simple event log made debugging "what just happened" far faster than reconstructing it from service logs across multiple hosts.

---

This repository is a sanitized portfolio version of a personal homelab project. Hostnames, internal IP addresses, tokens, webhook URLs, and other private configuration have been removed. See [docs/security.md](docs/security.md) for details.
