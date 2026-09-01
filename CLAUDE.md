# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains the **shared base Renovate preset** for all Polarion GitHub repositories at SBB.

It is extended by ecosystem-specific presets — **do NOT use this preset directly in repositories**:
- [casc-renovate-preset-polarion-docker](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-docker)
- [casc-renovate-preset-polarion-python](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-python)
- [casc-renovate-preset-polarion-java](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java)

## Architecture

```
casc-renovate-preset-polarion/
├── default.json     # Shared base preset (PRODUCTION)
├── renovate.json    # Self-test configuration (dogfooding)
├── README.md        # Documentation
└── CLAUDE.md        # This file
```

**`default.json`** is consumed by child presets via:
```json
{ "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion"] }
```

## What Lives Here vs Child Presets

**Base preset (`default.json`) contains:**
- `config:best-practices` + `:semanticCommits`
- All top-level settings (automergeType, prCreation, internalChecksFilter, etc.)
- `lockFileMaintenance` (Monday before 4am)
- Automerge `packageRules`: non-major → major block → internal-SBB stability skip → docker/github-tags timestamp-optional → github-actions → pre-commit → skip org-internal workflows
- Semantic-commit-type `packageRules`: catch-all `fix` → chore carve-outs for github-actions, pre-commit and lock-file-only updates
- `osvVulnerabilityAlerts` + `vulnerabilityAlerts`

**Child presets add only:**
- `enabledManagers` (ecosystem-specific)
- `constraints` (python version, where applicable)
- Ecosystem-specific `packageRules` (Maven registry, Python version lock, SBB packages, etc.)

## packageRules Ordering (Critical)

Rules are applied in order — **later rules override earlier ones**. The base preset order is intentional:

**Automerge and stability (1-7):**

1. Automerge non-major (minor/patch/pin/digest)
2. Block major updates (`automerge: false`)
3. Skip the stability wait for internal SBB packages and images (`minimumReleaseAge: "0 days"`)
4. Treat docker and github-tags as stable without a release timestamp (`minimumReleaseAgeBehaviour: "timestamp-optional"`) — without this they never leave `pending`
5. Override: github-actions — automerge all including major, `groupName: "github-actions"` (single branch/PR)
6. Override: pre-commit — automerge all including major, `groupName: "pre-commit hooks"` (single branch/PR)
7. Skip: `github-workflows-polarion` — org-internal reusable workflows tracked on `@main`, disabled from Renovate

**Semantic commit type (8-11):**

8. Catch-all: every dependency update is typed `fix`, overriding upstream `:semanticPrefixFixDepsChoreOthers`, which types only runtime dependencies `fix` and everything else `chore`
9. Carve-out: github-actions stays `chore`
10. Carve-out: pre-commit stays `chore`
11. Carve-out: a lock-file-only update (`isLockfileUpdate`) stays `chore`

`lockFileMaintenance` needs no carve-out and must not be given one: it carries no package name, and `PackageNameMatcher` returns `false` on a falsy `packageName` before `matchRegexOrGlob` applies its `pattern === "*"` short-circuit — so rule 8 never reaches it and it keeps the global `semanticCommitType`. A rule restating that would validate cleanly and never fire.

**No security packageRule, deliberately.** `matchCategories` is compared against the closed `Categories` list in `lib/constants/category.ts` — 26 *manager* categories, no `security` among them — so a `matchCategories: ["security"]` rule validates and never fires. Neither replacement works either: `prPriority` inside `vulnerabilityAlerts` makes `renovate-config-validator` warn in every consumer repository, and `matchJsonata: ["isVulnerabilityAlert = true"]` validates cleanly but is appended after the generated vulnerability rules, so it can never match. None is needed: Renovate sorts vulnerability branches first ahead of `prPriority` and bypasses the branch, commit, hourly-commit and pull-request limits for them unconditionally.

**Grouping rationale:** Without grouping, N separate action/hook updates create N branches that serialize: merge one → rebase others → CI reruns → repeat. Grouping collapses N updates into one branch, one CI run, one merge.

**Where to append.** Rules 9-11 only work because they sit AFTER rule 8 — a new rule appended at the end therefore lands after them and will override a commit type it did not mean to touch. A new automerge or stability rule belongs at the end of block 1-7, not at the end of the file.

Child preset rules are appended after all of these and can safely add more specific rules.

## Development Commands

```bash
# JSON syntax check
jq empty default.json

# Run pre-commit hooks (MANDATORY before commit)
pre-commit run --all-files
```

## Important Constraints

### Deployment Model
- **CRITICAL:** Changes to `default.json` are **live immediately** when merged to `main`
- Changes here affect ALL Polarion GitHub repositories via the child presets
- No CI/CD pipeline, no staging environment — test carefully before pushing

### Semantic Commits
- Format: `type(scope): description`
- Examples:
  - `fix: correct automerge rule ordering for major updates`
  - `feat: add prPriority for security vulnerability alerts`
