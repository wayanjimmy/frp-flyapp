---
name: deploy-fly
description: Deploy FRP to Fly.io and verify service health. Use when deploying changes to production.
---

# Deploy to Fly.io

Deploy the current repository state to Fly.io and verify the FRP server is healthy.

## Scope

**In scope:** `fly deploy`, post-deploy health checks, rollback steps.

**Out of scope:** Changing FRP version or patches (use `upgrade-frp` skill).

## Prerequisites

- `flyctl` installed and authenticated: `fly auth whoami`
- Correct Fly app configured in `fly.toml`
- Required secrets set (verify with `fly secrets list`):
  - `FRP_DASHBOARD_PWD`
  - `FRP_TOKEN`

## Steps

### 1. Pre-deploy Checks

```bash
fly status
fly secrets list
```

Confirm you're on the correct branch/commit.

### 2. Deploy

```bash
fly deploy
```

Watch the build output for:
- Patches applying cleanly
- Web dashboard building
- Go compilation succeeding

### 3. Post-deploy Verification

Check status:
```bash
fly status
```

Check logs for FRP startup:
```bash
fly logs
```

Verify dashboard is accessible:
- Visit `https://<app-name>.fly.dev/` (port 8080 via Fly proxy)

### 4. Record Deployment

Note the deployed version:
- Git SHA: `git rev-parse --short HEAD`
- FRP version: Check `ARG VERSION` in Dockerfile

## Rollback

If deployment fails or service is unhealthy:

### List Previous Releases

```bash
fly releases
```

### Rollback to Previous Version

```bash
fly deploy --image registry.fly.io/<app-name>:<previous-deployment-id>
```

Or redeploy from a known-good git commit:
```bash
git checkout <good-sha>
fly deploy
```

## Exposed Ports

The deployment exposes these ports (defined in `fly.toml`):

| Port | Protocol | Purpose |
|------|----------|---------|
| 7000 | TCP/UDP | FRP server |
| 7001 | UDP | FRP KCP/QUIC |
| 8080 | TCP | Dashboard (internal) |
| 25565 | TCP/UDP | Minecraft proxy |
| 19132 | UDP | Bedrock proxy |

## Guardrails

- Never change secrets during routine deploy unless explicitly requested
- If deploy fails: stop, rollback, then investigate locally
- Verify local build succeeds before deploying (use `upgrade-frp` skill)
