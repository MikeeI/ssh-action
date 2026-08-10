# ISSUE-001 — action metadata: public inputs use ineffective runtime names

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

Root [S]: The composite adapter omits `proxy_protocol` and maps three inputs to names that `drone-ssh v1.8.2` does not read.

## Reach and impact

Reach [S]: Calls using `allenvs`, either cipher input, or a non-default `proxy_protocol` traverse the mismatched adapter.
Impact [S]: Explicit values do not reach the default binary through its recognized names; [N] user-visible failure frequency is not measured.

## Evidence

- [S] `action.yml:132,135,137` — Exports `INPUT_ALL_ENVS`, `INPUT_CIPHER`, and `INPUT_PROXY_CIPHER`.
- [S] `action.yml:105-141` — Contains no `INPUT_PROXY_PROTOCOL` mapping.
- [S] `drone-ssh v1.8.2/main.go` — Reads `INPUT_ALLENVS`, `INPUT_CIPHERS`, `INPUT_PROXY_CIPHERS`, and `INPUT_PROXY_PROTOCOL`.
- [S] `upstream/master@b838bc2f27cd449b957159452d432aff91697637` — Matches the fork for the affected source.

## Prior art

Coverage: Not completed; checked=2026-08-10.
Gaps: Open and closed issues, pull requests, discussions, releases, and relevant `drone-ssh` history remain to be searched.

Target fit: Undecided — prior-art research and focused reproduction are incomplete.

## Direction

Add `INPUT_PROXY_PROTOCOL` and replace the three ineffective mappings with the names consumed by the default binary.

## Bounds

- Preserve: Public action input names, values, and `drone-ssh`-owned validation.
- Exclude: Public input renames, runtime validation, and compatibility claims for unverified custom binary versions.
- Cost: One metadata adapter change plus focused verification of four environment values.

## Verification

- `uses: ./` with a controlled executable wrapper handling `--version` → Records the four recognized environment values exactly.

## Missing

- Complete upstream prior-art search.
- Focused local action reproduction against the recorded source revision.

## Resume

Index: Research adapter prior art
Next: Search upstream issues, pull requests, discussions, releases, and `drone-ssh` history for the four mappings.
Done when: Every plausible prior-art candidate is classified and the target fit is recorded.

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
