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
| 1 | Filters, line 1 | Search (free text, e.g. "Accounting shared folder"), *Done by* (person / AI / automatic rule), person, action, period. Active filters are blue. |
| 2 | Filters, line 2 | Ticket number, AI assistant, AI version (US-NK-10), *Clear all*. All filters are in the URL (`?actor=&actor_type=&action=&target=&evidence=&agent=&prompt_version=&from=&to=`). |
| 3 | Table | Time, *Done by*, action as words ("Access given", "Sent to a technician"; the code is kept for filtering only), *About* (link: ticket → SCR-T-02, setting → its page), details as a sentence ("Lan Nguyen can now open the folder (read only)"). Times in Asia/Ho_Chi_Minh (UTC+7), stated in the footer. |
| 4 | Done by | Person with role / **AI** badge + assistant name + "decided on its own" / Automation + rule name (D-NK-IA-06). |
| 5 | Correction row | "Correction added" — "Fixes an earlier entry: wrong person named"; the old entry is not changed. |
| 6 | Side panel | Action and time; done by, ticket, about; **What changed** written as sentences ("Before — Access: none", "After — Access: can open, cannot edit", "Added to group: Finance – Accounting (read only)"); passwords shown as "hidden for security". |
| 7 | Related | *Open the automatic action* → SCR-NK-05, *See why the AI approved* → SCR-WF-31, AI version. Raw ids (`run_id`, `decision_id`) are not shown. |
| 8 | No edit or delete | Header pill and the note "Entries cannot be changed or deleted. To fix a mistake, an administrator adds a correction." (D-NK-IA-05). |

| State | Source | Behavior |
|---|---|---|
| default | `READY` | As in the PNG; first row selected. |
| loading | `LOADING` | Skeleton rows; filters stay usable. |
| empty | `EMPTY` | "Nothing found — No entries match. Remove a filter or choose a longer period." |
| masked | `MASKED` | "hidden for security" in place of the value. |
| invalid period | `ERROR` `422 INVALID_DATE_RANGE` | "Period too long — Choose a period of one year or less." Previous results kept. |
| not admin | `FORBIDDEN` | SCR-90; the Audit group is not in the sidebar. |

Accessibility: rows and evidence links are reachable by keyboard; Esc closes the panel and returns focus to its row; removed / added values are labelled in text, not only by color.

### SCR-NK-04 Workflow runs (shown as "Automatic actions") — `/audit/workflow-runs` · Admin · [PNG](../../image/admin/15_SCR-NK-04_Workflow-runs.png)

Purpose: find failing automated actions across tickets and the connector behind them (US-NK-06, US-NK-07).

**Layout:** page header "Automatic actions" with *Export*; alert banner; four counters; filter bar; table with paging; legend of the status chips.

![SCR-NK-04](../../image/admin/15_SCR-NK-04_Workflow-runs.png)

| # | Element | Note |
|---|---|---|
| 1 | Alert banner | "37 automatic actions failed since 08:00. The company directory refused them: the service desk account is not allowed to make these changes." with *Show these* (Flow 3, step 2). |
| 2 | Counters | In progress, Succeeded, Failed, Resolved manually — counters by status. |
| 3 | Filters | Ticket number, status, reason ("Not allowed"), action, system ("Company directory"), period (`?status=&error=&operation=&connector=&ticket=&from=&to=`). |
| 4 | Table | No. (#903 → SCR-NK-05), ticket → SCR-T-02, action in words ("Add to group", "Reset MFA", "Give folder access"), status chip, reason in words ("Not allowed"), tries, started, took. |
| 5 | Status chips | In progress groups `QUEUED`, `RUNNING`, `RETRY_WAIT`, `VERIFYING`; hovering shows the step in words. |

| State | Source | Behavior |
|---|---|---|
| default | `READY` | Filtered on failed runs of AD-Directory today. |
| loading | `LOADING` | Counters and rows as skeletons. |
| empty | `EMPTY` | "No runs for these filters." |
| no failures | `READY` | The alert banner is not shown. |
| invalid period | `ERROR` | Message under the filters; previous results kept. |
| not admin | `FORBIDDEN` | SCR-90; technicians see runs in the ticket's Automation tab. |

### SCR-NK-05 Workflow run detail (shown as an automatic action) — `/audit/workflow-runs/:id` · Admin · [PNG](../../image/admin/16_SCR-NK-05_Workflow-run-detail.png)

Purpose: understand in plain words why one automatic action failed and what to do next (US-NK-06, US-NK-09).

**Layout:** back link *‹ Automatic actions*; title as a sentence ("Add Bao Vu to the Marketing shared folder"), *Open ticket*; summary card; failure banner; **What happened** card; note on actions.

![SCR-NK-05](../../image/admin/16_SCR-NK-05_Workflow-run-detail.png)

| # | Element | Note |
|---|---|---|
| 1 | Summary | Ticket, requester, system ("Company directory"), why it ran (*The AI approved the request* → SCR-WF-31), change in words, who handles it now. Status chip and the reason ("Not allowed by the directory") in the card header. |
| 2 | Failure banner | "Approved, but the change could not be made" with the reason in one sentence and who got the ticket (`FAILED_STEP`). |
| 3 | What happened | A timeline in plain words: "Tried to add Bao Vu to the group — the company directory was busy…", "Tried again", "Refused — the service desk account is not allowed…", "Handed to a technician". Dot color: amber = temporary problem, red = refused, blue = handed over. |
| 4 | What to do next | Numbered steps: ask the directory administrator for permission, then *Try again* on the ticket. |
| 5 | Technical details | Collapsed row "Technical details for the IT developer team"; only there are the masked request and response shown. *Try again* and *Resolved manually* stay on the ticket's Automation tab (F05 SCR-WF-20, 21). |

| State | Source | Behavior |
|---|---|---|
| failed | `FAILED_STEP` | As in the PNG. |
| succeeded | `READY` | No failure banner; status chip ✓ Succeeded; the timeline ends with "Checked: the change is in place". |
| in progress | `READY` | Chip ◷ In progress; the last line says "Will try again at 09:55". |
| audit unavailable | AC-NK-09.1, BR-NK-06 (`AUDIT_UNAVAILABLE`) | Chip ✕ Failed; banner "The change was not made because it could not be recorded in the activity log. The ticket went to a technician." |
| not admin | `FORBIDDEN` | SCR-90. |
