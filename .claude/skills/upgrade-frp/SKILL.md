---
name: upgrade-frp
description: Upgrade FRP version in Dockerfile and verify patches apply cleanly with local build. Use when upgrading FRP to a new version.
---

# Upgrade FRP

Upgrade the FRP (Fast Reverse Proxy) version used by this Fly.io deployment.

## Scope

**In scope:** Finding latest FRP version, bumping Dockerfile, validating build/patch application.

**Out of scope:** Deploying to Fly.io (use `deploy-fly` skill).

## Prerequisites

- Clean git working tree (recommended)
- `podman` or `docker` installed
- Network access to github.com

## Steps

### 1. Check Latest FRP Version

```bash
curl -s https://api.github.com/repos/fatedier/frp/releases/latest | jq -r .tag_name
```

Or use GitHub CLI:
```bash
gh release view --repo fatedier/frp --json tagName -q .tagName
```

### 2. Update Dockerfile

Edit the `ARG VERSION` line in `Dockerfile`:

```dockerfile
ARG VERSION=0.XX.X
```

Note: Use version without the `v` prefix (e.g., `0.67.0` not `v0.67.0`).

### 3. Build Locally to Validate

```bash
podman build -t frp-flyapp:test .
```

**Success criteria:**
- Tarball downloads successfully
- Both patches apply cleanly:
  - `patches/udp-proxy-fly-global-services.patch`
  - `patches/kcp-quic-fly-global-services.patch`
- Web dashboard builds (`npm install && npm run build` in `web/frps`)
- Go compilation succeeds (`go install ./cmd/frps`)

### 4. Commit the Change

```bash
git add Dockerfile
git commit -m "chore: bump frp to vX.X.X"
```

## Troubleshooting

### Patch Fails to Apply

If patches fail with conflicts:

1. Check the patch output for which hunks failed
2. Look at upstream changes in the affected files:
   - `server/proxy/udp.go`
   - `server/service.go`
3. Manually update the patch to match new code structure
4. Regenerate patch if needed:
   ```bash
   diff -u original.go modified.go > patches/patch-name.patch
   ```

### npm Build Fails

- Check if FRP changed their web dashboard structure
- Verify Node.js/npm compatibility with Alpine packages

### Download Fails

- Confirm the tag exists: `https://github.com/fatedier/frp/releases/tag/vX.X.X`
- Ensure version format matches (no `v` prefix in Dockerfile)

## Output

- Updated `Dockerfile` with new `ARG VERSION`
- Verified local build
- Git commit ready for deploy
