# Architecture

## Overview

Jarvis is a centralized monitoring and automation platform for a multi-node homelab. A single Flask application serves as the hub: it collects health and status data from every node in the environment, presents that data through a unified dashboard, and exposes a controlled set of operational actions that can be triggered manually or by scheduled automation.

The environment consists of distinct functional roles rather than a single machine, and Jarvis is designed to provide one coherent view across all of them.

## Component Roles

| Component | Role |
|---|---|
| **Mini-PC (AI + Automation)** | Hosts the Jarvis application itself, along with local AI/LLM services. Acts as the central point from which status is collected and actions are dispatched. |
| **Dell Server (Media + Storage)** | Provides media services and primary storage. Reports service health, disk usage, and container status back to Jarvis. |
| **Proxmox Host** | Hypervisor node running virtual machines. Reports VM inventory and supports snapshot operations as part of maintenance workflows. |
| **Windows Endpoints** | Personal workstations included in health monitoring for uptime and basic resource visibility. |
| **Tailscale** | Provides the private network overlay that connects all nodes, allowing Jarvis to reach each host securely without exposing services to the public internet. |
| **AI Services** | Local LLM workloads used for vision, automation assistance, and operational analysis through Ollama and Open WebUI. |

## Data Flow

1. **Collection** — A status-collection script runs on the Mini-PC. It gathers local metrics directly (memory usage, Docker container state, AI service status) and connects to the Dell Server, the Proxmox host, and the Windows endpoints over the Tailscale network using SSH, with short connection timeouts.
2. **Aggregation** — The Flask application parses the output of the collection script into a structured, per-host representation and caches it briefly to avoid issuing redundant SSH calls on every dashboard load.
3. **Presentation** — The aggregated status is rendered as a single dashboard view and is also available as a JSON endpoint for programmatic consumption.
4. **Degradation handling** — If any individual host is unreachable, that host is reported as offline rather than causing a failure elsewhere in the system, so the dashboard always reflects the best available picture of the environment.

```
            ┌────────────────────────────┐
            │      Dashboard (Browser)    │
            └──────────────┬─────────────┘
                            │ HTTP
                            ▼
            ┌────────────────────────────┐
            │   Jarvis (Mini-PC)          │
            │   - Status aggregation      │
            │   - Action dispatch         │
            │   - Event logging           │
            └───┬───────────────────┬─────┘
                │                   │
     Local data │                   │ SSH over Tailscale
                ▼                   ▼
   ┌──────────────────┐   ┌────────────────────────────┐
   │  AI / Automation  │   │  Dell Server (Media/Storage)│
   │  services          │   │  Proxmox Host                │
   │                    │   │  Windows Endpoints            │
   └──────────────────┘   └────────────────────────────┘
                            │
                            ▼
            ┌────────────────────────────┐
            │   Automated Reporting       │
            │   (status summaries)        │
            └────────────────────────────┘
```

## Monitoring

Jarvis continuously evaluates the health of each component:

- **Mini-PC** — memory utilization, running containers, AI service availability
- **Dell Server** — disk usage, key service status, container health
- **Proxmox Host** — virtual machine inventory and state
- **Windows Endpoints** — uptime and basic resource availability

Container and service health are evaluated against expected state, and any deviation (a stopped or unhealthy container, an unreachable host) is surfaced as an alert on the dashboard rather than requiring manual inspection.

## Automation

A defined set of operational actions can be triggered from the dashboard or invoked programmatically:

- Service restarts on the Dell Server
- Virtual machine snapshots on the Proxmox Host
- Wake-on-LAN to power on offline endpoints
- Forced status refresh across all hosts

Actions are implemented as discrete, auditable operations rather than open-ended command execution. Higher-risk actions can be routed through an approval step before they are executed, separating the request from the action itself. Every action attempt, successful or not, is recorded in an event log.

## Reporting

A scheduled job periodically generates a summary of overall environment health and delivers it through an outbound notification channel. The destination and any associated credentials are configured outside of the application and are not part of this repository.

## Security Boundary

The public version of this project excludes all credentials, network identifiers, private configuration, and operational secrets. Security-sensitive configuration is managed outside of source control. See [security.md](security.md) for details.
