# UI States and Accessibility — F08 Activity Log

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [User flows](02-user-flows.md)

## 1. Who did it (D-NK-IA-06)

Every entry carries one actor marker, with an icon and text, never color alone. The markers match F04 [UI states](../../f04-ai-ticket-processing/information-architecture/03-ui-states-and-accessibility.md).

| Marker | `actor_type` | Shown as |
|---|---|---|
| Person | `USER` | Display name and role ("Hai · Technician") |
| AI | `AI` | **AI** badge and the domain agent ("AI · Helpdesk agent"); requester sees "IT assistant" |
| Automation | `AUTOMATION` | Gear icon and the evidence ("Automation · rule *Accounting access*", "Automation · run R-882") |
| Correction | any | "Corrects entry #…" with a link; the old entry stays unchanged |

## 2. State contract

| State | When | What the user sees | Where |
|---|---|---|---|
| `LOADING` | Query running | Skeleton rows; the rest of SCR-T-02 never waits | All |
| `READY` | Entries returned | The timeline or table | All |
| `EMPTY` | No entry matches | "No events for these filters" | SEC-NK-01, SCR-NK-02, SCR-NK-04 |
| `SUMMARY_ONLY` | Role sees a summary (requester: status line; technician: steps, results and reason) | No "show details" or *View AI decision* control | SEC-NK-02 |
| `MASKED` | A value was masked | `***` with tooltip "Hidden for security" | Diff panel, attempts |
| `FAILED_STEP` | Approved but execution failed, verification mismatch | Red step with icon and "Approved — execution failed", assignee | SEC-NK-02 |
| `FORBIDDEN` | Role or scope not allowed | SCR-90; never says whether the ticket exists | All |
| `ERROR` | `422 INVALID_DATE_RANGE`, service down | Message under the filters; previous results kept | SCR-NK-02, SCR-NK-04 |

## 3. Status chips for workflow runs

| Status | Chip |
|---|---|
| `QUEUED`, `RUNNING`, `RETRY_WAIT`, `VERIFYING` | Clock icon · "In progress" (exact status in tooltip) |
| `SUCCEEDED` | Check icon · "Succeeded" |
| `FAILED` | Cross icon · "Failed" + error type |
| `MANUALLY_RESOLVED` | Person icon · "Resolved manually" |

## 4. Responsive layout

| Width | Layout |
|---|---|
| ≥ 1200 px | SCR-NK-02: table with diff side panel; SCR-T-02 `#history`: timeline full width |
| 768–1199 px | Diff panel opens below the selected row |
| < 768 px | Table becomes cards; filters collapse into a *Filters* button; the page never scrolls sideways |

## 5. Accessibility (WCAG 2.1 AA)

- The timeline is an ordered list (`<ol>`) with time as `<time datetime>`.
- The approval stepper uses text for each step result ("Passed", "Failed"), not only ✓ / ✗.
- The diff panel labels removed and added values in text, not only by color; contrast ≥ 4.5:1.
- Filters, rows and evidence links are reachable by keyboard; the diff panel returns focus to its row on close (Esc).
