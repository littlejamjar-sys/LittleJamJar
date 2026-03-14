# turfrig

A Next.js web application running on a Raspberry Pi, served via Cloudflare Tunnel.

---

## How Deployment Works

**The short version:** push to `main` and the site updates itself.

When you push code to the `main` branch on GitHub, a GitHub Actions workflow
automatically runs on the Pi, builds the app, and restarts it. You never need
to SSH in and manually copy files or restart processes.

```
You push to GitHub main
        |
        v
GitHub Actions triggers the deploy workflow
        |
        v
The Pi (self-hosted runner) pulls the code, runs npm ci + npm run build
        |
        v
PM2 restarts the app — site is live with the new code
```

### Making a change

1. Edit code locally (or via an AI agent like OpenCode/Claude).
2. Commit your changes.
3. Push to `main`:
   ```bash
   git push origin main
   ```
4. Watch the deployment in the GitHub Actions tab of your repository.
5. The site is live within ~1-2 minutes.

### Checking deployment status

Go to your GitHub repository → **Actions** tab → select the latest workflow run.
Green = deployed. Red = something failed — click the run to see the logs.

---

## First-Time Setup

### 1. Create the GitHub repository

```bash
cd /home/winter/turfrig-tmp
git init
git add .
git commit -m "initial commit"
git branch -M main
git remote add origin git@github.com:YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

### 2. Register the Pi as a GitHub Actions self-hosted runner

This only needs to be done once.

**On GitHub:** Go to your repository → Settings → Actions → Runners →
**New self-hosted runner** → choose **Linux** / **ARM64**.

GitHub will give you a set of commands to run. They look like this (use the
exact commands GitHub provides, as the token changes each time):

```bash
# On the Pi, as the winter user:
mkdir -p ~/actions-runner && cd ~/actions-runner

# Download the runner package (GitHub gives you the exact URL)
curl -o actions-runner-linux-arm64.tar.gz -L https://github.com/actions/runner/releases/download/vX.X.X/actions-runner-linux-arm64-X.X.X.tar.gz
tar xzf ./actions-runner-linux-arm64.tar.gz

# Configure (use the token and URL from GitHub's instructions)
./config.sh --url https://github.com/YOUR_USERNAME/YOUR_REPO --token YOUR_TOKEN

# Install and start as a persistent background service
sudo ./svc.sh install
sudo ./svc.sh start
```

The runner will now start automatically on boot and listen for new deployments.

### 3. Verify the runner is online

On GitHub: repository → Settings → Actions → Runners. The Pi should show as
**Idle** (green dot). If it shows **Offline**, restart the service:

```bash
cd ~/actions-runner
sudo ./svc.sh restart
```

### 4. Trigger your first deployment

Push any commit to `main`. The Actions tab should show a new workflow run
within seconds.

---

## Local Development

```bash
npm install
npm run dev     # starts dev server at http://localhost:3000
npm run build   # production build (same as CI runs)
npm run lint    # lint check
```

---

## Environment Variables

Do not commit secrets or environment-specific config to the repository.

- **Pi-side config:** create `/home/winter/turfrig-tmp/.env.local` (already
  gitignored). This file persists across deployments because the runner checks
  out code into the same directory.
- **GitHub Actions secrets** (if the build step needs env vars): repository →
  Settings → Secrets and variables → Actions → New repository secret.

---

## Infrastructure Overview

| Component | Details |
|---|---|
| Server | Raspberry Pi, ARM64, `winter` user |
| Runtime | Node.js 20 via PM2 |
| Tunnel | Cloudflare Tunnel (`jamjar`) |
| CI/CD | GitHub Actions, self-hosted runner |
| Workflow | `.github/workflows/deploy.yml` |

For AI agent and developer guidelines, see [AGENTS.md](./AGENTS.md).
