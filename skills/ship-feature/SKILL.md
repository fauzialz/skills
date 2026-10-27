---
name: ship-feature
description: Ship finished feature work end to end - commit uncommitted changes grouped by context and Jira ticket, push, open a PR that follows the repo's PR template, sync a release-ready twin branch via cherry-pick, and move the Jira ticket to a testing status. Run only when the user explicitly invokes it.
disable-model-invocation: true
---

# Skill: ship-feature

Take feature work from "done coding" to "ready to test". Run the steps in order. Skip a
step only when it has nothing to do (e.g. no uncommitted changes).

## Rule: asking the user

Whenever you need to ask the user anything, use the AskUserQuestion tool if it exists.
Fall back to a plain-text question only when it does not. Ask one thing at a time, with a
recommended option first.

Never force-push. Never rewrite history.

User rules override any default attribution text: no `Co-Authored-By` trailer on commits
and no "Generated with Claude Code" line in PR bodies, unless the user asks for them.

## 0. Preflight

Check before doing anything. If one is missing, stop and tell the user what to fix.

- `gh auth status` works.
- The Atlassian MCP is connected. Get the `cloudId` and site URL from
  `getAccessibleAtlassianResources`. If no ticket is known anywhere, skip step 5 with a note.

## 1. Commit

Skip if the working tree is clean.

1. Run `git status` and `git diff`. Group changes by context (one logical change per group).
2. Find the Jira ticket key for each group from the chat context and what the user said.
   Do not read it from the branch name; branch names may be generic.
3. If a group has no known ticket, ask the user for one (or "no ticket").
4. Read recent `git log` and any commit convention docs. Write messages in that style and
   put the ticket key where that style puts it.
5. Do not commit secrets or junk (`.env`, keys, build output, large binaries). Leave them
   out and tell the user, or ask if unsure.
6. Show the planned commits (files, message, ticket), then create them.
7. If a pre-commit hook fails, fix the cause and make a new commit. Never use `--no-verify`.

## 2. Push

Push the current branch. If the user named another branch, run `git push origin HEAD:<branch>`.
If the target is `main`, `master`, or a release branch, ask for confirmation first.
Use `-u` when there is no upstream yet.

## 3. Pull request

0. Check for an open PR on this branch (`gh pr view`). If one exists, push only and update
   its body if needed. Do not create a second PR.
1. Ask which base branch the PR goes to. Suggest a default (the branch this was cut from,
   else the repo default).
2. Find the PR template: `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE/`,
   `docs/`, or repo root. Keep its headings and fill every section.
   No template: use Summary / Why / How to test / Ticket.
3. Build the ticket link with the Atlassian MCP (site base URL plus ticket key). List every
   ticket involved.
4. Write the body for a reader with ADHD:
   - short bullets, plain words, one idea per line
   - the most important point first
   - no emojis
5. Create the PR with `gh pr create` and return the URL.

## 4. Twin branch (release-ready copy)

Goal: the feature can ship alone with one PR to `main`, a release branch, or a tag's
release branch.

1. Look for an existing twin of the feature branch named `<feature>-pair`,
   `<feature>-release`, `<feature>-main`, or `<feature>-master` (local and remote).
   Reuse it if found.
2. If none exists, ask the user to confirm creating `<feature>-release`, and which base to
   cut it from: `main` (default), a release branch, or a tag. Create it from that base.
3. Work in a temporary `git worktree` so the user's checkout and state stay untouched.
   Remove the worktree when done.
4. Candidate commits are `<PR base>..HEAD` of the feature branch, minus merge commits and
   minus commits already on the twin (`git cherry`). If the list looks odd (other
   features' commits), show it and ask before picking.
5. Cherry-pick in order with `-x`.
6. On a conflict: abort that pick, show the conflicting files, and ask the user. Do not
   auto-resolve.
7. Push the twin with a plain `git push`. Open a PR from the twin only if the user asks.
   If the twin was cut from a tag, ask which branch that PR should target.

## 5. Jira

1. Use the Atlassian MCP: `getTransitionsForJiraIssue` for each ticket.
2. Pick the closest testing status ("Ready to test", "In QA", "Ready for QA", or similar).
   If it is not directly reachable, transition through the intermediate statuses.
3. If a transition needs required fields you cannot fill, stop on that ticket and tell the
   user.
4. Report the path taken (e.g. "To Do -> In Progress -> Ready to test").

## Final report

End with a short list: commits made, branch pushed, PR URL, twin branch and picked
commits, ticket status path. Keep it to a few lines.
