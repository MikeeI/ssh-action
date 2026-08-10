# ISSUE-001 — action metadata: public inputs use ineffective runtime names

State: Ready
Mode: Pull request
Target: New pull request
Location: Not published.
Priority: High
Confidence: High
Type: correctness
Created: 2026-08-10
Updated: 2026-08-10
Source: `upstream/master@b838bc2f27cd449b957159452d432aff91697637`

## Root

Root [S]: The composite adapter omits `proxy_protocol` and maps three inputs to names that `drone-ssh v1.8.2` does not read.

## Reach and impact

Reach [S]: Calls using `allenvs`, either cipher input, or a non-default `proxy_protocol` traverse the mismatched adapter.
Impact [S]: Explicit values do not reach the default binary through its recognized names; [N] user-visible failure frequency is not measured.

## Evidence

- [S] `action.yml:132,135,137` — Exports `INPUT_ALL_ENVS`, `INPUT_CIPHER`, and `INPUT_PROXY_CIPHER`.
- [S] `action.yml:105-141` — Contains no `INPUT_PROXY_PROTOCOL` mapping.
- [S] `drone-ssh v1.8.2/main.go` — Reads `INPUT_ALLENVS`, `INPUT_CIPHERS`, `INPUT_PROXY_CIPHERS`, and `INPUT_PROXY_PROTOCOL`.
- [S] `upstream/master@b838bc2f27cd449b957159452d432aff91697637` — Matches the fork for the affected source.
- [S] `drone-ssh v1.8.2@9d94a36c84c95a692c27313a6726437ed344ae2a/main.go:105,175,236,276` — The pinned binary consumes `INPUT_CIPHERS`, `INPUT_PROXY_PROTOCOL`, `INPUT_PROXY_CIPHERS`, and `INPUT_ALLENVS`.
- [S] `yq v4.53.3` parsed the branch metadata and confirmed all four exact mappings while the three obsolete names are absent.

## Prior art

Coverage: Both repositories' issues, pull requests, discussions, releases, and naming history searched; checked=2026-08-10.
Gaps: Repository-scoped discussion searches found no target; no material prior-art gap remains.

- `https://github.com/appleboy/ssh-action/pull/303` — Related; introduced the composite adapter and the mismatches.
- `https://github.com/appleboy/ssh-action/pull/301` — Related; documented `allenvs` without repairing runtime mapping.
- `https://github.com/appleboy/ssh-action/issues/336` — Distinct; old protocol report corroborates `INPUT_ALLENVS`.
- `https://github.com/appleboy/drone-ssh/pull/252` — Related; standardized the consumed `INPUT_*` names.
- `https://github.com/appleboy/drone-ssh/pull/264` — Related; introduced `allenvs` with `INPUT_ALLENVS`.
- `https://github.com/appleboy/drone-ssh/commit/a83bebeafe7cc0aa7d61351b641763d254d81b54` — Related; established `INPUT_PROXY_PROTOCOL`.

Target fit: New pull request — the adapter correction is bounded, source-proven, and has no active implementation.

## Direction

Add `INPUT_PROXY_PROTOCOL` and replace the three ineffective mappings with the names consumed by the default binary.

## Bounds

- Preserve: Public action input names, values, and `drone-ssh`-owned validation.
- Exclude: Public input renames, runtime validation, and compatibility claims for unverified custom binary versions.
- Cost: One metadata adapter change plus focused verification of four environment values.

## Verification

- `uses: ./` with a controlled executable wrapper handling `--version` → Records the four recognized environment values exactly.

## Missing

None.

## Resume

Index: Approve pull request draft
Next: Review the exact target and draft, then approve or revise publication.
Done when: The user approves the exact current target and draft.

## Shared change pressure

Copies [S]: Four public-input-to-runtime mappings in the composite adapter.
Pressure [S]: Public inputs and the binary environment contract form one adapter handoff and must change together.
Drift [S]: All four mappings currently diverge from the default binary contract.
Owner: `action.yml` composite step environment.
Cost: Keep the correction explicit; do not introduce a generator or cross-repository schema.

## API and compatibility

