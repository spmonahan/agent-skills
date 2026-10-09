---
name: status-pls
description: Show or schedule a compact status table for the current work.
disable-model-invocation: true
---

# Status please

Usage:

```text
/status-pls
/status-pls every 15 minutes
/status-pls remove schedule
```

Interpret arguments as one of three modes:

- no arguments: report status now;
- a recurring cadence: manage the status schedule;
- `stop`, `unschedule`, `remove schedule`, or equivalent wording: remove the
  status schedule.

This skill reports only. Do not pause, redirect, or duplicate the work being
reported.

## Report status now

Build the report from the current session state. Use the current task,
completed and in-progress steps, concrete blockers, and the latest known
validation or delivery state. Do not start a new investigation merely to make
the report more detailed.

When the current agent is orchestrating subagents, inspect the current agent
tree and report the relevant work owned by each subagent. Use the latest
available subagent state or result. Include the orchestrator's own work only
when it has a distinct workstream to report.

Return only a compact Markdown table. Use these default columns:

| PR or Work item ID | Short description | Status |
|---|---|---|

Adapt the table to the work:

- Omit `PR or Work item ID` when no row has one.
- Add `Agent` when separate agents own separate workstreams.
- Add another column only when it makes the status materially easier to scan.
- Use one row per distinct workstream, PR, work item, or subagent assignment.
- Keep `Short description` to a few words on one physical line.
- Keep `Status` to one to three short sentences. Use `<br>` between sentences
  only when it improves scanning.
- State the current phase, meaningful progress, and blocker or next concrete
  step. Prefer observable facts over estimates.
- Use `Blocked` plainly when progress requires user input or an external
  action.

If no work is active, return a one-row table stating that there is no active
work. Do not add prose before or after the table.

## Manage the schedule

A session may have at most one recurring status schedule.

1. List the session's active schedules and identify schedules whose prompt
   invokes `/status-pls`.
2. If no status schedule exists, create one using the requested cadence. The
   scheduled prompt must be exactly `/status-pls` so each run reports current
   state rather than changing the schedule.
3. If one already exists, ask the user to choose:
   - **Keep existing schedule**: leave it unchanged and create nothing.
   - **Replace with new schedule**: stop the existing schedule, then create the
     requested schedule.
4. If historical state contains multiple status schedules, treat the oldest as
   the existing schedule and stop the duplicates before asking. Keeping
   preserves that one schedule; replacing stops it before creating one
   schedule.

Use a relative interval for cadences such as `every 15 minutes` and a cron
expression for calendar cadences. If the cadence is ambiguous or cannot be
represented by the scheduling tool, ask the user for a specific recurring
cadence before creating anything.

Completion means exactly one status schedule exists at the requested cadence,
or the user chose to keep the existing schedule.

## Remove the schedule

1. List the session's active schedules and identify every schedule whose prompt
   invokes `/status-pls`.
2. Stop every identified status schedule.
3. If none exists, state that no status schedule was active.

Completion means no active status schedule remains for this session.
