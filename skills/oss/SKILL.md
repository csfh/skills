---
name: oss
description: >
  Audits a repository against an Echo-shaped open-source setup
  (https://github.com/modoterra/echo): OSI LICENSE, README, CONTRIBUTING,
  assignment CLA, SECURITY.md, GitHub issue/PR templates, CODEOWNERS, PR CI,
  community policy, and GitHub repo metadata. Reports gaps and scaffolds the
  missing files when asked. Use when checking OSS readiness, community health,
  LICENSE, CLA, CONTRIBUTING, SECURITY.md, GitHub templates, or when the user
  runs /oss.
compatibility: Designed for Agent Skills-compatible coding agents. Requires repository tools; GitHub API (`gh`) when the remote is GitHub.
metadata:
  author: Christoffer Hallas
  version: "1.0.0"
---

# OSS

Audit (and optionally scaffold) a repository so it is a complete open-source
project in the same **shape** as [modoterra/echo](https://github.com/modoterra/echo).

Echo is the reference for **legal, community, GitHub, and contributor** setup.
Do not copy Echo's language, crates, LLVM CI, `echo26` suite, `www/` site, or
product docs.

## Modes

| Mode | When |
|------|------|
| **audit** (default) | `/oss`, "is this OSS-ready", community-health check |
| **scaffold** | User asks to fix, add, or bring the project up to the bar |

Audit first even in scaffold mode. Do not overwrite existing policy files without
showing the diff and getting agreement when they already have real content.

## Identity

Resolve once; reuse everywhere (LICENSE, CLA, SECURITY, README, templates).

| Field | Discovery |
|-------|-----------|
| **Project name** | README `#` title, else repo name |
| **One-line description** | GitHub `description`, else README first paragraph |
| **Copyright holder / CLA assignee** | Existing `LICENSE`; else GitHub org/owner. `csfh` or `modoterra` → **Christoffer Hallas** |
| **License** | Existing OSI license if GitHub-detectable; else **MIT** (Echo default). Do not change an existing OSI license unless asked |
| **Security contact** | Existing `SECURITY.md`. `csfh` or `modoterra` → `oss@christofferhallas.com` |
| **Community contact** | README Community section. `csfh` or `modoterra` → `oss@christofferhallas.com` |
| **CODEOWNERS** | Existing file. Else ask. Do not invent handles |

Ask for any field you cannot resolve. Do not guess emails or owners.

Year in copyright lines: current calendar year, unless `LICENSE` already has a year range that should be extended.

## Workflow

### 1. Orient

- Repo root, default branch, `git remote -v`
- Language / package manifests (`Cargo.toml`, `package.json`, `pyproject.toml`, `go.mod`, `composer.json`, …)
- Existing policy files (list below)
- Whether the user wants audit or scaffold

### 2. Probe GitHub (when the remote is GitHub)

```bash
gh repo view --json name,nameWithOwner,description,homepageUrl,url,visibility,isPrivate,licenseInfo,repositoryTopics,hasIssuesEnabled,hasWikiEnabled,hasDiscussionsEnabled,hasProjectsEnabled,isSecurityPolicyEnabled,securityPolicyUrl,issueTemplates,pullRequestTemplates,contactLinks,codeOfConduct,defaultBranchRef
gh api "repos/{owner}/{repo}" --jq '{web_commit_signoff_required,has_pages}'
gh api "repos/{owner}/{repo}/community/profile"
```

If `gh` is missing or unauthenticated, score GitHub-only checks as **unverified** and continue with the tree.

### 3. Score the bar

For each check: **pass**, **fail**, **warn**, **n/a**, or **unverified**. Cite the path or API field.

A file that exists but is a stub, placeholder, or Echo-specific leftover copied into the wrong project is **fail**.

### 4. Report

Use the [report format](#report-format). In audit mode, stop after the report unless the user asks to scaffold.

### 5. Scaffold (only if asked)

1. Present the fail/warn list and which files you will add or edit.
2. Fetch Echo **policy** files as templates (see [Scaffold](#scaffold)).
3. Substitute identity; strip Echo product specifics.
4. Wire README sections and package-manager license metadata.
5. Re-score and show the new report.

## The bar

Required unless marked recommended. **n/a** only when the check cannot apply (no GitHub remote, no package manifest). **unverified** when GitHub must be read and `gh` cannot.

### Legal

| Check | Pass when |
|-------|-----------|
| `LICENSE` at repo root | Filename is `LICENSE` (no extension). GitHub license API returns an SPDX id. Default text is the MIT License with `Copyright (c) <year> <holder>` |
| Package license metadata | Manifest SPDX matches `LICENSE` (`workspace.package.license` / `package.license` in Cargo, `license` in `package.json` / `composer.json`, `[project] license` in `pyproject.toml`, …) |
| Public source | GitHub `visibility` is public. A private repo is **fail** for OSS readiness |

Keep an existing OSI-approved license. Flag non-OSI, missing, or undetectable licenses as **fail**.

### README (`README.md`)

Must include, in this spirit (Echo's README is the outline):

1. What the project is (opening paragraph; matches GitHub description when one exists)
2. Honest status
3. How to build, install, or use (commands that work in *this* tree)
4. Pointers to docs / contributing / security
5. **Contributing:** CLA by submission, links to `CONTRIBUTING.md` and `CLA.md`
6. **Community:** Echo's policy, with this project's contact (see below)
7. **License:** SPDX name + link to `LICENSE` + copyright holder

Community section (required wording, contacts substituted):

```markdown
## Community

Use common sense and decency. There is no formal code of conduct. We reserve
the right to moderate this community to the extent of the law and the policy
of the host. Write <community-email> if you need us.
```

Do **not** add `CODE_OF_CONDUCT.md` or Contributor Covenant. If one exists, **warn** (house-style mismatch). Delete it only when asked.

### Contribution

| File | Pass when |
|------|-----------|
| `CONTRIBUTING.md` | License + CLA (accept by submission, IP assignment to the copyright holder), setup, focused PRs, tests/docs, issue vs security reporting |
| `CLA.md` | Assignment CLA in Echo's structure: accept-by-contribution (no signature bot required), copyright + necessary patent rights to the assignee, moral-rights waiver, license-back, representations, third-party submissions, Delaware / assignee jurisdiction unless the holder is elsewhere and the user names it |
| `AGENTS.md` | **Recommended.** Workflow and where facts live for humans and coding agents. Skip only for tiny single-file repos |

### Security

| File | Pass when |
|------|-----------|
| `SECURITY.md` | Supported-version policy; **do not** open public issues for vulns; private email; coordinated disclosure; non-security bugs go to GitHub issues |

### GitHub community files

| Path | Pass when |
|------|-----------|
| `.github/CODEOWNERS` | At least a default owner (`* @org-or-user`). Valid handles |
| `.github/ISSUE_TEMPLATE/bug_report.md` | Summary, expected, actual, reproduction, environment |
| `.github/ISSUE_TEMPLATE/feature_request.md` | Problem, proposal, alternatives, scope |
| `.github/ISSUE_TEMPLATE/config.yml` | `contact_links` entry pointing at `SECURITY.md` / the security email. `blank_issues_enabled` may stay true |
| `.github/pull_request_template.md` | Summary + checklist; states that submitting the PR accepts `CLA.md` |

Issue templates may be YAML forms instead of Markdown if they collect the same fields.

### Automation and hygiene

| Check | Pass when |
|-------|-----------|
| PR CI | A workflow under `.github/workflows/` runs on `pull_request` and actually builds/tests this project. Release-only workflows do not satisfy this |
| `.gitignore` | Ignores secrets (`.env`, keys/pems) and this stack's build artifacts |
| Examples | **Recommended** when the project has a user-runnable CLI, library, or API: a small `examples/` (or documented equivalent) that still runs |
| `docs/` | **Recommended** when the README cannot hold the durable facts. If `docs/` exists, it needs a map (`docs/README.md` or a README table) |

### GitHub repository settings

| Setting | Pass when |
|---------|-----------|
| `description` | Set; matches the README opening |
| Topics | At least a few accurate topics |
| `homepage` | Set if the project has a public site; otherwise empty is fine |
| Issues enabled | `hasIssuesEnabled` true |
| Wiki | Off unless the project actually uses the GitHub wiki |
| License API | `licenseInfo.spdxId` (or `license.spdx_id`) is set |
| Web commit sign-off | **Recommended:** `web_commit_signoff_required` true (Echo). Report; set with user approval via `gh` |

Community profile `health_percentage` may stay under 100 because there is no code of conduct. Do not chase GitHub's CoC checkbox.

## Out of bar

Do not require, and do not copy from Echo:

- Language/product internals (`crates/`, `echo26/`, LLVM, `justfile`, `www/`, install tarballs)
- `CODE_OF_CONDUCT.md`, `FUNDING.yml`, `CITATION.cff`, `GOVERNANCE.md`, `SUPPORT.md`, `NOTICE`
- `brand/` assets (optional if the project has a mark)
- Discussions, Projects, a published GitHub Release (require a release process only if they already ship binaries)

## Scaffold

When creating policy files, fetch Echo's current versions and **substitute**, do not rewrite legal terms from memory:

| Echo source | Use for |
|-------------|---------|
| `LICENSE` | MIT text; replace copyright year and holder |
| `CLA.md` | Whole agreement; replace project name ("Echo" → this project), keep assignee = copyright holder; adjust email/product sentences that name Echo |
| `SECURITY.md` | Structure; replace project name, default branch, security email |
| `CONTRIBUTING.md` | Structure; replace setup/test commands with **this** repo's real commands; keep CLA-by-submission |
| `.github/ISSUE_TEMPLATE/*` | Structure; generic "version / commit", drop `xo` / Echo 2026 |
| `.github/pull_request_template.md` | Structure; CLA line + this repo's real checks |

URLs:

```
https://raw.githubusercontent.com/modoterra/echo/main/LICENSE
https://raw.githubusercontent.com/modoterra/echo/main/CLA.md
https://raw.githubusercontent.com/modoterra/echo/main/SECURITY.md
https://raw.githubusercontent.com/modoterra/echo/main/CONTRIBUTING.md
https://raw.githubusercontent.com/modoterra/echo/main/.github/ISSUE_TEMPLATE/config.yml
https://raw.githubusercontent.com/modoterra/echo/main/.github/ISSUE_TEMPLATE/bug_report.md
https://raw.githubusercontent.com/modoterra/echo/main/.github/ISSUE_TEMPLATE/feature_request.md
https://raw.githubusercontent.com/modoterra/echo/main/.github/pull_request_template.md
```

If the fetch fails, clone or vendor from a local Echo checkout. Do not invent a different CLA.

README edits: add missing **Contributing**, **Community**, and **License** sections; do not replace a good project-specific README with Echo's.

Package metadata: set license to the SPDX of `LICENSE` in the manifest this repo already uses. Do not add a new ecosystem.

GitHub settings (scaffold, with approval):

```bash
gh repo edit --description "..." --homepage "..." --add-topic "..." --enable-issues --disable-wiki
# optional, Echo default:
gh api -X PATCH "repos/{owner}/{repo}" -f web_commit_signoff_required=true
```

Do not `gh repo edit --visibility public` unless the user explicitly asks to publish.

CI: add a minimal `pull_request` workflow that runs this project's existing test/build command. Do not paste Echo's LLVM/release matrix.

## Report format

```markdown
## OSS report

**Project:** <name>  **Remote:** <url or local>
**License:** <SPDX or missing>  **Visibility:** public | private | unknown
**Overall:** ready | incomplete

| Check | Status | Evidence |
|-------|--------|----------|
| LICENSE | pass/fail/warn/n/a/unverified | path or API |
| … | | |

### Blocking
- …

### Recommended next
- …

### Notes
- …
```

**Overall ready** only when every required check is **pass** or **n/a**. **unverified** required checks make the overall **incomplete**. Failed recommended checks stay under Recommended next and do not block ready.

## Constraints

- Echo supplies **shape**. This project's commands, architecture, and docs stay this project's.
- Do not change license SPDX, copyright holder, or CLA assignee without agreement.
- Do not commit secrets, generated `node_modules` / `target`, or Echo product files.
- Do not claim GitHub settings you did not read from the API or `gh`.
- Legal files (`LICENSE`, `CLA.md`) keep Echo's legal wording after substitution. Surrounding README/CONTRIBUTING prose follows this repo's voice.
