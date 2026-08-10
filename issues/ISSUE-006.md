# ISSUE-006 — entrypoint: failed capture leaves output record unterminated

State: Ready
Mode: Pull request
Target: New pull request
Location: Not published.
Priority: High
Confidence: High
Type: reliability
Created: 2026-08-10
Updated: 2026-08-10
Source: `upstream/master@b838bc2f27cd449b957159452d432aff91697637`

## Root

Root [O]: A nonzero `drone-ssh` pipeline exits the strict-mode entrypoint before it writes the closing `GITHUB_OUTPUT` delimiter.

## Reach and impact

Reach [S]: Every `capture_stdout: true` invocation whose binary process returns nonzero reaches the incomplete framing path.
Impact [O]: Failed captured commands add a secondary runner parser error and do not expose captured output to later failure handlers.

## Evidence

- [S] `entrypoint.sh:3,73-76` — Enables `pipefail`, opens `stdout<<EOF`, runs through `tee`, and closes only after pipeline success.
- [S] `.github/workflows/main.yml:823-835` — Exercises a failing remote script with capture enabled but does not assert the output record.
- [O] `https://github.com/appleboy/ssh-action/issues/375` — Observes `Invalid value. Matching delimiter not found 'EOF'` and unavailable failure output after a nonzero remote command.
- [S] `upstream/master@b838bc2f27cd449b957159452d432aff91697637` — Matches the fork for the affected action, entrypoint, and workflow.
- [O] Controlled cached binary, `stdout=payload` without a final LF, exit 17 on the base → Entrypoint exited 17 and wrote `stdout<<EOF\npayload` without a closing delimiter; environment=Ubuntu 24.04, Bash 5.2.21.
- [S] `actions/runner@99e01149303b194e08778cdc9c03fa2703d03fb2/FileCommandManager.cs` — The parser rejects multiline values without a matching delimiter and excludes the framing newline from the parsed value.

## Prior art

Coverage: Issues, pull requests, discussions, releases, and capture history searched; checked=2026-08-10.
Gaps: Discussion search returned no repository-scoped candidate; no material prior-art gap remains.

- `https://github.com/appleboy/ssh-action/issues/375` — Duplicate symptom; the open canonical issue has no active fix.
- `https://github.com/appleboy/ssh-action/pull/287` — Related; introduced `capture_stdout`, not failure-safe framing.
- `https://github.com/appleboy/ssh-action/pull/374` — Related; refactored the capture path before issue 375 was filed.
- `https://github.com/appleboy/ssh-action/pull/403` — Distinct; closed output-duplication proposal changed stream ownership.
- `https://github.com/appleboy/ssh-action/pull/404` — Related; merged direct-write implementation retained early exit.
- `https://github.com/appleboy/ssh-action/issues/397` — Fixed; duplicate-output root cause resolved by pull request 404.

Target fit: New pull request — the bounded verified fix advances issue 375, and no active implementation owns it.

## Direction

Execute the pipeline as an `if` condition, capture its real failure status, close the output record on its own line, and return the saved status after successful framing.

## Bounds

- Preserve: `tee` failure handling, successful capture behavior, live logs, and original command failure status.
- Exclude: Fixed-delimiter collision, stderr capture, output-format replacement, and global `set +e` handling.
- Cost: One local failure-branch correction plus a focused shell-boundary check.

## Verification

- Controlled executable writes stdout without a final newline and exits 17 → Entrypoint exits 17 and leaves one complete parseable `stdout` record with live output.

## Missing

None.

## Resume

Index: Approve pull request draft
Next: Review the exact target and draft, then approve or revise publication.
Done when: The user approves the exact current target and draft.

## Bug reproduction

Environment: External report using `appleboy/ssh-action@master` on GitHub Actions with `capture_stdout: true`.
Reproduction: Run a remote script that exits nonzero and read the failed step plus downstream output.
Actual [O]: The step reports the remote failure, then `Matching delimiter not found 'EOF'`; captured output is unavailable.
Expected: Preserve the remote failure while exposing a complete captured stdout value to failure-handling steps.

