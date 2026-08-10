# ISSUE-003 — entrypoint: Windows Git Bash cannot select the released executable

State: Hold
Mode: Undecided
Target: Undecided
Location: Not published.
Priority: High
Confidence: High
Type: compatibility
Created: 2026-08-10
Updated: 2026-08-10
Source: `upstream/master@b838bc2f27cd449b957159452d432aff91697637`

## Root

Root [O]: The launcher rejects Git for Windows platform names and constructs a suffixless target although the selected Windows release is an `.exe`.

## Reach and impact

Reach [S]: Windows x64 invocations under supported Git for Windows Bash platform families reach the platform and asset selectors.
Impact [O]: The observed path exits before download, version validation, or SSH execution; [N] the corrected end-to-end journey is not measured.

## Evidence

- [S] `action.yml:101-104` — Runs the entrypoint with Bash.
- [S] `entrypoint.sh:24-30` — Accepts only literal `darwin`, `linux`, or `windows`.
- [O] `env SSH_CLIENT_OS=MINGW64_NT-10.0 GITHUB_ACTION_PATH=/tmp INPUT_CURL_INSECURE=false ./entrypoint.sh` → Exits 2 with `Unknown or unsupported platform`.
- [O] `https://github.com/appleboy/ssh-action/issues/362` — Reports `MINGW64_NT-*` failure and a second user report of the same error.
- [S] `drone-ssh v1.8.2` release assets — Publish `drone-ssh-1.8.2-windows-amd64.exe`; `.goreleaser.yaml` excludes Windows/ARM64.
- [S] `upstream/master@b838bc2f27cd449b957159452d432aff91697637` — Matches the fork for `action.yml` and `entrypoint.sh`.

## Prior art

Coverage: Issue 362 read; broader issues, pull requests, discussions, releases, and relevant `drone-ssh` history incomplete; checked=2026-08-10.
Gaps: Full upstream search and current `windows-2025` execution.

- `https://github.com/appleboy/ssh-action/issues/362` — Related; owns the same platform-rejection symptom and would receive new asset-suffix diagnosis.

Target fit: Issue 362 comment is the leading target if Report mode is selected and the complete search finds no competing implementation.

## Direction

Normalize supported `mingw*` and `msys*` platform families to `windows` and use `.exe` consistently for Windows targets.

## Bounds

- Preserve: Explicit platform and architecture overrides plus existing Linux and Darwin selection.
- Exclude: Windows ARM64 support and generic support claims beyond the selected release matrix.
- Cost: One platform-normalization and artifact-naming change plus one Windows x64 bootstrap check.

## Verification

- On `windows-2025` with `shell: bash`, set `GITHUB_ACTION_PATH`, `INPUT_CURL_INSECURE=false`, and `INPUT_CAPTURE_STDOUT=false`; run `bash ./entrypoint.sh --version` → Selects and executes the `.exe` asset.

## Missing

- Complete upstream prior-art search.
- Current `windows-2025` bootstrap reproduction.

## Resume

Index: Run Windows bootstrap
Next: Execute the focused `windows-2025` bootstrap scenario against the recorded source revision.
Done when: Platform values, selected asset, download, and `--version` result are recorded.

## Bug reproduction

Environment: Current entrypoint on Linux with explicit `SSH_CLIENT_OS=MINGW64_NT-10.0`; external issue reports Windows Git Bash.
Reproduction: Run the recorded environment override command before any download occurs.
Actual [O]: Exit code 2 and `Unknown or unsupported platform: MINGW64_NT-10.0`.
Expected: Normalize the supported Windows Bash family and select the published Windows AMD64 `.exe`.

## API and compatibility

Callers [S]: Windows x64 action invocations and explicit platform overrides.
Contract [S]: Platform normalization and release naming jointly determine one executable target.
Compatibility: Do not claim or silently route Windows ARM64 when the selected release has no artifact.
Migration: None for supported Windows x64 callers.
