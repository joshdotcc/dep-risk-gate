# Dependency Risk Gate (Go)

A lightweight Go-based tool and GitHub Action that evaluates dependency changes in pull requests and blocks risky changes. Focused on change-based risk detection, policy-driven decisions, and CI-native workflows.

---

## Getting Started

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/dep-risk-gate.git
   cd dep-risk-gate
   ```
2. Copy and configure environment variables:
   ```bash
   cp example.envrc .envrc
   # Edit .envrc with your API keys
   direnv allow
   ```
3. Run CLI locally:
   ```bash
   dep-risk-gate --base-sha origin/main --head-sha HEAD --policy policy.yaml
   ```
4. Integrate with GitHub Actions (see `.github/actions/dep-risk-gate/action.yml`).

---

## Project Overview

Dependency Risk Gate is a Go-based CLI and GitHub Action that evaluates *risk introduced by dependency changes in a pull request* and decides whether the PR should pass, warn, or fail.

It is intentionally **not** a full SCA scanner. It is a **decision layer** that answers one question:
> *Did this PR make dependency risk worse?*

---

## Design Philosophy

### Core Principles
- **Change-focused**: Only evaluate dependencies that changed in the PR
- **Low-noise**: No warnings for pre-existing issues
- **Fast & cheap**: Designed to stay under GitHub Actions free quota
- **Policy-driven**: Clear, explainable decisions
- **CI-native**: Built for PR workflows

### Non-Goals
- Full repository vulnerability inventory
- Competing with Snyk/Dependabot/etc.
- Scanning unchanged dependencies

---

## High-Level Flow

```text
Pull Request Opened
        ↓
Detect dependency file diffs
        ↓
Identify changed dependencies (before vs after)
        ↓
Fetch vulnerability & metadata via APIs
        ↓
Compute risk delta
        ↓
Evaluate policy
        ↓
Pass / Warn / Fail
```

---

## Risk Model

Each dependency version resolves to a simple risk profile:
- Number of vulnerabilities by severity
- Presence of known exploits (if available)
- Metadata (last published date, maintenance signals)

Example:
```text
Dependency: github.com/example/lib@v1.2.0
- Critical: 1
- High: 2
- Exploit available: true
- Last updated: 18 months ago
```

---

## Risk Delta Concept

The gate evaluates **risk difference**, not absolute risk.

| Scenario | Result |
|--------|--------|
| No dependency change | Pass silently |
| Upgrade removes vulns | Pass + optional positive comment |
| Upgrade introduces high vuln | Warn |
| New dependency w/ critical vuln | Fail |
| Downgrade introduces exploit | Fail |

---

## Policy Engine

Example Policy (YAML):
```yaml
fail_on:
  critical_vulns: true
  exploit_available: true
warn_on:
  high_vulns: true
```

Evaluation Order:
1. Did risk increase?
2. Does increase violate `fail_on` rules?
3. Else does it violate `warn_on` rules?
4. Else pass

---

## Supported Ecosystems
- v1: Go (`go.mod`, `go.sum`)
- Future: npm (`package.json`, `package-lock.json`), Python (`requirements.txt`)

---

## External APIs
- **OSV.dev**: Vulnerability data (free, JSON)
- Optional future APIs: deps.dev, GitHub Advisory API, NVD
- API keys via environment variables

---

## GitHub Action Usage

Trigger on `pull_request`:
```yaml
- uses: ./github/actions/dep-risk-gate
  env:
    OSV_API_KEY: ${{ secrets.OSV_API_KEY }}
```

Exit codes:
- `0` = pass
- `1` = fail

---

## Repository Structure

```text
dep-risk-gate/
├── cmd/
│   └── dep-risk-gate/
│       └── main.go
├── internal/
│   ├── diff/
│   ├── parser/
│   ├── api/
│   ├── risk/
│   ├── policy/
│   └── report/
├── .github/actions/dep-risk-gate/action.yml
├── policy.yaml.example
├── example.envrc
├── .envrc (gitignored)
├── .gitignore
└── README.md
```

---

## Example PR Comments

**Fail:**
```text
❌ Dependency Risk Gate: FAILED
New dependency introduces increased risk:
- github.com/example/oldlib@v1.2.0
  - Critical vulnerabilities: 1
  - Exploit available: yes
Policy: fail_on_critical = true
```

**Warn:**
```text
⚠️ Dependency Risk Gate: WARNING
Dependency upgrade introduces high-severity vulnerabilities:
- github.com/foo/bar v1.1.0 → v1.2.0
```

---

## Environment Variable Strategy

```bash
export OSV_API_KEY=changeme
export FAIL_ON_CRITICAL=true
```
- `.envrc` ignored
- `example.envrc` committed
- README explains `direnv`

---

## Why This Project Signals Senior AppSec
- Change-based risk modeling
- CI-native decisioning
- Thoughtful noise reduction
- Clear policy abstraction
- Realistic constraints

---

## Future Enhancements
- Risk scoring instead of binary rules
- Baseline snapshots
- Multiple policies per repo
- SARIF output
- JSON output for pipelines

---

## v1 Success Criteria
- PR with no dependency changes passes silently
- PR introducing critical vuln fails
- Decisions are explainable in one PR comment
- Action runtime stays under free GitHub quota

