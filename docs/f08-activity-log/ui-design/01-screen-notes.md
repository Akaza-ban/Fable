# Screen Notes and States — F08 Activity Log

> Design status: TARGET V1 (draft for team review)
>
> Related: [UI design overview](README.md) · [Decisions](00-decisions.md) · [UI states (IA)](../information-architecture/03-ui-states-and-accessibility.md)

Each screen lists its purpose, its layout, the elements worth noting and its states. The **Source** column names the row of the [state contract](../information-architecture/03-ui-states-and-accessibility.md#2-state-contract), or the rule or flow behind it. Sample data follows the [user flows](../information-architecture/02-user-flows.md): Flow 2 (wrong grant on `\\fs01\accounting`) and Flow 3 (broken AD connector).

## Audit

### SCR-NK-02 Activity log — `/audit/activity-log` · Admin · [PNG](../../image/admin/14_SCR-NK-02_Activity-log.png)

Purpose: find who did what, when and why across tickets, rules and settings, and see the before / after of each entry (US-NK-04, US-NK-10).

**Layout:** page header with the append-only pill; filter bar in two lines; table (left) and diff panel (right, 350 px); a *States* card at the bottom.

![SCR-NK-02](../../image/admin/14_SCR-NK-02_Activity-log.png)

| # | Element | Note |
|---|---|---|
| 1 | Filters, line 1 | Target (free text), actor type, actor, action, period. Active filters are blue. |
| 2 | Filters, line 2 | Evidence id, agent, model version, prompt version (US-NK-10), *Clear all*. All filters are in the URL (`?actor=&actor_type=&action=&target=&evidence=&agent=&prompt_version=&from=&to=`). |
| 3 | Table | Time (monospace), actor marker, action code, target (link: ticket → SCR-T-02, setting → its page), summary. Times in Asia/Ho_Chi_Minh (UTC+7), stated in the footer. |
| 4 | Actor marker | Person / AI + domain agent + prompt version / Automation + rule or run (D-NK-IA-06). |
| 5 | Correction row | `ENTRY_CORRECTION` — "Corrects entry #88207"; the old entry is not changed. |
| 6 | Diff panel | Action and entry id; actor, ticket, target; *Before → after* with the words *Removed* / *Added*; masked values as `***` "(Hidden for security)". |
| 7 | Evidence | `run_id` → SCR-NK-05, `decision_id` → SCR-WF-31, `analysis_id`, model / prompt version. |
| 8 | No edit or delete | Header pill and the note at the bottom of the panel (D-NK-IA-05). |

| State | Source | Behavior |
|---|---|---|
| default | `READY` | As in the PNG; first row selected. |
| loading | `LOADING` | Skeleton rows; filters stay usable. |
| empty | `EMPTY` | "No events for these filters. Clear a filter or widen the period." |
| masked | `MASKED` | `***` with the tooltip "Hidden for security", in the diff panel. |
| invalid period | `ERROR` `422 INVALID_DATE_RANGE` | Message under the filters; previous results kept. |
| not admin | `FORBIDDEN` | SCR-90; the Audit group is not in the sidebar. |

Accessibility: rows and evidence links are reachable by keyboard; Esc closes the panel and returns focus to its row; removed / added values are labelled in text, not only by color.

### SCR-NK-04 Workflow runs — `/audit/workflow-runs` · Admin · [PNG](../../image/admin/15_SCR-NK-04_Workflow-runs.png)

Purpose: find failing automated actions across tickets and the connector behind them (US-NK-06, US-NK-07).

**Layout:** page header with *Export CSV*; alert banner; four counters; filter bar; table with paging; legend of the status chips.

![SCR-NK-04](../../image/admin/15_SCR-NK-04_Workflow-runs.png)

| # | Element | Note |
|---|---|---|
| 1 | Alert banner | "37 runs failed since 08:00 — all on connector AD-Directory with error PERMANENT_4XX" with *Filter these runs* (Flow 3, step 2). |
| 2 | Counters | In progress, Succeeded, Failed, Resolved manually — counters by status. |
| 3 | Filters | Ticket or run id, status, error type, operation, connector, period (`?status=&error=&operation=&connector=&ticket=&from=&to=`). |
| 4 | Table | Run id → SCR-NK-05, ticket → SCR-T-02, operation, status chip, error type, attempts, started, duration. |
| 5 | Status chips | In progress groups `QUEUED`, `RUNNING`, `RETRY_WAIT`, `VERIFYING`; the exact status is in the tooltip. |

| State | Source | Behavior |
|---|---|---|
| default | `READY` | Filtered on failed runs of AD-Directory today. |
| loading | `LOADING` | Counters and rows as skeletons. |
| empty | `EMPTY` | "No runs for these filters." |
| no failures | `READY` | The alert banner is not shown. |
| invalid period | `ERROR` | Message under the filters; previous results kept. |
| not admin | `FORBIDDEN` | SCR-90; technicians see runs in the ticket's Automation tab. |

### SCR-NK-05 Workflow run detail — `/audit/workflow-runs/:id` · Admin · [PNG](../../image/admin/16_SCR-NK-05_Workflow-run-detail.png)

Purpose: see why one run failed: each attempt with the masked request and response (US-NK-06, US-NK-09).

**Layout:** back link *‹ Workflow runs*; header with run id, operation, *Open ticket*; summary card; failure banner; attempts card; note on actions.

![SCR-NK-05](../../image/admin/16_SCR-NK-05_Workflow-run-detail.png)

| # | Element | Note |
|---|---|---|
| 1 | Summary | Ticket, requester, connector, started by (Automation · AI decision → SCR-WF-31), operation, who got the ticket after the failure. Status chip and error type in the card header. |
| 2 | Failure banner | "Approved — execution failed", the reason, and that the ticket went to a technician (`FAILED_STEP`). |
| 3 | Attempts | One card per attempt: time, duration, result code (warning for transient, danger for permanent); request and response side by side; tokens and accounts as `***`. |
| 4 | Collapsed attempt | Middle attempts collapse to one line. |
| 5 | Action note | *Retry* and *Resolved manually* are on the ticket's Automation tab (F05 SCR-WF-20, 21); each retry adds attempts here. |

| State | Source | Behavior |
|---|---|---|
| failed | `FAILED_STEP` | As in the PNG. |
| succeeded | `READY` | No failure banner; status chip ✓ Succeeded; attempts show the verification step. |
| in progress | `READY` | Chip ◷ In progress; the last attempt shows "waiting for retry" with the next time. |
| audit unavailable | AC-NK-09.1, BR-NK-06 | Chip ✕ Failed with `AUDIT_UNAVAILABLE`; banner "The 'execution started' entry could not be written, so the connector was not called. The ticket went to a technician." No attempt cards. |
| not admin | `FORBIDDEN` | SCR-90. |
