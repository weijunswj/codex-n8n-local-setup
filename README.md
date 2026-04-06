# Codex + n8n Local Setup

This repository is a small public reference for getting Codex working with a local `n8n` instance on Windows.

It is focused on one job:

- run `n8n` locally in Docker
- connect Codex to `n8n_docs` for workflow design and validation
- connect Codex to `n8n_live` for safe live workflow creation and updates
- optionally expose local `n8n` through Cloudflare Tunnel or ngrok
- keep a clean path to move from localhost to a hosted VPS later

## What This Repo Contains

- [Local setup guide](./codex-n8n-local-setup.md)

That guide is the main deliverable in this repo. It is written as a short exact checklist, not a long tutorial.

## Who This Is For

This is for people who want to:

- use Codex as an n8n workflow builder
- run everything locally first
- keep workflow creation safe by using `n8n_docs` before `n8n_live`
- test webhook flows outside localhost when needed
- move to a hosted environment later without rebuilding the setup from scratch

## What The Setup Covers

- Docker Desktop
- local `n8n`
- Codex MCP config
- `AGENTS.md` routing rules
- `n8n_docs` setup
- `n8n_live` setup
- tunnel options for webhook testing
- Hostinger/VPS migration notes

## Recommended Starting Point

Start with the exact checklist here:

- [codex-n8n-local-setup.md](./codex-n8n-local-setup.md)

## Notes

- The guide assumes Windows.
- The local smoke test uses a tiny no-credentials workflow so you can confirm the stack works before adding third-party apps.
- OAuth app setup for services like Google Sheets can be added after the base Codex + n8n loop is stable.
