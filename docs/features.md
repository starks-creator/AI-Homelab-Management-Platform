# Features

## Dashboard

- Single-page status view with live health summary
- Per-host panels (compute, storage, virtualization, endpoints)
- Visual alerts for unhealthy containers or unreachable hosts
- Rolling event log of recent actions

## Monitoring

- Continuous health checks across all hosts
- Docker container status and alerting
- Disk, memory, and uptime tracking per host
- Graceful "unreachable" handling — one offline host doesn't break the dashboard

## Automation

- One-click maintenance actions (service restarts, status refresh)
- Wake-on-LAN for powered-down machines
- Approval queue for higher-risk actions before they execute
- Token-gated action API for safe use by scripts and scheduled jobs

## Virtualization

- Proxmox VM inventory and status
- On-demand VM snapshots ahead of risky changes

## AI Integration

- Local LLM status monitoring via Ollama
- Foundation for AI-assisted operational analysis (Open WebUI)

## Reporting

- Scheduled health summaries delivered to a private notification channel
- JSON status endpoint for external tooling or scripting

## Security

- All action endpoints require authentication
- Higher-risk actions gated behind an approval step
- Every action attempt is logged for auditability
