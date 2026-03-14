# Agent & AI Coding Guidelines

This document describes how to work on this codebase, both for human developers
and for AI coding agents (OpenCode, Claude, Copilot, etc.).

---

## Project Overview

**Stack:** Next.js 14, TypeScript, Tailwind CSS  
**Runtime:** Node.js 20 on a Raspberry Pi (ARM64)  
**Process manager:** PM2  
**Tunnel / routing:** Cloudflare Tunnel (`jamjar`)

---

## Deployment Model

> Direct file edits on the server are no longer the deployment method.
> All changes must go through GitHub and the CI/CD pipeline described below.

### Flow

```
Local machine / AI agent
        |
        | git push origin main
        v
   GitHub (main branch)
        |
        | triggers GitHub Actions workflow
        v
  Self-hosted runner on Pi
  (GitHub Actions runner process, shell executor)
        |
        | 1. git checkout
        | 2. npm ci
        | 3. npm run build
        | 4. pm2 restart
        v
  Live site (served via PM2 + Cloudflare Tunnel)
```

### Key files

| File | Purpose |
|---|---|
| `.github/workflows/deploy.yml` | CI/CD pipeline definition |
| `AGENTS.md` | This file — coding and deployment guidelines |
| `README.md` | User-facing deployment instructions |

---

## Working on the Codebase

### Making changes

1. Work on a feature branch or directly on `main` for small fixes.
2. Commit and push to `main` when ready to deploy.
3. GitHub Actions picks up the push automatically — no manual SSH required.

### Running locally

```bash
npm install
npm run dev       # dev server on http://localhost:3000
npm run build     # production build
npm run lint      # ESLint
```

### Do not

- Do **not** edit files directly on the Pi via SSH as a deployment method.
- Do **not** manually restart PM2 as a substitute for a proper deploy — always
  push through Git so the pipeline handles `npm ci`, `build`, and restart in
  the correct order.
- Do **not** commit `node_modules/`, `.next/`, or `.env*` files.

---

## Environment Variables

Secrets and environment config are **not** stored in the repository.  
They must be set on the Pi directly or via GitHub Actions secrets:

- GitHub Actions secrets: `Settings → Secrets and variables → Actions`
- Pi-side env: create `/home/winter/turfrig-tmp/.env.local` (gitignored)

---

## CI/CD Pipeline Details

The workflow lives at `.github/workflows/deploy.yml`.

- **Trigger:** push to `main`
- **Runner:** `self-hosted` (the Pi, registered as a GitHub Actions runner)
- **Steps:**
  1. `actions/checkout@v4` — checks out the latest code
  2. `npm ci` — clean install of dependencies
  3. `npm run build` — Next.js production build
  4. `pm2 restart all` — zero-downtime application restart

If the PM2 process does not exist yet (first deploy), the last step falls back
to `pm2 start npm --name "turfrig" -- start`.

---

## GitHub Runner

The self-hosted runner must be running on the Pi for deployments to work.  
See `README.md` for setup instructions.

The runner service is registered under the `winter` user and runs as a
persistent background service via `./svc.sh install && ./svc.sh start`.
