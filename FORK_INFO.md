# Fork tracking — intarweb/excalidraw-full

This is a soft fork of [`betterandbetterii/excalidraw-full`](https://github.com/betterandbetterii/excalidraw-full). We track upstream HEAD and publish a HEAD-tracked image to `ghcr.io/intarweb/excalidraw-full` carrying a small, upstreamable Dockerfile patch that makes the frontend AI features configurable via build-args (defaults unchanged upstream).

> All forks we manage with the sync-upstream + from-source-build pattern live under the [`intarweb`](https://github.com/intarweb) GitHub org. See the `ghcr-fork-mirror` skill for the canonical recipe.

## Branch model

| Branch | Purpose | Source of truth |
|---|---|---|
| `main` | Upstream-clean mirror, hard-reset to `upstream/main` each sync + ops overlay | upstream |
| `intarweb-dev` | Deploy track. `main` + our carried patches. Drives `:latest`. | us |
| `feat/*`, `fix/*` | Individual patch branches we PR upstream | us |

## Upstream sync

| Property | Value |
|---|---|
| Upstream | [`betterandbetterii/excalidraw-full`](https://github.com/betterandbetterii/excalidraw-full) |
| Upstream branch tracked | `main` |
| Sync cadence | Hourly + immediate on PR-branch push + manual `workflow_dispatch` |
| Sync mechanism | hard-reset `main` to `upstream/main` + re-apply ops overlay; regen `intarweb-dev` = `main` + open-PR cherry-picks |
| Sync workflow | [`.github/workflows/sync-upstream.yml`](.github/workflows/sync-upstream.yml) |

## Build pipeline

| Property | Value |
|---|---|
| Image | `ghcr.io/intarweb/excalidraw-full` |
| `:latest` source | our `build-from-source.yml`, built from `intarweb-dev` |
| `:sha-<short>` | Every build |
| Multi-arch | linux/amd64, linux/arm64 |
| Build workflow | [`.github/workflows/build-from-source.yml`](.github/workflows/build-from-source.yml) |
| Submodule note | The frontend is a git submodule (`excalidraw`, SSH URL, branch `multi-canvas`). build-from-source.yml is FORK-LOCAL (SKIP_BFS_HEAL=true): it rewrites SSH→HTTPS + checks out submodules recursively before the docker build. |

## Fleet build-args (passed by our CI only — NOT upstream defaults)

| Build arg | Value | Effect |
|---|---|---|
| `VITE_APP_OPENAI_API_URL` | `/api/v2` | Frontend AI endpoint → same-origin |
| `VITE_APP_OPENAI_MODEL` | `vllm/Qwen3.6-27B` | Text-path model |

Unset = upstream defaults preserved (the Dockerfile ARGs have no defaults).

## Local patches we carry on `intarweb-dev` (vs `upstream/main`)

<!-- intarweb:patches-start -->
| Commit | Subject | Status |
|---|---|---|
| (Dockerfile AI-config ARG/ENV) | `feat(docker): make frontend AI endpoint/model configurable via build-args` | carried on intarweb-dev; upstream PR optional |
<!-- intarweb:patches-end -->

## How to consume

```yaml
# docker-compose.yml
services:
  excalidraw:
    image: ghcr.io/intarweb/excalidraw-full:latest
```

Pin to a specific build for reproducibility:

```yaml
    image: ghcr.io/intarweb/excalidraw-full:sha-<short>
```

## Maintenance recipes

**Manually re-sync onto upstream:**
```bash
gh workflow run "Sync from upstream + auto-regen intarweb-dev" --repo intarweb/excalidraw-full
```

**Force a fresh build of `:latest` without an upstream change:**
```bash
gh workflow run "Build from source → GHCR" --repo intarweb/excalidraw-full --ref intarweb-dev
```
