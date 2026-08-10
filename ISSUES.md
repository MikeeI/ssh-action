# Issue and Pull Request Tracking

Read this index at the start of every agent session before repository work.
`FORMAT.md` owns research, lifecycle, drafting, implementation, and publication rules.
Each linked `issues/ISSUE-NNN.md` is the complete authoritative record for one root cause.
This file owns `Next finding ID` and projects current issue-file state.
`Next` is a 2–6 word projection of the issue record's `Resume/Index`.
When a row disagrees with its issue file, correct the row from the issue file in the same task.

Next finding ID: ISSUE-007

## Active

| ID | Finding | State | Mode | Target | Priority | Next | Location |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [ISSUE-001](issues/ISSUE-001.md) | action metadata: public inputs use ineffective runtime names | Published | Pull request | New pull request | High | Monitor pull request review | https://github.com/appleboy/ssh-action/pull/414 |
| [ISSUE-002](issues/ISSUE-002.md) | entrypoint: static output delimiter can truncate captured stdout | Published | Pull request | New pull request | High | Monitor pull request review | https://github.com/appleboy/ssh-action/pull/415 |
| [ISSUE-003](issues/ISSUE-003.md) | entrypoint: Windows Git Bash cannot select the released executable | Hold | Undecided | Undecided | High | Run Windows bootstrap | Not published. |
| [ISSUE-004](issues/ISSUE-004.md) | action output: drone-ssh merges stderr and status into stdout | Hold | Undecided | Undecided | High | Research stream ownership | Not published. |
| [ISSUE-005](issues/ISSUE-005.md) | documentation: version input promises latest instead of the pinned default | Hold | Undecided | Undecided | High | Confirm version intent | Not published. |
| [ISSUE-006](issues/ISSUE-006.md) | entrypoint: failed capture leaves output record unterminated | Published | Pull request | New pull request | High | Monitor pull request review | https://github.com/appleboy/ssh-action/pull/413 |

## Terminal

| ID | Finding | State | Mode | Target | Priority | Outcome | Location |
| --- | --- | --- | --- | --- | --- | --- | --- |
