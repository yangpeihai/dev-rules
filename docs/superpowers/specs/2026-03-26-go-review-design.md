---
title: go-review skill design
date: 2026-03-26
status: draft-approved
---

# go-review Skill Design

## Overview

`go-review` is a repository skill for reviewing Go code quality at module scope. It must detect syntax and build issues, run static analysis with graceful degradation, evaluate team Go conventions, and produce a non-overwriting report for each review run.

The skill is intended for day-to-day review and pre-merge validation. Its default behavior focuses on Git-changed Go files, then expands to affected packages. It must also support a full-module scan when explicitly requested.

## Goals

- Review Go changes with a stable, repeatable flow
- Prefer `golangci-lint` when available and degrade to built-in Go tooling when it is not
- Cover syntax, compile/test, static analysis, and team rule checks
- Generate both terminal summary output and a persisted Markdown report
- Prevent report overwrites across repeated review runs

## Non-Goals

- Replacing all human code review judgment with fully automated checks
- Scanning the entire repository by default
- Auto-fixing code during review
- Enforcing every team rule via hard-coded static checks in v1

## Scope

### Review scope

- Default scan mode: `changed`
- Optional scan mode: `full`
- Default target selection:
  - detect Git-changed `.go` files
  - map changed files to affected packages or module areas
  - run package-level checks where file-level checks would miss real issues

### Review dimensions

`go-review` covers:

1. Syntax and formatting validation
2. Build and test validation
3. Static analysis validation
4. Team Go rule review
5. Structured reporting and final verdict generation

## User-Facing Behavior

### Inputs

The skill must support these execution inputs:

- target module path
- scan mode: `changed` or `full`
- optional explicit report output directory

If the caller does not specify a scan mode, the skill uses `changed`.

### Outputs

Every run produces:

- terminal summary
- one Markdown report persisted to disk

The terminal summary includes:

- module path
- scan mode
- tools used
- degraded-mode note when applicable
- issue counts by severity
- final verdict
- report file path

## Report Design

### Output directory

Reports are written under:

```text
reports/go-review/
```

If the caller provides a custom output directory, the same naming rules still apply.

### Unique report naming

To prevent overwrite across multiple review runs, each report filename must include:

- module identifier
- scan mode
- execution timestamp
- Git short SHA when available, otherwise `nogit`

Filename pattern:

```text
<module>-<mode>-<timestamp>-<gitsha>.md
```

Example:

```text
reports/go-review/user-service-changed-20260326-143512-a1b2c3d.md
```

### Report sections

The Markdown report must contain these sections:

1. `Scan Context`
2. `Findings Summary`
3. `Syntax And Build Checks`
4. `Static Analysis`
5. `Team Rule Findings`
6. `Final Verdict`

Each finding should include, where available:

- severity: `blocking`, `warning`, or `info`
- rule or tool source
- affected file or package
- concise explanation
- suggested next action

## Execution Flow

### Step 1: Resolve review scope

For `changed` mode:

- inspect Git changes for `.go` files
- if no Go file changes exist, generate a report stating that no Go changes were detected
- expand changed files into affected packages for package-level checks

For `full` mode:

- scan all Go files under the specified module scope
- run package-level checks for the full module

### Step 2: Environment preflight

The automation layer must detect:

- whether `go` is installed
- whether the target path belongs to a Go module or can resolve a relevant `go.mod`
- whether `golangci-lint` is available

Failure handling:

- if `go` is unavailable, stop with `fail`
- if `golangci-lint` is unavailable, continue in degraded mode and record that fact in terminal and report outputs

### Step 3: Deterministic checks

Recommended order:

1. `gofmt -l`
2. `go test ./...` for affected packages or module scope
3. `go vet ./...` for affected packages or module scope
4. `golangci-lint run` when available

Execution policy:

- file-level formatting checks run on changed files where possible
- package-level validation is used for test, build, vet, and lint checks because many Go issues surface only at package scope

### Step 4: Team rule review

The skill supplements tool output with structured review against team Go rules. v1 covers:

1. formatting non-compliance
2. compile or test failures
3. static analysis findings
4. improper error handling
5. missing `context` or timeout for external calls
6. missing comments on exported identifiers
7. sensitive data or unsafe logging risk
8. SQL or command execution risk
9. package or file responsibility overload
10. unjustified `panic` usage

Rule classification:

- deterministic findings should come directly from tooling or script heuristics
- judgment-heavy findings should be emitted under a manual-review subsection with explicit wording that human review confirmed them

### Step 5: Verdict generation

Verdicts:

- `pass`: no blocking findings
- `pass_with_warnings`: warnings or info only
- `fail`: syntax, build, test, vet, lint, or severe rule violations exist

## Skill Structure

The first implementation should use this layout:

```text
skills/go-review/
  SKILL.md
  references/
    review-rules.md
    report-format.md
  scripts/
    go_review.py
```

### Responsibilities

`SKILL.md`

- explains when to use the skill
- defines default scan mode and fallback behavior
- instructs how to interpret output and verdicts
- tells the agent to persist the report and surface the path

`references/review-rules.md`

- lists the team review rules
- separates durable rule content from the main skill body

`references/report-format.md`

- defines section structure, naming rules, severity conventions, and examples

`scripts/go_review.py`

- detects changed Go files
- resolves scan mode
- probes required tools
- runs checks
- aggregates results
- writes the uniquely named report
- prints a concise terminal summary

## Error Handling

- Missing Git metadata does not block review; report uses `nogit`
- No changed Go files in `changed` mode does not block review; report should clearly mark the run as no-op
- Missing `golangci-lint` does not block review; report must say degraded mode was used
- Missing Go toolchain blocks review
- Invalid module path blocks review with a clear diagnostic in terminal and report output where possible

## Testing Strategy

Because this is a skill, implementation must follow skill-authoring validation discipline rather than only unit tests.

### Skill validation requirements

- create baseline pressure scenarios before finalizing the skill text
- confirm that, without the skill, an agent would likely produce inconsistent review output or omit degraded-mode/report-path handling
- verify that, with the skill, the agent follows the defined review flow and report rules

### Script verification requirements

- verify changed-mode file detection
- verify full-mode scanning
- verify degraded behavior when `golangci-lint` is unavailable
- verify unique report naming across repeated runs
- verify terminal summary includes the saved report path

## Open Decisions Resolved

- default scan mode is `changed`
- full-module scan remains available as an explicit mode
- reports must be persisted and uniquely named
- review scope is module-oriented, not whole-repository by default
- `golangci-lint` is preferred but not mandatory for a usable review result

## Implementation Notes

- keep v1 focused on a stable workflow and report format
- avoid over-claiming fully automated rule understanding for semantic issues
- prefer clear degraded-mode messaging over silent fallback
- preserve compatibility with existing repository skill conventions
