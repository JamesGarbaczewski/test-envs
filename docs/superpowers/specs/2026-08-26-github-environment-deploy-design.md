# GitHub Deploy Promotion Pipeline — Design

## Goal

Validate deploy promotion rules for stg/prod without GitHub Environment required reviewers.

## Structure

- **`deploy.yml`** — orchestrator (lifecycle, validation, deploy calls)
- **`_deploy_app.yml`** — reusable deploy stub + lifecycle tag

## Core rules

1. **Branch restriction** — stg and prod deploys only from `release/*` or `hotfix/*` branches.
2. **Stg-before-prod** — prod deploy requires the `stg` git tag to point at `${{ github.sha }}`.

## Triggers

| Trigger | Path |
|---------|------|
| Push `develop` | dev (automatic) |
| Manual | selected lifecycle; stg/prod include promotion checks |

## Tagging

After each deploy, `_deploy_app.yml` moves a floating tag matching the lifecycle name.