## API and compatibility

Callers [S]: Workflows using `capture_stdout: true` and downstream `if: failure()` handlers.
Contract [S]: The action output framing and command exit propagation form one capture lifecycle.
Compatibility: Output-file write failures remain boundary failures; fixed-delimiter collision remains owned by ISSUE-002.
Migration: None.

## Implementation

Branch: `fix/capture-failure-output`
Base: `upstream/master@b838bc2f27cd449b957159452d432aff91697637`
Scope: Close captured output records before returning a nonzero `drone-ssh | tee` pipeline status.
Commit: `f39ff007253f7a61459ecbd0878d6ab14a8ca84d`
Push: `origin/fix/capture-failure-output`
Checks:

- `bash -n entrypoint.sh && shellcheck entrypoint.sh && git diff --check` → Passed.
- Controlled base executable writes `payload` without LF and exits 17 → Exit 17; output record remains unterminated.
- Controlled branch executable writes `payload` without LF and exits 17 → Exit 17; one complete output record.
- Controlled branch executable writes newline-terminated or empty stdout and exits 0 → Existing parsed value shape preserved.
- Controlled branch executable writes stdout ending in NUL and exits 17 → NUL preserved; separator and delimiter remain standalone.

## Draft

Title:

```text
fix: close captured output before returning failures
```

Body:

```markdown
## Summary

With `capture_stdout: true`, a nonzero `drone-ssh | tee` pipeline exits under `set -euo pipefail` before the closing `GITHUB_OUTPUT` delimiter is written.
This change records the pipeline status, completes the output record, and returns that status afterward.

## Evidence

- [`entrypoint.sh` on the current base](https://github.com/appleboy/ssh-action/blob/b838bc2f27cd449b957159452d432aff91697637/entrypoint.sh#L73-L76) opens the multiline value and executes the pipeline before writing its closing delimiter.
- [Issue #375](https://github.com/appleboy/ssh-action/issues/375) reports `Matching delimiter not found 'EOF'` and unavailable captured output after a nonzero remote command.
- A controlled executable that printed `payload` without a final newline and exited 17 reproduced exit 17 with the unterminated record `stdout<<EOF\npayload`.

## Changes

- Run the pipeline as an `if` condition so strict mode does not skip output-record cleanup.
- Preserve the pipeline's `pipefail` status and return it only after the closing delimiter is written.
- Add a separator only when stdout lacks a final newline, preserving successful newline-terminated and empty values.

## Risks and boundaries

- The delimiter remains the existing static `EOF`; delimiter collision is intentionally outside this pull request.
- Remote stderr ownership, output buffering, and command failure semantics remain unchanged.
- Output-file read or write failures still take precedence over the saved command status.

## Verification

- `bash -n entrypoint.sh && shellcheck entrypoint.sh && git diff --check` — passed.
- Controlled executable writes `payload` without a final newline and exits 17 — the entrypoint exits 17 and writes one complete `stdout` record.
- Controlled executable writes newline-terminated or empty stdout and exits 0 — the existing parsed value shape is preserved.
- Controlled executable writes stdout ending in NUL and exits 17 — the NUL is preserved and the output record remains complete.

I checked the relevant issues, comments, pull requests, discussions, and releases; this pull request is not a duplicate.

### Disclosure

Investigated thoroughly with GPT-5.6 Codex (high reasoning effort), using [Oh My Pi](https://github.com/can1357/oh-my-pi) as the agent framework.

This report is not generic or unreviewed AI-generated output. Its claims were checked against the cited evidence, and it includes the relevant detail intended to help maintainers resolve the issue.

If reports like this are not useful to the project, please let me know and I will refrain from submitting similar ones. My intent is to help without wasting maintainer time or energy or discouraging their work.

Thank you for your work.
```
