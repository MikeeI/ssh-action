# ISSUE-002 — entrypoint: static output delimiter can truncate captured stdout

State: Published
Mode: Pull request
Target: New pull request
Location: https://github.com/appleboy/ssh-action/pull/415
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
- [S] GitHub's multiline environment-file contract warns that a delimiter must not occur on its own line in the value.
- [S] `actions/runner@99e01149303b194e08778cdc9c03fa2703d03fb2/FileCommandManager.cs` — A line equal to the delimiter closes the value with ordinal equality.
- [O] Controlled base capture of `vor\nEOF\nnach\n` → Output file contains a first standalone `EOF` before `nach`; environment=Ubuntu 24.04, Bash 5.2.21.
- [O] Controlled branch capture of the same value → One randomized framing pair surrounds all three ordered lines.

## Prior art

Coverage: Issues, pull requests, discussions, releases, and delimiter history searched; checked=2026-08-10.
Gaps: Repository-scoped discussion search found no target; no material prior-art gap remains.

- `https://github.com/appleboy/ssh-action/pull/287` — Related; introduced static `EOF` capture framing.
- `https://github.com/appleboy/ssh-action/issues/375` — Distinct; owns failed-pipeline framing, not delimiter collision.
- `https://github.com/appleboy/ssh-action/issues/397` — Fixed; duplicate-output root cause is separate.
- `https://github.com/appleboy/ssh-action/pull/403` — Distinct; closed proposal changed output-stream ownership.
- `https://github.com/appleboy/ssh-action/pull/404` — Related; retained static framing while fixing duplication.
- `https://github.com/appleboy/ssh-action/releases/tag/v1.2.1` — Related; released the initial captured output.
- `https://github.com/appleboy/ssh-action/releases/tag/v1.2.5` — Related; released the current direct-write framing.

Target fit: New pull request — no canonical collision thread or active implementation exists.

## Direction

Generate one randomized delimiter per invocation and use it for both framing lines while retaining the streaming `tee` path.

## Bounds

- Preserve: `pipefail`, binary exit propagation, live logs, multiline content, and the public `stdout` output.
- Exclude: Output buffering, a new output format, or changes to separately owned `drone-ssh` streams.
- Cost: A local framing change with nonzero residual delimiter-collision risk.

## Verification

- Capture remote lines `vor`, `EOF`, and `nach` → Preserves exact order and content.
- Repeat with a failing command → Preserves failure propagation.

## Missing

None.

## Resume

Index: Monitor pull request review
Next: Monitor pull request 415 and respond only with new evidence or requested bounded changes.
Done when: Upstream merges, closes, or requests a bounded change.

## API and compatibility

Callers [S]: Workflows consuming `steps.<id>.outputs.stdout` with `capture_stdout: true`.
Contract [S]: `action.yml` promises captured command stdout as one multiline string.
Compatibility: Preserve successful output bytes, ordering, live log visibility, and action failure status.
Migration: None.

## Implementation

Branch: `fix/capture-output-delimiter`
Base: `upstream/master@b838bc2f27cd449b957159452d432aff91697637`
Scope: Generate one randomized Bash-only delimiter per captured output invocation.
Commit: `d3b63444cdbf907b1f806e6a7b5d28236f224e55`
Push: `origin/fix/capture-output-delimiter`
Checks:

- `bash -n entrypoint.sh && shellcheck entrypoint.sh && git diff --check` → Passed.
- Controlled base capture writes `vor`, `EOF`, and `nach` → Static `EOF` closes before `nach`.
- Controlled branch capture writes the same lines → One randomized framing pair encloses all three lines.
- Controlled branch executable exits 17 → Entrypoint still exits 17 through the unchanged `pipefail` pipeline.

## Draft

Title:

```text
fix: randomize the captured output delimiter
```

Body:

```markdown
## Summary

Captured stdout currently uses the fixed multiline delimiter `EOF`, although a remote command may legitimately emit `EOF` on a line by itself.
This change creates one randomized Bash-only delimiter per capture and uses it for both framing lines while retaining the streaming `tee` path.

## Evidence

- [`entrypoint.sh` on the current base](https://github.com/appleboy/ssh-action/blob/b838bc2f27cd449b957159452d432aff91697637/entrypoint.sh#L73-L76) frames every captured value with the literal line `EOF`.
- [GitHub's multiline-string contract](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands#multiline-strings) requires that the delimiter not occur on a line by itself within the value.
- [The runner parser](https://github.com/actions/runner/blob/99e01149303b194e08778cdc9c03fa2703d03fb2/src/Runner.Worker/FileCommandManager.cs) closes a multiline value when a line equals its delimiter.
- A controlled capture of `vor\nEOF\nnach\n` produced static framing in which the first `EOF` closes the value before `nach`.

## Changes

- Generate one delimiter from eight underscore-separated Bash `$RANDOM` components for each capture invocation.
- Use the same quoted delimiter in the opening and closing `GITHUB_OUTPUT` lines.
- Keep the `drone-ssh | tee` pipeline, live logs, byte ordering, and public `stdout` output unchanged.

## Risks and boundaries

- `$RANDOM` is seedable pseudo-randomness, not cryptographic entropy; this removes the deterministic collision but not every theoretical or adversarial collision.
- GitHub warns that delimiter framing cannot safely represent completely arbitrary content without buffering; this pull request intentionally preserves streaming.
- Nonzero-pipeline cleanup, output without a final newline, and stderr ownership remain unchanged and outside this correction.

## Verification

- `bash -n entrypoint.sh && shellcheck entrypoint.sh && git diff --check` — passed.
- Controlled capture writes `vor`, `EOF`, and `nach` — one randomized framing pair encloses all three lines in order.
- Controlled executable exits 17 — the unchanged `pipefail` pipeline still returns 17.

I checked the relevant issues, comments, pull requests, discussions, releases, and capture history; this pull request is not a duplicate.

### Disclosure

Investigated thoroughly with GPT-5.6 Codex (high reasoning effort), using [Oh My Pi](https://github.com/can1357/oh-my-pi) as the agent framework.

This report is not generic or unreviewed AI-generated output. Its claims were checked against the cited evidence, and it includes the relevant detail intended to help maintainers resolve the issue.

If reports like this are not useful to the project, please let me know and I will refrain from submitting similar ones. My intent is to help without wasting maintainer time or energy or discouraging their work.

Thank you for your work.
```
