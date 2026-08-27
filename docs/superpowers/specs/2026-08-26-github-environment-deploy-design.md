# GitHub Environment Deploy Pipeline — Design

## Goal

Learn and validate GitHub Environment approval behavior using a pipeline that mirrors `border-operations-service` structure, without real deployment infrastructure.

## Architecture

Single orchestrator workflow (`deploy.yml`) sets a `lifecycle` variable based on the triggering event, then calls a reusable stub deploy workflow (`_deploy_app.yml`). GitHub Environment protection rules on `stg` and `prod` cause jobs to pause for approval.

## Triggers and lifecycle mapping

| Trigger | Lifecycle | Approval required |
|---------|-----------|-------------------|
| `workflow_dispatch` → `dev` | `dev` | No |
| `workflow_dispatch` → `test` | `test` | No |
| `workflow_dispatch` → `stg` | `stg` | Yes |
| `workflow_dispatch` → `prod` | `prod` | Yes |
| `push` to `release/**` or `hotfix/**` | `stg` | Yes |
| `release` `published` | `prod` | Yes |

No auto-deploy on `develop`. Dev and test deploys are manual only.

`stg` and `prod` deploys are limited to `release/*` and `hotfix/*` branches via workflow validation and GitHub Environment deployment branch rules.

## GitHub Environments (repo Settings)

| Environment | Protection rules |
|-------------|------------------|
| `dev` | None |
| `test` | None |
| `stg` | Required reviewers; deployment branches `release/*`, `hotfix/*` |
| `prod` | Required reviewers; deployment branches `release/*`, `hotfix/*`; tags allowed for release-triggered deploys |

## Test scenarios

1. Manual dispatch to `dev` or `test` — runs immediately, no approval gate.
2. Manual dispatch to `stg` or `prod` — waits for approval.
3. Push to `release/1.0.1` — auto deploys to `stg`, waits for approval.
4. Publish GitHub Release — auto deploys to `prod`, waits for approval.
