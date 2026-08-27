# test-envs

Sandbox repo for testing GitHub Environment approval behavior in a deploy pipeline modeled after `border-operations-service`.

## Setup

Create four environments in **Settings → Environments**:

| Environment | Required reviewers | Deployment branches (optional) |
|-------------|-------------------|-------------------------------|
| `dev` | None | — |
| `test` | Yes | — |
| `stg` | Yes | `release/*`, `hotfix/*` |
| `prod` | Yes | `release/*`, `hotfix/*` |

Add yourself as a required reviewer on `test`, `stg`, and `prod`.

## Workflows

Two pipelines, each with sequential environment gates in a **single run**:

| Workflow | Trigger | Flow |
|----------|---------|------|
| **Deploy Dev Test** (`deploy-dev-test.yml`) | Push to `develop` (merge) | `dev` (auto) → **approve** `test` |
| **Deploy Stg Prod** (`deploy-stg-prod.yml`) | Push to `release/**` or `hotfix/**` | **approve** `stg` → **approve** `prod` |

Both call **`_deploy_app.yml`**, a reusable stub deploy. The `environment:` key on each job drives the approval gate.

After each successful deploy, a floating git tag (`dev`, `test`, `stg`, `prod`) moves to that commit so you can see what's deployed under **Tags**.

## Flow diagrams

**Merge to develop**

```
push develop  →  deploy dev (no gate)  →  wait for test approval  →  deploy test
```

**Cut release branch**

```
push release/1.0.1  →  wait for stg approval  →  deploy stg  →  wait for prod approval  →  deploy prod
```

## Manual runs

Both workflows support `workflow_dispatch` for testing:

- **Deploy Dev Test** — run from `develop` (or any branch for experimentation)
- **Deploy Stg Prod** — run from a `release/*` or `hotfix/*` branch

## Test checklist

### Dev → Test pipeline

1. Push or merge to `develop`
2. Confirm `deploy-dev` completes without approval
3. Confirm run pauses on `deploy-test` → approve → completes

### Stg → Prod pipeline

1. Create and push a release branch:
   ```bash
   git checkout -b release/1.0.1
   git push -u origin release/1.0.1
   ```
2. Confirm run pauses on `deploy-stg` → approve → stg deploy completes
3. Confirm run pauses on `deploy-prod` → approve → prod deploy completes

### Environment branch rules (optional)

On `stg` and `prod`, configure **Deployment branches and tags** to `release/*` and `hotfix/*` as a second layer of protection in GitHub settings.
