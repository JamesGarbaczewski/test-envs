# test-envs

Sandbox repo for testing deploy promotion rules modeled after `border-operations-service`.

## Structure

Two workflow files:

| File | Role |
|------|------|
| **`deploy.yml`** | Orchestrator — set lifecycle, validate, call deploy |
| **`_deploy_app.yml`** | Shared deploy stub + git tag by lifecycle |

## Promotion rules (in `deploy.yml`)

| Lifecycle | Branch check (`release/*`, `hotfix/*`) | Stg tag check |
|-----------|----------------------------------------|---------------|
| `dev`, `test` | — | — |
| `stg` | Yes | — |
| `prod` | Yes | Yes (`stg` tag must match commit) |

After each deploy, `_deploy_app.yml` moves a floating tag (`dev`, `test`, `stg`, `prod`) to the deployed commit.

## Triggers

| Event | Deploy path |
|-------|-------------|
| Push to `develop` | `dev` (automatic) |
| Manual → `dev` / `test` / `stg` / `prod` | Selected lifecycle |

## Test checklist

### Dev (automatic)

Push or merge to `develop` → **Deploy** runs for `dev`.

### Test

**Actions → Deploy → Run workflow** → select `test` and branch (typically `develop`).

### Stg

1. Cut and push a release branch:
   ```bash
   git checkout -b release/1.0.1
   git push -u origin release/1.0.1
   ```
2. **Actions → Deploy → Run workflow** → select `stg` and branch `release/1.0.1`.
3. Confirm the `stg` tag moves to that commit.

### Prod

1. After stg deploy above, **Actions → Deploy → Run workflow** → `prod` from `release/1.0.1`.
2. Confirm prod completes and `prod` tag moves.

### Failures

1. Manual **Deploy** → `prod` from `main` → fails branch check.
2. Manual **Deploy** → `prod` before stg deploy for that commit → fails stg tag check.
