# Issue and Pull Request Publication Status

This file is the sole source of truth for every finding's ID, delivery mode, lifecycle status, evidence, and location.
Read and update this ledger instead of inferring state from chat history, clone reports, or earlier reviews.
`FORMAT.md` owns research, drafting, implementation authorization, approval, and publication rules.

Next finding ID: ISSUE-2026-007

## Findings

### ISSUE-2026-001 — action metadata: public inputs use ineffective runtime names

- Status: Hold.
- Delivery mode: Undecided.
- Location: Not published.
- Evidence class: Source-proven.
- Internal priority: High.
- Confidence: High.
- Type: mapping.
- Publication target: Undecided.
- Summary: The composite-action adapter omits `proxy_protocol` and maps the cipher and `allenvs` inputs to names that `drone-ssh v1.8.2` does not read.
- Evidence: Current `upstream/master` commit `b838bc2f27cd449b957159452d432aff91697637` matches the fork for the affected files.
- Evidence: `action.yml:132,135,137` exports `INPUT_ALL_ENVS`, `INPUT_CIPHER`, and `INPUT_PROXY_CIPHER`, while `action.yml:105-141` has no `INPUT_PROXY_PROTOCOL`.
- Evidence: `drone-ssh v1.8.2` `main.go` reads `INPUT_ALLENVS`, `INPUT_CIPHERS`, `INPUT_PROXY_CIPHERS`, and `INPUT_PROXY_PROTOCOL`.
- Shared change pressure: The public action inputs and the binary environment contract describe one composite adapter handoff and must change together.
- Impact: Source proves that explicit `allenvs`, `cipher`, `proxy_cipher`, and non-default `proxy_protocol` values do not reach the default binary through its recognized names; user-visible failures are not yet reproduced.
- Proposed direction: Add `INPUT_PROXY_PROTOCOL` and replace the three ineffective environment names with the names consumed by the default binary.
- Risks and boundaries: Preserve public input names and values, leave validation to `drone-ssh`, account for workflows that expose the current internal environment names through `envs`, and scope compatibility claims to verified binary versions.
- Verification: Run `uses: ./` with a controlled executable wrapper that handles `--version` and records the four recognized environment values exactly.
- Missing publication evidence: Search all upstream prior art and reproduce the complete mapping through a focused local action invocation before selecting a publication target.

### ISSUE-2026-002 — entrypoint: static output delimiter can truncate captured stdout

- Status: Hold.
- Delivery mode: Undecided.
- Location: Not published.
- Evidence class: Source-proven.
- Internal priority: High.
- Confidence: High.
- Type: output.
- Publication target: Undecided.
- Summary: The `capture_stdout` serializer uses the fixed control line `EOF`, which is also a valid line in arbitrary remote output.
- Evidence: Current `upstream/master` commit `b838bc2f27cd449b957159452d432aff91697637` matches the fork for `entrypoint.sh`.
- Evidence: `entrypoint.sh:73-76` opens and closes the `stdout` output with a static `EOF` delimiter while streaming unmodified binary stdout between those lines.
- Evidence: `action.yml:88-91` exposes the captured value as the action's only output.
- Shared change pressure: The arbitrary command-output stream and its `GITHUB_OUTPUT` framing must remain distinguishable for every captured value.
- Impact: A standalone `EOF` output line deterministically terminates the value early and can make subsequent lines invalid workflow-command data; occurrence in real workflows is not measured.
- Proposed direction: Generate one high-entropy delimiter per invocation and use it for both framing lines while retaining the current streaming `tee` path.
- Risks and boundaries: Preserve `pipefail`, binary exit propagation, live logs, and multiline content; a generated delimiter has negligible but nonzero theoretical collision risk.
- Verification: Capture the three remote lines `vor`, `EOF`, and `nach`, assert exact ordered preservation, and repeat with a failing remote command to confirm unchanged failure propagation.
- Missing publication evidence: Search all upstream prior art and run the focused delimiter reproduction against current `upstream/master`.

