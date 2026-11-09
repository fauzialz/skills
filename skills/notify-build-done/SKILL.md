---
name: notify-build-done
description: Watch a GitHub Actions run until it finishes, then send a short non-technical Slack DM (in Indonesian) telling QA, PM or another non-technical person that the app's bug fix or feature "sudah bisa dicek". Run only when the user explicitly invokes it with a run URL and a person's name.
disable-model-invocation: true
---

# Skill: notify-build-done

Tell a non-technical person that a build is ready to check. Run the steps in order.

Input from the user: a GitHub Actions run URL (or run id) and who to message.
Example: `/notify-build-done https://github.com/<owner>/<repo>/actions/runs/123 to Annisa`

## Rule: asking the user

Whenever you need to ask the user anything, use the AskUserQuestion tool if it exists.
Fall back to a plain-text question only when it does not. Ask one thing at a time, with a
recommended option first.

Anything you write to the user (questions, status, final report) must be easy to read for a
person with ADHD: short bullets, plain words, one idea per line, most important point first.

## 0. Preflight

If one is missing, stop and tell the user what to fix.

- `gh auth status` works.
- The Slack MCP is connected (`users_search`, `conversations_add_message`).
- The Atlassian MCP is connected. Only needed if step 3 has to fall back to Jira.
- A run URL or run id was given. If not, ask for it.
- A recipient was named. If not, ask who to message. Never pick a default person.

Parse the URL: `github.com/<owner>/<repo>/actions/runs/<run-id>[/job/<job-id>]`.

## 1. Check the run

```
gh run view <run-id> --repo <owner>/<repo> --json status,conclusion,headBranch,headSha,displayTitle
```

If the URL has a `/job/<job-id>`, also check that job:
`gh api repos/<owner>/<repo>/actions/jobs/<job-id> --jq '.status + " " + (.conclusion // "")'`.
Use the job result as the deciding result. Otherwise use the run result.

- `completed` with `success`: go to step 3.
- `completed` with anything else (failure, cancelled, timed out): go to step 4.
- Not completed: go to step 2.

## 2. Wait without burning tokens

Waiting must cost nothing. Do not poll from the model.

1. Start the watcher as a background Bash command (`run_in_background`), output thrown away:
   `gh run watch <run-id> --repo <owner>/<repo> --exit-status >/dev/null 2>&1`
2. Tell the user once: waiting for the run, the Claude session must stay open.
3. Do nothing else. Do not use `/loop` or `ScheduleWakeup` to poll. Do not read the
   watcher's output file.
4. When the watcher exits, you are woken up. Run step 1 again to read the real result.
5. If the watcher stopped but the run is still not completed, do not guess. Tell the user
   the run is still going and stop.

## 3. Build the message

Find what is in this build. Use the cheapest source that works. Stop at the first one that
gives a clear app name and bug or feature name.

1. **Chat context first.** If this session already shows the work (ticket key, ticket
   title, what was fixed or built, app name) and the chat has not switched to other work,
   reuse it. Do not call Jira.
2. **Jira only if the chat has no usable name.**
   - Ticket keys (pattern `[A-Z][A-Z0-9]+-\d+`): from the commit messages and PR titles for
     `headSha`
     (`gh pr list --repo <owner>/<repo> --search <headSha> --state all --json title,number`).
   - For each key, `getJiraIssue` (Atlassian MCP). Take the title and issue type.
3. **Still unclear?** Ask the user for the name.
4. App name: the product name the person knows (chat context, the Jira title prefix like
   `[RMS]`, or the repo name). If unsure, ask the user. Never use branch names, pipeline
   names, or repo slugs.

If the chat covered several pieces of work, name all of them, not just the latest.

Write the message in Indonesian. Rules:

- Short. One idea per line.
- Contains: app name + bug or feature name + "sudah bisa dicek".
- Rewrite ticket titles in plain words. No technical terms: no API, hook, refactor,
  pipeline, branch, staging, deploy, ticket keys.
- No links. No GitHub words.
- Start with "Hi <first name>,".

One item:

```
Hi Annisa, <App>: <nama fitur / bug> sudah bisa dicek 🙌
```

Several items:

```
Hi Annisa, <App> sudah bisa dicek:
- <fitur / bug 1>
- <fitur / bug 2>
```

## 4. Failed or cancelled run

Do not send anything to Slack. Tell the user:

- the run did not succeed, and the result (failed / cancelled)
- the run URL
- that nobody was messaged

Stop.

## 5. Send

1. `users_search` with the name. If there are several matches, ask which one. If there is
   none, tell the user and stop.
2. Send the message with `conversations_add_message` to the person's `DMChannelID`.
   Use `content_type` `text/plain`.
3. Never message anyone other than the named person.

## Final report

A few short lines:

- run result
- who was messaged
- the exact message sent
