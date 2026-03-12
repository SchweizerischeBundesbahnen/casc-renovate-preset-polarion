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
- `prCreation: status-success` — PRs created only after CI passes
- `internalChecksFilter: strict` — enforces `minimumReleaseAge` strictly
- **3-day stabilization** for all updates (strictly enforced)
- Lock file maintenance every Monday before 4am (automerged)
- OSV vulnerability alerts with `security` label and no waiting period
- Security PRs get `prPriority: 99` to jump the rate limit queue

### Automerge Rules (inherited by all presets)

| Update Type | Automerged? | Stabilization | Notes |
|-------------|-------------|---------------|-------|
| Minor/patch | ✅ Yes | 3 days | Requires CI to pass |
| Major | ❌ No | — | Manual review required |
| GitHub Actions (any incl. major) | ✅ Yes | 3 days | Grouped into one branch/PR |
| Pre-commit hooks (any incl. major) | ✅ Yes | 3 days | Grouped into one branch/PR, ignoreTests: true |
| Security vulnerabilities | ❌ No | 0 days | Labeled `security`, priority queue |
| Lock file maintenance | ✅ Yes | — | Monday before 4am |
| Org-internal reusable workflows | ⏭️ Skipped | — | `open-source-polarion-java-repo-template` tracked on `@main` |

---

## Deployment

Changes to `default.json` are **live immediately** when merged to `main`. All consuming repositories receive updates on the next Renovate run — there is no staging environment.
