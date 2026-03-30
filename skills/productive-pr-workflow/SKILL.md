---
name: productive-pr-workflow
description: "Use when: preparing a PR with Productive linkage. Extract Productive task number from branch name, commit staged changes, generate PR body from .github/PULL_REQUEST_TEMPLATE.md, open PR, optionally move task to Under PR review, and optionally store PR URL in task custom PR link field. Keywords: productive task, task number from branch, PR template, open PR, under PR review, custom PR link field."
---
# Productive PR Workflow
Use this workflow when preparing a pull request with Productive linkage.
## Purpose
Execute a complete delivery flow tied to Productive:
1. Resolve Productive task number from the current git branch name.
2. Commit staged changes.
3. Generate PR description from `.github/PULL_REQUEST_TEMPLATE.md` and insert Productive task link.
4. Open a pull request.
5. Ask user whether to move Productive task to `Under PR review`.
6. If yes, move task status.
7. If task has a custom PR link field, write the PR URL to that field.
## Inputs
- Optional commit message.
- Optional PR title.
- Optional base branch (default to `master` unless user says otherwise).
- Optional explicit task number if branch parsing fails.
## Required Tools
- Terminal and git commands.
- Productive resource APIs for describe/query/update operations.
- User prompt/confirmation flow.
## Workflow
### 1) Resolve branch and task number
- Run `git branch --show-current`.
- Extract task number from branch using regex candidates in order:
  - `(?:task|pt|prod|productive)[-_/# ]?(\d+)`
  - `(?:^|[-_/])(\d{2,})(?:[-_/]|$)`
- If parsing fails, ask user for task number.
- Keep `task_number` as plain digits, for example `683`.
### 2) Resolve Productive task record
- Query `tasks` by `task_number` (`task_number eq <task_number>`).
- If multiple matches appear, ask user to choose.
- Capture:
  - `task id`
  - `project id`
  - existing `custom_fields`
### 3) Commit staged changes
- Validate there are staged changes with `git diff --cached --name-only`.
- If none staged, stop and ask user to stage files.
- Use provided commit message, otherwise default to `chore: update dependencies`.
- Commit with `git commit -m "<message>"`.
### 4) Build PR description from template
- Read `.github/PULL_REQUEST_TEMPLATE.md`.
- Replace placeholders:
  - `__TASK_NUMBER__` -> extracted task number
  - `__ADD_URL_TO_PRODUCTIVE_TASK__` -> `https://app.productive.io/1-infinum/tasks/<task_id>`
- Keep section headings from template.
- Fill `Problem` and `Solution` from commit diff:
  - `git show --name-only --stat --pretty=format:%B HEAD`
  - `git show --pretty=format: --unified=0 HEAD`
### 5) Open PR
- Push current branch if needed: `git push -u origin <branch>`.
- Determine base branch: use user-provided base branch, otherwise `master`.
- Determine PR title: use user-provided title, otherwise first line of commit message, prefixed with task number in square brackets if not already present. e.g. `[#683] Fix login bug`.
- Ask user if the PR should be marked as draft.
- Create PR via GitHub CLI:
  - `gh pr create --base <base> --head <branch> --title "<title>" --body-file <tempfile> --draft <draft>`
- Read PR URL:
  - `gh pr view --json url,number,baseRefName,headRefName,title`
- Add `ai-assisted` label
### 6) Ask whether to move task to Under PR review
- Ask user: `Move Productive task #<task_number> to Under PR review?`
- If answer is no, finish.
### 7) Move task status to Under PR review
- Resolve workflow status id once per project:
  - query `workflow_statuses` by `name eq Under PR review`
  - if not found, list project statuses and ask user to choose.
- Update task with `workflow_status_id` set to the resolved id.
### 8) Write PR URL to custom PR link field when available
- Inspect task schema for custom fields support (`describe_resource(tasks, include: ["schema"])`).
- Load task details and inspect existing `custom_fields` keys.
- Detect candidate custom field by search terms:
  - `pr link`, `pull request`, `github pr`, `merge request`, `mr link`
  - use organization and project metadata search.
- If exactly one candidate is found, set `custom_fields[<field_id>] = <pr_url>`.
- If multiple candidates are found, ask user which field to use.
- If no candidate exists, skip and note it.
## Safety checks
- Never rewrite unrelated git history.
- Use `--force-with-lease` only when explicitly asked.
- Do not change Productive task status without explicit user confirmation.
- If any external call fails, report exact step and continue with remaining safe steps when possible.
## Output contract
Return a compact summary containing:
- commit hash
- PR URL
- Productive task URL
- whether status was changed to Under PR review
- whether custom PR link field was updated (and field id/name)