### ISSUE-2026-003 — entrypoint: Windows Git Bash cannot select the released executable

- Status: Hold.
- Delivery mode: Undecided.
- Location: Not published.
- Evidence class: Observed and Source-proven.
- Internal priority: High.
- Confidence: High.
- Type: mapping.
- Publication target: Undecided.
- Summary: The launcher rejects Git for Windows platform names and constructs a suffixless target even though the selected Windows release is an `.exe`.
- Evidence: Current `upstream/master` commit `b838bc2f27cd449b957159452d432aff91697637` matches the fork for `action.yml` and `entrypoint.sh`.
- Evidence: `action.yml:101-104` runs the entrypoint with Bash, while `entrypoint.sh:24-30` accepts only literal `darwin`, `linux`, or `windows`.
- Evidence: `env SSH_CLIENT_OS=MINGW64_NT-10.0 GITHUB_ACTION_PATH=/tmp INPUT_CURL_INSECURE=false ./entrypoint.sh` exits with code 2 and reports `Unknown or unsupported platform`.
- Evidence: Open upstream issue `https://github.com/appleboy/ssh-action/issues/362` reports `MINGW64_NT-*` failure and contains a second user report of the same error.
- Evidence: The `drone-ssh v1.8.2` release names its Windows AMD64 executable `drone-ssh-1.8.2-windows-amd64.exe`, and its `.goreleaser.yaml` excludes Windows/ARM64.
- Shared change pressure: Runner platform normalization and release-artifact naming jointly determine one executable target and must describe the same supported matrix.
- Impact: The Windows x64 Git Bash path is observed to fail before download, version validation, or SSH execution; the complete corrected journey is not yet verified.
- Proposed direction: Normalize the supported `mingw*` and `msys*` platform families to `windows` and use the `.exe` suffix consistently for Windows targets.
- Risks and boundaries: Do not claim Windows ARM64 support, preserve explicit platform and architecture overrides, and scope custom-version compatibility to releases with the same asset schema.
- Verification: On `windows-2025` with `shell: bash`, set `GITHUB_ACTION_PATH`, `INPUT_CURL_INSECURE=false`, and `INPUT_CAPTURE_STDOUT=false`, then run `bash ./entrypoint.sh --version` and assert selection and execution of the `.exe` asset.
- Missing publication evidence: Search all upstream prior art and complete the `windows-2025` bootstrap reproduction.

### ISSUE-2026-004 — action output: drone-ssh merges stderr and status into stdout

- Status: Hold.
- Delivery mode: Undecided.
- Location: Not published.
- Evidence class: Source-proven.
- Internal priority: High.
- Confidence: High.
- Type: output.
- Publication target: Undecided.
- Summary: The default `drone-ssh` binary writes remote stderr and its final success status to the stdout stream captured by the action.
- Evidence: Current `upstream/master` commit `b838bc2f27cd449b957159452d432aff91697637` matches the fork for `action.yml`, `entrypoint.sh`, and `README.md`.
- Evidence: `action.yml:81-91` and `README.md:128-134,384-403` describe the action output as standard output from executed commands.
- Evidence: `entrypoint.sh:73-76` captures the complete binary stdout stream into `GITHUB_OUTPUT`.
- Evidence: In `drone-ssh v1.8.2` `plugin.go`, both `stdoutChan` and `stderrChan` call stdout-backed `p.log`, and `Plugin.Exec` prints its three-line success status with `fmt.Println`.
- Shared change pressure: Remote stdout, remote stderr, and binary status must retain distinct stream ownership for the action output contract to remain truthful.
- Impact: Source proves that captured output contains data beyond remote stdout; downstream parsing and exact-match impact are not measured.
- Proposed direction: Give `drone-ssh` separate stdout and error/status writers, release that contract, and update the action's default pin only after the corrected release exists.
- Risks and boundaries: The corrective owner is `drone-ssh`, direct binary consumers may observe stream changes, and `ssh-action` must not parse arbitrary remote text to remove status lines.
- Verification: Execute a remote command that writes distinct stdout and stderr markers, assert that the action output contains only stdout, and confirm that stderr plus final status remain visible in the step log.
- Missing publication evidence: Search prior art in both repositories, reproduce the current mixed output, and establish the appropriate cross-repository publication target.

