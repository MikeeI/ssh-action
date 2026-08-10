# ISSUE-002 — entrypoint: static output delimiter can truncate captured stdout

State: Hold
Mode: Undecided
Target: Undecided
Location: Not published.
Priority: High
Confidence: High
Type: correctness
Created: 2026-08-10
Updated: 2026-08-10
Source: `upstream/master@b838bc2f27cd449b957159452d432aff91697637`

## Root

Root [S]: The `capture_stdout` serializer uses the fixed control line `EOF`, which is also valid arbitrary remote output.

## Reach and impact

Reach [S]: Every captured command whose stdout contains a standalone `EOF` line reaches the collision.
Impact [S]: The line terminates the value early and can expose later lines as invalid workflow-command data; [N] real-workflow occurrence is not measured.

## Evidence

- [S] `entrypoint.sh:73-76` — Uses static `EOF` framing around unmodified binary stdout.
- [S] `action.yml:88-91` — Exposes the captured value as the action's only output.
- [S] `upstream/master@b838bc2f27cd449b957159452d432aff91697637` — Matches the fork for `entrypoint.sh`.

## Prior art

Coverage: Not completed; checked=2026-08-10.
Gaps: Open and closed issues, pull requests, discussions, releases, and GitHub Actions framing history remain to be searched.

Target fit: Undecided — prior-art research and a focused collision reproduction are incomplete.

## Direction

Generate one high-entropy delimiter per invocation and use it for both framing lines while retaining the streaming `tee` path.

## Bounds

- Preserve: `pipefail`, binary exit propagation, live logs, multiline content, and the public `stdout` output.
- Exclude: Output buffering, a new output format, or changes to separately owned `drone-ssh` streams.
- Cost: A local framing change with negligible but nonzero theoretical delimiter-collision risk.

## Verification

- Capture remote lines `vor`, `EOF`, and `nach` → Preserves exact order and content.
- Repeat with a failing command → Preserves failure propagation.

## Missing

- Complete upstream prior-art search.
- Focused delimiter-collision reproduction against the recorded source revision.

## Resume

Index: Reproduce EOF collision
Next: Run a controlled capture whose output contains a standalone `EOF` line.
Done when: The current truncation and runner result are recorded with exact command and output.

## API and compatibility

Callers [S]: Workflows consuming `steps.<id>.outputs.stdout` with `capture_stdout: true`.
Contract [S]: `action.yml` promises captured command stdout as one multiline string.
Compatibility: Preserve successful output bytes, ordering, live log visibility, and action failure status.
Migration: None.
