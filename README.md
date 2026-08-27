# test-envs

Sandbox repo for testing GitHub Environment approval behavior in a deploy pipeline modeled after `border-operations-service`.

## Setup

Create four environments in **Settings → Environments**:

| Environment | Required reviewers | Notes |
|-------------|-------------------|-------|
| `dev` | None | Manual dispatch only |
| `test` | None | Manual dispatch only |
| `stg` | Add yourself (or a team) | Used for release/hotfix branch deploys |
| `prod` | Add yourself (or a team) | Used for GitHub Release deploys |

Add yourself as a required reviewer on `stg` and `prod` so you can approve deployments from the Actions UI.

### Deployment branches and tags (`stg` and `prod`)

On each of `stg` and `prod`, open **Deployment branches and tags** → **Selected branches and tags**:

**Branches**
- `release/*`
- `hotfix/*`

**Tags** (required for `prod` only)

GitHub Release deploys run on a tag ref, not a branch ref. On `prod`, also allow tags so release-triggered deploys can reach the environment gate:

- `*` (permissive, fine for a test repo), or
- a semver pattern like `*.*.*` if you prefer

The workflow `validate-ref` job still enforces that prod releases must point at a commit on a `release/*` or `hotfix/*` branch, even when the run ref is a tag.

### Stg-before-prod promotion

After a successful deploy, the workflow moves a floating git tag (`dev`, `test`, `stg`, or `prod`) to the deployed commit.

Before prod, `validate-ref` fetches the `stg` tag and checks it points at `${{ github.sha }}`. If not, prod fails.

Expected flow: push `release/1.0.1` → approve stg deploy (`stg` tag moves) → publish GitHub Release → prod validates `stg` tag → approve prod deploy (`prod` tag moves).

You can also inspect what's deployed per environment anytime via **Tags** in GitHub (`dev`, `test`, `stg`, `prod`).

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
| Push to `release/**` or `hotfix/**` | `stg` | Yes |
| Publish GitHub Release | `prod` | Yes |
| Manual run → `stg` / `prod` from other branches | — | Blocked by workflow + environment rules |

Pushing to `develop` does not trigger a deploy. Manual `stg`/`prod` runs must use the **branch dropdown** to select a `release/*` or `hotfix/*` branch.

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

### Phase 3 — branch restrictions

1. Try manual dispatch to `stg` from `main` → `validate-ref` should fail.
2. Try manual dispatch to `stg` from `release/1.0.1` → should proceed to approval.
3. Push `hotfix/1.0.2` → should deploy to `stg` after approval.
4. Publish a release whose tag is **not** on a release/hotfix branch → `validate-ref` should fail.

### Phase 4 — stg before prod

1. Publish a GitHub Release **without** deploying the same commit to stg first → prod should fail at validation.
2. Push `release/1.0.1`, approve stg deploy, then publish the release → prod should proceed to approval.
