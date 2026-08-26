---
name: jfrog-oss-compliance
description: Audit a JFrog OSS repository for compliance with the JFrog OSS Policy (July 2025). Use when the user asks to check, review, or audit a repo for OSS policy compliance, OSSGOV review, license/copyright/CLA hygiene, or asks "is this repo compliant". Input is a GitHub URL or a local filesystem path.
---

# JFrog OSS Policy Compliance Check

Audits a JFrog-owned repository against the JFrog OSS Policy (July 2025) and produces a pass/fail/warning report plus a developer TODO list.

## Authoritative source

A structured summary of the policy ships with this skill at `./policy-summary.md` (relative to this SKILL.md). Read it for clause-level detail when in doubt. Key sections: §2 (Consumption), §3 (Contribution), §6 (license classification = Appendix A), §7-8 (copyright + license markings = Appendix B/C).

The canonical policy PDF is owned by JFrog P&E/Legal/CTO. If `policy-summary.md` is stale relative to a newer PDF, refresh it — the skill should not depend on any path outside its own folder.

## Inputs

User provides one of:
- **Local path** (preferred): `/path/to/repo` — analyze directly with `find`/`grep`/`Read`.
- **GitHub URL**: `https://github.com/jfrog/<repo>` — use `gh` CLI (`gh repo view`, `gh api repos/.../contents/...`). If the repo is private and `gh` returns 404, ask the user to clone locally and provide the path.

## Checks (in order)

| # | Check | Policy ref | Pass criteria |
|---|-------|------------|---------------|
| 1 | Owned by JFrog org | §4.4 | Repo is under `github.com/jfrog/*` (or `github.jfrog.info/jfrog/*` for internal) |
| 2 | LICENSE file = Apache-2.0 | App. C | Root `LICENSE` contains canonical Apache-2.0 text |
| 3 | NOTICE file present | App. B/C | Root `NOTICE` or `NOTICE.txt` exists with primary JFrog copyright + 3rd-party attributions (license + homepage + copyright per dep) |
| 4 | Copyright header on **every** source file | App. C | Every `*.py`, `*.ts`, `*.js`, `*.go`, `*.java`, `*.sh` etc. starts with `(c) JFrog Ltd.` or `© JFrog Ltd.` |
| 5 | README.md | §4.3 | Present and non-trivial (purpose, install, usage) |
| 6 | CONTRIBUTING.md | §4.3 | Present; references JFrog CLA at `https://jfrog.com/cla/` |
| 7 | JFrog CLA workflow | §4.4 | `.github/workflows/cla.yml` exists and uses `jfrog/.github/actions/cla@main` (or equivalent older `cla-assistant/github-action` pointing at `https://jfrog.com/cla/`) |
| 8 | License declared in package metadata | App. C | `pyproject.toml`, `package.json`, `go.mod` or equivalent declares `Apache-2.0` |
| 9 | Secret hygiene | §4.4 | `.env*` in `.gitignore`; no obvious hardcoded `api_key`/`token`/`password`/`secret` in source |
| 10 | CI/CD present | §4.3 (maintenance) | At least one workflow under `.github/workflows/` (build/lint/test/release) |
| 11 | Project health | §3.2 | Recent commits, ≥1 release tag, contributors, open issues being addressed (informational) |
| 12 | Dependency licenses acceptable | App. A | Direct deps fall under Acceptable bucket (Unencumbered / Notice / Reciprocal). Flag any in Restricted/Restricted_if_statically_linked. **Mark as WARNING** unless a JFrog Xray SBOM has been submitted — that's the policy's authoritative process check |
| 13 | Quarterly SBOM submission | FAQ | Cannot be verified from the repo. **Always WARNING** with note: "Confirm SBOM via JFrog Xray has been emailed to `ossgov@jfrog.com` this quarter." |

## How to run the checks (local path)

```bash
# 1. License & top-level files
ls -la <repo>
head -5 <repo>/LICENSE                    # check Apache-2.0
cat <repo>/NOTICE 2>/dev/null              # may be NOTICE or NOTICE.txt
cat <repo>/CONTRIBUTING.md
cat <repo>/.github/workflows/cla.yml

# 2. Copyright header audit (adjust extensions per stack)
for f in $(find <repo>/src <repo>/tests -type f \( -name "*.py" -o -name "*.ts" -o -name "*.js" -o -name "*.go" -o -name "*.java" -o -name "*.sh" \)); do
  head -3 "$f" | grep -q "JFrog Ltd" || echo "MISSING: $f"
done

# 3. Package metadata
grep -E "license" <repo>/package.json <repo>/pyproject.toml 2>/dev/null

# 4. Secret scan
grep -rIE "(api[_-]?key|secret|token|password)\s*=\s*['\"]" <repo>/src 2>/dev/null

# 5. Git activity
cd <repo> && git log --oneline -10 && git remote -v
```

## How to run the checks (GitHub URL via gh)

```bash
gh repo view <owner>/<repo> --json licenseInfo,defaultBranchRef,pushedAt,stargazerCount,forkCount
gh api repos/<owner>/<repo>/contents/ --jq '.[].name'
gh api "repos/<owner>/<repo>/git/trees/HEAD?recursive=1" --jq '.tree[].path'
gh api repos/<owner>/<repo>/contents/LICENSE --jq '.content' | base64 -d | head
gh api repos/<owner>/<repo>/contents/NOTICE --jq '.content' | base64 -d 2>/dev/null
gh api "repos/<owner>/<repo>/commits?per_page=10" --jq '.[].commit.message'
```

If 404 on private repo, ask user to clone and provide local path.

## Output format

Produce a markdown report with this exact shape:

```
## OSS Policy Compliance Report
**Repo:** <owner>/<name>
**Date:** <today>
**Policy:** JFrog OSS Policy (July 2025)

### Summary: <N> PASS / <N> FAIL / <N> WARNING

| # | Check | Status | Details |
|---|-------|--------|---------|
| 1 | ... | PASS/FAIL/WARNING | <one-line evidence> |

---

### TODO for Repo Owner

**Must fix:** <FAIL items as numbered actionable list with file paths>
**Recommended:** <WARNING items>

---

Verdict: <one-sentence overall>
```

## Conventions

- Default 13 checks above. Skip a check (don't fake a PASS) if not applicable to the stack — note "N/A" with reason.
- For copyright header check: report `<missing>/<total>` count, list up to 5 example missing files, don't dump the full list.
- For dependency license check: list direct runtime deps (not dev deps) and their licenses. Don't try to enumerate transitives — defer to Xray SBOM.
- Always include the SBOM warning (#13) — it's a process check the repo can't satisfy alone.
- When proposing fixes, give the exact header/snippet to add (e.g., `// (c) JFrog Ltd. (2026)` for TS, `# (c) JFrog Ltd. (2026)` for Python/shell).

## Reference repos (gold-standard examples)

- `jfrog/agenteval` — strong example: full copyright coverage, NOTICE.txt with license+homepage+copyright per dep, canonical CLA workflow.
- `jfrog/JFrog-AzureML-integration` — earlier audit; was missing copyright headers and CONTRIBUTING.md.
- `jfrog/opencode-jfrog-plugin` — near-compliant; was missing copyright headers and `license` field in `package.json`.
