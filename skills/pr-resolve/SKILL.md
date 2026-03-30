---
name: pr-resolve
description: Fetch and critically analyze PR discussion threads for the current branch. Do **not** make any changes to the codebase unless the user explicitly asks.
disable-model-invocation: true
---

## Context

This workflow is for reviewing and addressing PR feedback. Focus on threads **not** initiated by the PR author, unless explicitly instructed otherwise.

---

## Workflow

### 1. Fetch the PR URL
- Use `gh` CLI to find the PR URL based on the currently checked-out branch.

### 2. Fetch and analyze PR details
- Retrieve all PR comments and threads using the PR URL.
- Identify and record the PR author.
- Filter out any threads/comments **initiated by the PR author**.
- Confirm the filtered set before proceeding: state how many threads will be reviewed and by whom they were raised.

### 3. Iterate through each relevant thread

For each thread **not** initiated by the PR author:

1. **Summarize the thread** — what the reviewer is asking or pointing out, and which file/lines it references.
2. **Show relevant code** — include the code snippet being discussed for context.
3. **Form an independent opinion** — before suggesting anything, critically evaluate whether the reviewer's feedback is correct, necessary, or a matter of preference:
   - If you **agree** with the feedback: explain why it's valid and propose concrete action items or code changes.
   - If you **disagree** (fully or partially): explicitly say so, explain your reasoning, and weigh the trade-offs. Do not default to agreeing with the reviewer just because they raised the comment.
   - If the feedback is **ambiguous or stylistic**: flag it as such and present both sides.
4. **Use the `AskUserQuestion` tool** after each thread to ask the user whether they want to act on it, discuss it further, or skip it. Do not proceed to the next thread until the user responds. Options should include: "Address it", "Discuss further", "Skip".

---

## Critical thinking requirement

Approach every thread with genuine skepticism in both directions — toward the reviewer and toward the existing code. Do not rubber-stamp suggestions. If a reviewer's comment would introduce complexity, break a pattern, or goes against established conventions in the codebase, say so clearly and justify your position.

---

## Output format

```
### Thread 1 — @reviewer_handle on `app/models/user.rb:42`

**Comment:** "This query could lead to N+1 issues."

**Code in question:**
  users.each { |u| u.posts.count }

**My assessment:** Agree. This is a classic N+1 pattern. The `posts` association
is loaded per-user without eager loading.

**Suggested fix:** Use `includes(:posts)` or `counter_cache` depending on
whether raw counts or full records are needed elsewhere.

---
Want to dive deeper into this thread, address it, or move on?
```
