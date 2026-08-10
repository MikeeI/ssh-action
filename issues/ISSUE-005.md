# ISSUE-005 — documentation: version input promises latest instead of the pinned default

State: Hold
Mode: Undecided
Target: Undecided
Location: Not published.
Priority: High
Confidence: High
Type: maintainability
Created: 2026-08-10
Updated: 2026-08-10
Source: `upstream/master@b838bc2f27cd449b957159452d432aff91697637`

## Root

Root [S]: All three public README tables describe an automatic latest-version policy that the runtime does not implement.

## Reach and impact

Reach [S]: Users and maintainers consulting any public parameter table encounter the wrong omitted-input behavior.
Impact [S]: Omission selects pinned `1.8.2`, not an automatically resolved latest release; [N] resulting confusion or compatibility failures are not measured.

## Evidence

- [S] `entrypoint.sh:9` — Selects `DRONE_SSH_VERSION="${DRONE_SSH_VERSION:-1.8.2}"`.
- [S] `action.yml:141` — Passes the optional public input to the runtime owner.
- [S] `README.md:101`, `README.zh-cn.md:101`, and `README.zh-tw.md:101` — State that omission uses the latest version and show no default.
- [S] `upstream/master@b838bc2f27cd449b957159452d432aff91697637` — Matches the fork for the affected files.

## Prior art

Coverage: Not completed; checked=2026-08-10.
Gaps: Upstream issues, pull requests, discussions, releases, and maintainer intent for default-version presentation remain to be searched.

Target fit: Undecided — current wording intent and prior art are unresolved.

## Direction

Describe omission as using the action-pinned version, currently `1.8.2`, in `action.yml` and all three README tables without changing runtime behavior.

## Bounds

- Preserve: `entrypoint.sh` as runtime owner and explicit `version` overrides.
- Exclude: Dynamic latest-version resolution, input-format changes, and unrelated translation cleanup.
- Cost: Four public-description updates that must remain synchronized when the runtime pin changes.

## Verification

- Compare metadata and all three tables against empty and explicit `DRONE_SSH_VERSION` paths → Every surface describes the actual selection behavior.

## Missing

- Complete upstream prior-art search.
- Maintainer intent for presenting the pinned default.

## Resume

Index: Confirm version intent
Next: Search upstream history and discussions for the intended omitted-version contract and presentation.
Done when: Maintainer intent or authoritative history is recorded and the contribution target is classified.

## Shared change pressure

Copies [S]: Runtime pin, action metadata description, and three translated README rows.
Pressure [S]: Every surface describes one omitted-input release-selection decision.
Drift [S]: Runtime selects `1.8.2` while all README tables say latest.
Owner: `entrypoint.sh` owns selection; metadata and README tables consume that decision.
Cost: Keep explicit synchronized descriptions; do not add a generator for four short rows.

## API and compatibility

Callers [S]: Workflows omitting or explicitly setting `version` and users relying on the public documentation.
Contract [S]: Empty input selects the action-owned pin; explicit input overrides it.
Compatibility: Documentation changes only; runtime selection remains unchanged.
Migration: None.
