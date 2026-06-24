# Security Notes

This repository is a **sanitized portfolio version** of a private home lab project. The following were intentionally removed or generalized before publishing, and should never be added back:

## Excluded from this repo

- **Action tokens** — the live system gates every `/actions/*` endpoint behind a private token (`.action_token`, loaded from a local file or environment variable, never hardcoded). No real token is included here.
- **Discord webhook URLs** — the live system posts daily reports to a private Discord channel via webhook. The webhook URL is loaded from a local, gitignored file at runtime and is not present in this repo.
- **Hostnames / internal IPs** — real hostnames, LAN addresses, and SSH targets from the live lab have been replaced with generic descriptions (e.g. "media server," "hypervisor," "workstation") in the docs.
- **SSH keys / credentials** — none are included in this repository.
- **Tailscale / VPN config** — any private networking configuration used to reach lab hosts remotely is excluded.
- **Logs** — operational logs can contain hostnames, IPs, and usage patterns, and are excluded via `.gitignore`.

## Design-level security notes

- All mutating endpoints (`/actions/*`) require a token passed via a custom header; there is no unauthenticated write path.
- Higher-risk actions (snapshots, restarts) can be routed through an approve/reject queue instead of executing immediately, adding a human checkpoint.
- Every action attempt — success or failure — is recorded in an event log for auditability.
- SSH calls to remote hosts use short connect timeouts and batch mode, so a single offline or misbehaving host can't hang the whole status check.

## If you fork this project

Generate your own action token and webhook configuration locally, store them outside of version control (e.g. a gitignored `.env` or local file), and update `.gitignore` to match whatever secret files your deployment introduces.
