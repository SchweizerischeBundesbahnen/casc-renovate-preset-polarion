# casc-renovate-preset-polarion

**Base Renovate preset for all Polarion GitHub repositories**

This is the shared base preset. Use an ecosystem-specific preset instead:

| Ecosystem | Preset |
|-----------|--------|
| Docker / Dockerfile | [casc-renovate-preset-polarion-docker](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-docker) |
| Python (uv/pep621) | [casc-renovate-preset-polarion-python](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-python) |
| Java (Maven) | [casc-renovate-preset-polarion-java](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java) |

---

## What This Base Preset Provides

All ecosystem presets extend this and inherit:

- `config:best-practices` + semantic commits
- Branch-based automerge with platform automerge enabled
- `prCreation: not-pending` — no pull request while a branch status check is still queued or running
- `internalChecksFilter: strict` — enforces `minimumReleaseAge` strictly
- **3-day stabilization** by default, with two exceptions: internal SBB packages and images wait `0 days`, and a `docker` or `github-tags` release counts as aged when it carries no timestamp
- Lock file maintenance every Monday before 4am (automerged)
- OSV vulnerability alerts with `security` label and no waiting period
- Vulnerability pull requests are prioritised and bypass the rate limits — Renovate does this natively for every alert, so this preset adds no rule for it
- Every dependency update is typed `fix`, except workflow actions, pre-commit hooks and lock-file updates, which stay `chore`

### Automerge Rules (inherited by all presets)

| Update Type | Automerged? | Stabilization | Notes |
|-------------|-------------|---------------|-------|
| Minor/patch | ✅ Yes | 3 days, 0 for internal SBB | Requires CI to pass |
| Major | ❌ No | — | Manual review required |
| GitHub Actions (any incl. major) | ✅ Yes | 3 days, none if the tag carries no timestamp | Grouped into one branch/PR |
| Pre-commit hooks (any incl. major) | ✅ Yes | 3 days, none if the tag carries no timestamp | Grouped into one branch/PR |
| Security vulnerabilities | ❌ No | 0 days | Labeled `security`; Renovate prioritises these natively |
| Lock file maintenance | ✅ Yes | — | Monday before 4am |
| Org-internal reusable workflows | ⏭️ Skipped | — | `github-workflows-polarion` tracked on `@main` |

### Semantic Commit Types

`fix` for every dependency update, so the type does not depend on which dependency section a package happens to sit in. Upstream `:semanticPrefixFixDepsChoreOthers` types only runtime dependencies `fix`, which left Maven `test` scope, dev dependency groups and npm `devDependencies` reading `chore` for the same library.

| Update | Type |
|--------|------|
| Any dependency | `fix` |
| GitHub Actions | `chore` |
| Pre-commit hooks | `chore` |
| Lock file maintenance, lock-file-only updates | `chore` |

`fix` is a releasing type, so in a repository running release-please or a comparable tool a dependency bump now cuts a patch release. The three carve-outs are what keeps build and commit-gate tooling from doing the same.

---

## Deployment

Changes to `default.json` are **live immediately** when merged to `main`. All consuming repositories receive updates on the next Renovate run — there is no staging environment.
