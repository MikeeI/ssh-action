# ISSUE-006 — entrypoint: failed capture leaves output record unterminated

State: Hold
Mode: Undecided
Target: Undecided
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

## Prior art

Coverage: Issue 375 read; broader issues, pull requests, discussions, releases, and output-handling history incomplete; checked=2026-08-10.
Gaps: Complete thread-adjacent research and focused reproduction against current upstream.

- `https://github.com/appleboy/ssh-action/issues/375` — Related; owns the exact failed-capture symptom and observed runner error.

Target fit: Issue 375 comment is the leading target if Report mode is selected and new reproduction or fix evidence materially advances it.

## Direction

Execute the pipeline as an `if` condition, capture its real failure status, close the output record on its own line, and return the saved status after successful framing.

## Bounds

- Preserve: `tee` failure handling, successful capture behavior, live logs, and original command failure status.
- Exclude: Fixed-delimiter collision, stderr capture, output-format replacement, and global `set +e` handling.
- Cost: One local failure-branch correction plus a focused shell-boundary check.

## Verification

- Controlled executable writes stdout without a final newline and exits 17 → Entrypoint exits 17 and leaves one complete parseable `stdout` record with live output.

## Missing

- Complete prior-art research around issue 375.
- Focused reproduction against the recorded source revision.

## Resume

Index: Reproduce failed capture
Next: Reproduce the nonzero capture path with controlled stdout and record the output file plus exit status.
Done when: Exit status, live output, and malformed current record are captured exactly.

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
