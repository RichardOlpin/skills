---
name: scope
description: Keep a session on one GitHub issue. Claim it at the start, judge every tangent as absorb-or-park, and report progress toward the issue's "Done when". Use when a session starts work ("work on #N", "pick up X"), when an adjacent problem turns up mid-task (a bug, cleanup, refactor, or doc gap the issue does not name), or when the user asks for status, progress, or "where are we".
---

A session works **one issue**: its **session issue**. The issue says what done looks like, the branch name carries its number, and GitHub holds the progress. Nothing lives only in the conversation, so a fresh session or a second terminal can see exactly where things stand.

## Start

1. **Get an issue number.** If the user gave none, search open issues (`gh issue list --search "<words>"`). If nothing fits, draft a title and a `Done when:` line, show them, and create the issue once the user agrees. No issue, no work.
2. **Check the claim.** `gh issue view <n> --json state,labels,assignees,body`. If it carries the `in-progress` label, another session holds it: stop, show the branch (`git branch -r --list '*/<n>-*'`), and ask the user. Never work an issue another session claimed.
3. **Claim it.** `gh issue edit <n> --add-label in-progress --add-assignee @me`. Name the branch `<type>/<n>-<slug>` and create the worktree the way the repo's `AGENTS.md` says.
4. **Restate the scope in three lines:** the goal, the `Done when`, and the milestones (open sub-issues) if the issue has any. If the issue has no `Done when`, propose one and add it to the body before writing code.
5. **Tell the user** they can type `/goal <the Done when>` so Claude checks it before stopping, and `/rename <n>-<slug>` so this terminal is easy to tell apart from the others. Those are the user's commands; do not try to run them.

Later in the session, recover the session issue from `git branch --show-current`, not from memory.

## Tangents: absorb or park

Before doing any work the session issue does not name, whether you found it or the user raised it, decide:

**Absorb** it only when all of these hold:

- the `Done when` cannot be met without it, or it sits in lines you are already changing and fits in the same small commit;
- it changes no behavior outside the session issue's area;
- it needs no decision from the user.

**Park** it otherwise. Parking is `gh issue create` with a title, what you saw (`file:line`, error text), why it matters, and `Found while working on #<n>`. If it serves the same parent goal, attach it as a sub-issue of that parent (`gh issue edit <parent> --add-sub-issue <new>`); otherwise leave it standalone. Then carry on with the session issue.

Say which you chose in one line: `Tangent: <what>. Parked as #<m>: <why>.` or `Tangent: <what>. Absorbing: <why>.` Only ask when the call is genuinely close, and then lead with your recommendation. When the user starts down a tangent, apply the same test and point it out once; the user decides.

## Status

When the user asks for progress, report against the session issue:

- **Done when**, and how close the branch is to it (what is committed, what is left).
- **Milestones**, if the issue or its parent has sub-issues: `gh api repos/{owner}/{repo}/issues/<n> --jq .sub_issues_summary` for the count and percent, and `gh api repos/{owner}/{repo}/issues/<n>/sub_issues --jq '.[] | "#\(.number) \(.state) \(.title)"'` for the list.
- **Parked this session**: the issues you filed, by title.
- **Next step**: one line.

For "what is every session doing", list `gh issue list --label in-progress --json number,title,assignees`.

## Finish

- The PR body says `Closes #<n>`. A parked tangent is never closed by this PR.
- If the session ends before the issue is done, comment the progress and the next step on the issue (`gh issue comment <n>`), then release the claim with `gh issue edit <n> --remove-label in-progress` so the next session can take it.