Callers [S]: Workflows using `allenvs`, `cipher`, `proxy_cipher`, or non-default `proxy_protocol`.
Contract [S]: `action.yml` exposes the public inputs while `drone-ssh v1.8.2` owns the consumed environment names.
Compatibility: Account for workflows that explicitly forward the current internal environment names through `envs`.
Migration: None for documented action inputs.

## Implementation

Branch: `fix/input-env-mappings`
Base: `upstream/master@b838bc2f27cd449b957159452d432aff91697637`
Scope: Map four existing public inputs to the exact environment names consumed by pinned `drone-ssh v1.8.2`.
Commit: `29227624ecdc540d002ca21a346882febadf83ea`
Push: `origin/fix/input-env-mappings`
Checks:

- `yq v4.53.3` parsed `action.yml` and resolved all four public input declarations → Passed.
- Exact mapping assertions for the four recognized environment names → All returned `true`.
- Absence assertions for `INPUT_ALL_ENVS`, `INPUT_CIPHER`, and `INPUT_PROXY_CIPHER` → All returned `true`.
- `git diff --check` → Passed.

## Draft

Title:

```text
fix: map action inputs to drone-ssh environment names
```

Body:

```markdown
## Summary

The composite adapter exports three public inputs under environment names that pinned `drone-ssh v1.8.2` does not consume and omits `proxy_protocol` entirely.
This change maps the four existing public inputs to the binary's recognized names without changing the action interface.

## Evidence

- [`action.yml` on the current base](https://github.com/appleboy/ssh-action/blob/b838bc2f27cd449b957159452d432aff91697637/action.yml#L105-L141) exports `INPUT_ALL_ENVS`, `INPUT_CIPHER`, and `INPUT_PROXY_CIPHER` and has no `INPUT_PROXY_PROTOCOL`.
- [`drone-ssh v1.8.2`](https://github.com/appleboy/drone-ssh/blob/9d94a36c84c95a692c27313a6726437ed344ae2a/main.go#L102-L106) consumes `INPUT_CIPHERS`.
- The same pinned source consumes [`INPUT_PROXY_PROTOCOL`](https://github.com/appleboy/drone-ssh/blob/9d94a36c84c95a692c27313a6726437ed344ae2a/main.go#L169-L177), [`INPUT_PROXY_CIPHERS`](https://github.com/appleboy/drone-ssh/blob/9d94a36c84c95a692c27313a6726437ed344ae2a/main.go#L233-L237), and [`INPUT_ALLENVS`](https://github.com/appleboy/drone-ssh/blob/9d94a36c84c95a692c27313a6726437ed344ae2a/main.go#L273-L277).
- [Pull request #303](https://github.com/appleboy/ssh-action/pull/303) introduced the composite mapping layer; no active issue or pull request currently fixes these names.

## Changes

- Map `allenvs` to `INPUT_ALLENVS`.
- Map `cipher` and `proxy_cipher` to `INPUT_CIPHERS` and `INPUT_PROXY_CIPHERS`.
- Map `proxy_protocol` to `INPUT_PROXY_PROTOCOL`.
- Preserve every public input name, value, default, and binary-owned validation path.

## Risks and boundaries

- The mappings are proven against the action's default `drone-ssh v1.8.2`; explicit custom version overrides remain outside this compatibility claim.
- No compatibility aliases, action-side validation, documentation changes, or generated mapping layer are added.
- Existing `envs` forwarding remains unchanged.

## Verification

- `yq v4.53.3` parsed `action.yml` and confirmed all four public declarations and exact recognized mappings.
- Assertions confirmed that `INPUT_ALL_ENVS`, `INPUT_CIPHER`, and `INPUT_PROXY_CIPHER` are no longer injected.
- `git diff --check` — passed.

I checked the relevant issues, comments, pull requests, discussions, releases, and both repositories' naming history; this pull request is not a duplicate.

### Disclosure

Investigated thoroughly with GPT-5.6 Codex (high reasoning effort), using [Oh My Pi](https://github.com/can1357/oh-my-pi) as the agent framework.

This report is not generic or unreviewed AI-generated output. Its claims were checked against the cited evidence, and it includes the relevant detail intended to help maintainers resolve the issue.

If reports like this are not useful to the project, please let me know and I will refrain from submitting similar ones. My intent is to help without wasting maintainer time or energy or discouraging their work.

Thank you for your work.
```
