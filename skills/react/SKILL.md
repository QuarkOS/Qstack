---
name: react
description: >-
  End-to-end React repo change workflow. Use for /react, React PR work on
  react/react, or when the human wants poteto-mode build plus lauren-mode and
  eps1lon-mode dual review before publish.
disable-model-invocation: true
---

# react

Run React work as a gated pipeline. Build under `/poteto-mode`. Prove with verification and `/swarm`. Open the PR the poteto way. Dual-review as Lauren and eps1lon. Fix. Then publish.

## Skills this pipeline loads

Read each skill in full before the step that needs it. Do not paste their contents here.

- `/poteto-mode` from the **pstack** plugin. Playbooks, principles, opening-a-pr, shipping.
- `/create-verification-skill` from the **pstack** plugin. Project-local `verify-<app>` generation.
- `/swarm` from the **pstack** plugin. Parallel coverage or races.
- [`lauren-mode`](../lauren-mode/SKILL.md) in this plugin. Compiler and poteto React habits.
- [`eps1lon-mode`](../eps1lon-mode/SKILL.md) in this plugin. Fiber/DOM/Flight and review habits.

If `lauren-mode` or `eps1lon-mode` is missing from this plugin install, stop and say which path is absent. Do not invent their rules.

If pstack is not installed, stop and tell the human to install pstack first. `/react` does not reimplement poteto-mode.

## Pipeline

Open a todolist with these steps verbatim before other todos. Mark skip with a one-line reason when a step does not apply.

1. Enter poteto-mode
2. Ensure verification skill
3. Build the change
4. Swarm verify
5. Open the PR
6. Dual review
7. Fix review issues
8. Publish

### 1. Enter poteto-mode

Load `/poteto-mode`. Match the task to a playbook. Copy that playbook's steps into the todolist under this pipeline. Nontrivial design goes through the how or architect triggers poteto-mode already names.

### 2. Ensure verification skill

If the workspace has no project-local `.cursor/skills/verify-*/SKILL.md` that covers the surface you will change, run `/create-verification-skill` first and prove it once on one mapped feature.

If a verify skill already exists, read it and use it. Do not regenerate from scratch unless it is broken against the current checkout.

### 3. Build the change

Execute the matched poteto-mode playbook. Prefer the smallest diff that fixes the root cause. For compiler work, bias to `lauren-mode` PR craft. For Fiber, DOM, Flight, test, or CI work, bias to `eps1lon-mode` PR craft. When both apply, one concern per PR and stack.

### 4. Swarm verify

Load `/swarm`. Frame a done predicate against the real verify skill and repo tests. Fan out coverage slices when the change spans packages or release lines. Race only when the human asked for a bakeoff.

Aggregate to PASS, ISSUES, or BLOCKED. Do not open a PR on BLOCKED. Fix ISSUES before step 5.

Also run the verify skill's doctor and one drive path that exercises the changed behavior. Keep proof artifacts where the verify skill names.

### 5. Open the PR

Follow poteto-mode's Opening a PR playbook. Worktree off main. deslop and no-comments before review. technical-writing then unslop for title and body.

Title style follows the area. Compiler changes use lauren-mode brackets. Runtime or infra changes use eps1lon-mode brackets. Do not mix both styles in one title.

Return the PR URL. Do not babysit yet.

### 6. Dual review

Spawn two local review workers in one message. Both read the same PR diff and linked proof. Neither edits code.

- Worker A loads `lauren-mode` and reviews as that skill. Report PASS, ISSUES, or BLOCKED with cited paths.
- Worker B loads `eps1lon-mode` and reviews as that skill. Same report shape.

Use `subagent_type: "react-agent"` when available, otherwise `poteto-agent`. Set `run_in_background: true`. Prefer `environment: "local"` so they can read the worktree and `gh`.

You own the merge of both reports. Deduplicate. Rank correctness and security above nits. Discard pure style noise with a one-line reason.

### 7. Fix review issues

Fix every ISSUES item that survives triage. Re-run the verify path and any failing suite named in the reports. Re-swarm only the slices that were red.

If a reviewer report is wrong, dismiss it with evidence in the PR thread or in the chat summary. Do not churn the diff for a false claim.

Loop dual review once after substantive fixes. Stop after one clean dual PASS or after the human redirects.

### 8. Publish

Publish means the PR is ready for human or core-team merge, not a silent force-merge to `react/react` main.

- Ensure the PR is open and ready, not draft.
- Push the fixed branch.
- Paste the PR URL plus the dual-review verdict and verify evidence pointers.
- Run poteto-mode Shipping only when the human explicitly asks to land a green stack. Otherwise stop at ready.

Never force-push shared main. Never merge `react/react` without an explicit human land request.

## Defaults

Project verify skills live under `.cursor/skills/verify-<app>/`.

If the checkout is not `react/react` or a React workspace fork, say so and ask whether to continue with the same pipeline on the current repo.
