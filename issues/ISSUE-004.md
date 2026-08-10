# ISSUE-004 — action output: drone-ssh merges stderr and status into stdout

State: Hold
Mode: Undecided
Target: Undecided
Location: Not published.
Priority: High
Confidence: High
Type: API
Created: 2026-08-10
Updated: 2026-08-10
Source: `upstream/master@b838bc2f27cd449b957159452d432aff91697637`

## Root

Root [S]: The default `drone-ssh` binary writes remote stderr and final status text to the stdout stream captured by the action.

## Reach and impact

Reach [S]: Every `capture_stdout: true` invocation captures the binary's combined stdout-owned stream.
Impact [S]: Captured output contains data beyond remote stdout; [N] downstream parsing and exact-match impact are not measured.

## Evidence

- [S] `action.yml:81-91` and `README.md:128-134,384-403` — Describe the action output as standard output from executed commands.
- [S] `entrypoint.sh:73-76` — Captures the complete binary stdout stream into `GITHUB_OUTPUT`.
- [S] `drone-ssh v1.8.2/plugin.go` — Routes both `stdoutChan` and `stderrChan` through stdout-backed `p.log`.
- [S] `drone-ssh v1.8.2/plugin.go` — Prints the three-line success status with `fmt.Println`.
- [S] `upstream/master@b838bc2f27cd449b957159452d432aff91697637` — Matches the fork for `action.yml`, `entrypoint.sh`, and `README.md`.

## Prior art

Coverage: Not completed; checked=2026-08-10.
Gaps: Both repositories' issues, pull requests, discussions, releases, and stream-ownership history remain to be searched.

Target fit: Undecided — the corrective owner and any canonical existing thread require cross-repository research.

## Direction

Give `drone-ssh` separate stdout and error/status writers, release that contract, and update the action pin only after that release exists.

## Bounds

- Preserve: Remote stdout capture, visible stderr and status diagnostics, direct binary behavior unless explicitly migrated, and action failure semantics.
- Exclude: Parsing arbitrary remote text in `ssh-action` to strip status or error lines.
- Cost: Cross-repository implementation, compatibility review, release coordination, and action-pin adoption.

## Verification

- Run a remote command with distinct stdout and stderr markers → Action output contains only stdout while stderr and status remain visible in the step log.

## Missing

- Complete prior-art search in `appleboy/ssh-action` and `appleboy/drone-ssh`.
- Focused reproduction of the current mixed output.
- Evidence-backed corrective owner and contribution target.

## Resume

Index: Research stream ownership
Next: Search both repositories for stdout, stderr, status, and `capture_stdout` ownership discussions and implementations.
Done when: Every plausible prior-art candidate is classified and the corrective repository plus target fit are recorded.

## Shared change pressure

Copies [S]: Remote stdout, remote stderr, and final status currently share one stdout-backed writer.
Pressure [S]: These streams must retain distinct ownership for the action's standard-output contract to remain truthful.
Drift [S]: Current binary behavior and action documentation disagree about captured stream content.
Owner: `drone-ssh` stream writers, followed by `ssh-action` release-pin adoption.
Cost: Requires cross-repository release coordination; do not add an action-side text filter.

## API and compatibility

Callers [S]: Direct `drone-ssh` consumers and workflows consuming the action's `stdout` output.
Contract [S]: The action documents remote standard output while the default binary writes additional data to stdout.
Compatibility: Direct binary consumers may observe stream changes and require explicit release notes.
Migration: Update the action pin only after a compatible corrected binary release exists.
