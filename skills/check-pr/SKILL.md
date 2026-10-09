---
name: check-pr
description: Check a pull request, fix build failures or address review feedback, and push the changes.
disable-model-invocation: true
---

# Check a pull request

Usage:

```text
/check-pr <PR URL> fix build issues
/check-pr <PR URL> respond to feedback
/check-pr <PR URL> fix build issues in a subagent
/check-pr <PR URL> respond to feedback in a subagent
```

The PR URL is required. Treat the remaining text as the requested task. If it
does not clearly select **build issues** or **feedback**, ask the user which
mode to run.

## Dispatch

When either build or feedback mode says to use a subagent, delegate the complete
task.

Prefer the existing subagent that authored the PR when it is available. Reuse
that agent by sending it a follow-up turn so it retains the implementation
context, prior decisions, and repository state. Identify it from explicit
orchestrator state, a known agent ID, or the current agent tree; do not infer
agent identity from the PR author's platform account. If the author subagent is
unavailable, completed without a resumable conversation, or cannot accept more
work, start one general-purpose subagent instead.

Give the selected subagent:

- the exact user request and PR URL;
- the selected mode and the workflow below;
- responsibility for investigation, edits, validation, commit, push, and PR
  replies;
- the repository's applicable agent instructions and the original PR
  prompt/work item context;
- an explicit requirement to preserve unrelated work and avoid force-pushing.

Give the subagent ownership of that scope. Do not duplicate its investigation
while it runs. When it finishes, verify that its reported commit is on the PR
source branch. If it could not complete a step, finish only the missing step or
report the concrete blocker.

Otherwise, perform the workflow directly.

## Establish PR context

1. Parse the URL and use the authenticated platform tooling available for that
   host. Do not install tooling or change registries.
2. Record the repository, PR number, source repository and branch, target
   branch, current head SHA, and whether the source branch is writable.
3. Read the PR title, description, linked issue or work item, commits, changed
   files, status checks, and applicable repository guidance. These are the
   source of truth for the PR's original intent.
4. Preserve the current working tree. Use an isolated checkout when the current
   checkout is dirty, belongs to another repository, or is not safe to switch.
5. Fetch and check out the PR source branch without rewriting history. Before
   pushing, confirm the remote and branch still match the PR and account for a
   head change instead of overwriting it.

If the branch is not writable, complete the investigation and local fix, then
report the exact permission or fork limitation. Never substitute a different
branch and claim the PR was updated.

## Build issues

1. Inspect every currently failing or cancelled PR check and its logs. Separate
   code failures from infrastructure, authentication, policy, and flaky
   failures.
2. Reproduce the smallest relevant failure with existing repository commands
   when local validation is permitted. Follow repository build, dependency,
   and package-feed rules.
3. Fix the root cause and any tightly coupled failures. Keep the change scoped
   to restoring the intended PR behavior.
4. Run the smallest existing validation that covers the fix. Do not start an
   application or server unless the user explicitly requested it.
5. Commit and push normally to the PR source branch. Never force-push.

Completion means the fix commit is present on the PR source branch and the
relevant check is passing or rerunning. If no code change can fix the failure,
report the failing check, evidence, and required external action without making
an unrelated commit.

## Feedback

1. Collect all open, unresolved review threads and top-level review comments,
   including replies and referenced code. Ignore resolved threads unless a new
   unresolved reply makes them active again.
2. Compare each item with the current PR head and original intent. Classify it:
   - **Accept**: valid, compatible feedback. Bias toward this classification.
   - **Needs decision**: implementing it would remove, reverse, or materially
     narrow a requirement from the PR description, linked issue/work item, or
     original user prompt.
   - **No change**: already satisfied, obsolete after later commits, or
     disproved by current code and tests.
3. Implement every independent **Accept** item, including appropriate tests.
   Work through all such items even when one or more items need a decision.
4. For **No change** items, gather exact current-code or test evidence. Do not
   dismiss feedback based only on reviewer identity or assertion.
5. Validate, commit, and push the completed accepted fixes to the PR source
   branch without force-pushing.
6. Reply to each handled thread with the outcome and evidence. For accepted
   items, cite the pushed commit or changed code. For no-change items, explain
   the precise evidence. Resolve a thread only when the platform permits it and
   the concern is fully addressed.
7. After all independent work is pushed, ask one consolidated user question
   covering every **Needs decision** item. For each item, quote or link the
   feedback, identify the conflicting original requirement, and state the two
   concrete choices. Leave those threads unresolved until the user decides.

User input gates only the conflicting items. It must not delay investigation,
fixes, validation, pushes, or replies for independent items.

Completion means every open feedback item is either addressed on the pushed PR
head, answered with evidence, or presented to the user as a specific conflict
with the original intent.

## Final response

State the PR URL and pushed commit SHA, then summarize:

- build checks fixed or feedback items addressed;
- validation performed and current check state;
- unresolved external blockers or user decisions.

Never claim the PR was updated unless the commit is visible on its source
branch.