### ISSUE-2026-005 — documentation: version input promises latest instead of the pinned default

- Status: Hold.
- Delivery mode: Undecided.
- Location: Not published.
- Evidence class: Source-proven.
- Internal priority: High.
- Confidence: High.
- Type: duplicated decision.
- Publication target: Undecided.
- Summary: All three public README tables describe an automatic latest-version policy that the runtime does not implement.
- Evidence: Current `upstream/master` commit `b838bc2f27cd449b957159452d432aff91697637` matches the fork for the affected files.
- Evidence: `entrypoint.sh:9` selects `DRONE_SSH_VERSION="${DRONE_SSH_VERSION:-1.8.2}"`, and `action.yml:141` passes the optional input to that runtime owner.
- Evidence: `README.md:101`, `README.zh-cn.md:101`, and `README.zh-tw.md:101` state that omission uses the latest version and show no default.
- Shared change pressure: The runtime pin and every public description of the omitted-input behavior represent one release-selection decision.
- Impact: Source proves that documented automatic updates do not occur; resulting user confusion or compatibility failures are not measured.
- Proposed direction: Describe omission as using the action-pinned version, currently `1.8.2`, in `action.yml` and all three README tables without changing runtime behavior.
- Risks and boundaries: Keep `entrypoint.sh` as the runtime owner and update all public descriptions whenever the pin changes; do not add unrelated input-format claims.
- Verification: Compare the final metadata and three tables against the empty and explicit `DRONE_SSH_VERSION` paths in `entrypoint.sh`.
- Missing publication evidence: Search all upstream prior art and confirm maintainer intent for how the pinned default should be presented before selecting a publication target.

### ISSUE-2026-006 — entrypoint: failed capture leaves output record unterminated

- Status: Hold.
- Delivery mode: Undecided.
- Location: Not published.
- Evidence class: Observed and Source-proven.
- Internal priority: High.
- Confidence: High.
- Type: output.
- Publication target: Undecided.
- Summary: A nonzero `drone-ssh` pipeline exits the strict-mode entrypoint before it writes the closing `GITHUB_OUTPUT` delimiter.
- Evidence: Current `upstream/master` commit `b838bc2f27cd449b957159452d432aff91697637` matches the fork for `action.yml`, `entrypoint.sh`, and `.github/workflows/main.yml`.
- Evidence: `entrypoint.sh:3,73-76` enables `pipefail`, opens `stdout<<EOF`, runs `drone-ssh` through `tee`, and writes the closing delimiter only after the pipeline succeeds.
- Evidence: `.github/workflows/main.yml:823-835` exercises a failing remote script with `capture_stdout: true` but does not assert the resulting output record.
- Evidence: Open upstream issue `https://github.com/appleboy/ssh-action/issues/375` observes `Invalid value. Matching delimiter not found 'EOF'` and unavailable failure output after a remote command exits nonzero.
- Shared change pressure: Pipeline exit propagation and output framing form one capture lifecycle and must complete the output record before returning the command result.
- Impact: Source and the upstream reproduction prove that failed captured commands add a secondary runner parser error and do not expose the captured output to later failure-handling steps.
- Proposed direction: Execute the pipeline as an `if` condition, capture its real failure status, close the output record with the delimiter on its own line, and return the saved status after successful framing.
- Risks and boundaries: Preserve `tee` failure handling, successful capture behavior, and the original command status; fixed-delimiter collision remains separately owned by ISSUE-2026-002.
- Verification: Use a controlled executable that writes stdout without a final newline and exits 17, then assert exit status 17, live output, and one complete parseable `stdout` record.
- Missing publication evidence: Read all prior work around issue 375 and reproduce the failure against current `upstream/master` before selecting a publication target.
