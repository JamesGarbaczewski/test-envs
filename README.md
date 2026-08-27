# test-envs

Sandbox repo for testing GitHub Environment approval behavior in a deploy pipeline modeled after `border-operations-service`.

## Setup

Create four environments in **Settings → Environments**:

| Environment | Required reviewers | Notes |
|-------------|-------------------|-------|
| `dev` | None | Manual dispatch only |
| `test` | None | Manual dispatch only |
| `stg` | Add yourself (or a team) | Used for release branch deploys |
| `prod` | Add yourself (or a team) | Used for GitHub Release deploys |

Add yourself as a required reviewer on `stg` and `prod` so you can approve deployments from the Actions UI.

## Workflows

- **`deploy.yml`** — sets lifecycle from the trigger event, then calls `_deploy_app.yml`
- **`_deploy_app.yml`** — reusable stub deploy; the `environment:` key drives approval gates

## Triggers

| Event | Target environment | Approval? |
|-------|-------------------|-----------|
| Manual run → `dev` | `dev` | No |
| Manual run → `test` | `test` | No |
| Manual run → `stg` | `stg` | Yes |
| Manual run → `prod` | `prod` | Yes |
| Push to `release/**` | `stg` | Yes |
| Publish GitHub Release | `prod` | Yes |

Pushing to `develop` does not trigger a deploy.

## Test checklist

### Phase 1 — manual environments

1. **Actions → Deploy → Run workflow** → choose `dev` → should complete without approval.
2. Run again → choose `test` → should complete without approval.
3. Run again → choose `stg` → job shows **Waiting for approval** → approve → completes.
4. Run again → choose `prod` → same approval flow.

### Phase 2 — release flow

1. Create and push a release branch:
   ```bash
   git checkout -b release/1.0.1
   git push -u origin release/1.0.1
   ```
2. Confirm the Deploy workflow runs for `stg` and waits for approval.
3. Create a GitHub Release from `release/1.0.1` (tag e.g. `1.0.1`).
4. Confirm a second Deploy workflow runs for `prod` and waits for approval.

## Future: deployment branch/tag rules

After the basic approval flow works, configure **Deployment branches and tags** on `stg` and `prod` to restrict which refs can deploy (e.g. `release/**` for stg, release tags for prod).
