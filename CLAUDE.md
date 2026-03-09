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
- Automerge `packageRules`: non-major → major block → github-actions → pre-commit → security priority
- `osvVulnerabilityAlerts` + `vulnerabilityAlerts`

**Child presets add only:**
- `enabledManagers` (ecosystem-specific)
- `constraints` (python version, where applicable)
- Ecosystem-specific `packageRules` (Maven registry, Python version lock, SBB packages, etc.)

## packageRules Ordering (Critical)

Rules are applied in order — **later rules override earlier ones**. The base preset order is intentional:

1. Automerge non-major (minor/patch/pin/digest)
2. Block major updates (`automerge: false`)
3. Override: github-actions — automerge all including major
4. Override: pre-commit — automerge all including major (`ignoreTests: true`)
5. Security priority (`prPriority: 99`)

Child preset rules are appended after these and can safely add more specific rules.

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
