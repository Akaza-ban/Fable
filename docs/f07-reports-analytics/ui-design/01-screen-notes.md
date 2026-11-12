# Screen Notes and States — F07 Reports & Analytics

> Design status: TARGET V1 (draft for team review)
>
> Related: [UI design overview](README.md) · [Decisions](00-decisions.md) · [UI states (IA)](../information-architecture/03-ui-states-and-accessibility.md)

Each screen lists its purpose, its layout, the elements worth noting and its states. The **Source** column names the row of the [data state contract](../information-architecture/03-ui-states-and-accessibility.md#1-data-state-contract), or the rule or acceptance criterion behind it. Sample data: last 12 weeks to 9 Nov 2026, about 8,000 tickets a week.

## Reports

### SCR-BC-03 Reports › AI — `/analytics/ai` · Admin · [PNG](../../image/admin/10_SCR-BC-03_Reports-AI.png)

Purpose: show whether the AI is on track for the self-resolution target, why tickets still go to a person, and whether the AI output is good (US-BC-04, US-BC-08, US-BC-09).

**Layout:** page header (title "Reports", subtitle "Click any number to see the tickets behind it", *Export*); tabs Tickets / SLA / **AI** / Satisfaction; filter bar; five tiles; two rows of two cards; the Quality card full width.

![SCR-BC-03](../../image/admin/10_SCR-BC-03_Reports-AI.png)

| # | Element | Note |
|---|---|---|
| 1 | Filter bar | Period (default *Last 12 weeks*, highlighted when not the default), type, domain, group, priority, channel, handler. The line under it says, in words, which settings are used: "Vietnam time · a ticket counts as solved by the AI only if it is not reopened within 7 days · target 50%". Filters live in the URL (D-BC-IA-01). |
| 2 | Tile **Solved by AI alone** (M-04) | 41.2% with "3,280 of 7,962 · last full week": a rate always comes with its count (UI states §3). |
| 3 | Tile **Not counted yet** | 312 tickets closed in the last 7 days; not counted in M-04 yet (`UNDER_OBSERVATION`). |
| 4 | Tiles **Done automatically**, **AI asked for details**, **Sent to a technician** | M-08 routes A, B, C with counts. |
| 5 | Chart **Tickets solved by AI alone, week by week** | Weekly purple line, dashed red **Target 50%** line, **AI** badge, *View as table*, and a summary in plain words under the chart ("In short: …"). |
| 6 | Card **How the AI solved them** | "Did the change itself", "Answered with a help article", "User found an answer before sending", "After the AI asked for details"; then the split by AI assistant (domain agent). |
| 7 | Card **Why tickets still went to a technician** | Ranked reasons in plain words (e.g. "Requested access is not on the approved list"); each reason opens the ticket list SCR-T-04 with the same filters (Flow 1, step 4). |
| 8 | Card **Why the AI handed tickets over** (M-10) | Each `escalationClass` shown as a sentence: "AI could not fix it" (`AGENT_CANNOT_RESOLVE`), "User said it was not fixed", "AI was not sure enough", "Automatic change failed", "AI was offline". |
| 9 | Card **How accurate the AI is** | Filters *AI version* and *Compare with* (model / prompt version, US-BC-09); tiles "Wrong category, fixed by technicians" (M-05), "Closed by AI, then reopened" (M-06), "AI answer did not help" (M-11); table of recently corrected categories using category names ("Printer" → "Docking & monitors"), never codes. |

| State | Source | Behavior |
|---|---|---|
| default | `READY` | As in the PNG. |
| loading | `LOADING` | Each tile and chart shows its own skeleton; the filter bar stays usable; previous figures stay greyed while filters change. |
| not enough data | `NOT_ENOUGH_DATA` (BR-BC-01) | M-04 shows "Not enough data for this period", never `0%`. |
| AI data unavailable | `UNAVAILABLE` | "AI data unavailable" on the affected tiles only; the others stay `READY`. |
| invalid period | `ERROR` `422 INVALID_DATE_RANGE` | Message under the period filter; previous figures kept. |
| technician | `FORBIDDEN` (D-BC-IA-03) | The AI tab is not shown; the URL opens SCR-90. |

### SCR-BC-05 Report settings — `/analytics/settings` · Admin · [PNG](../../image/admin/11_SCR-BC-05_Report-settings.png)

Purpose: set the parameters that change how figures are computed for everyone, and the weekly digest (US-BC-07, US-BC-11).

**Layout:** page header with *Cancel* and *Save changes*; info banner; three cards with label / help text on the left and the control on the right.

![SCR-BC-05](../../image/admin/11_SCR-BC-05_Report-settings.png)

| # | Element | Note |
|---|---|---|
| 1 | Header note | "Changes affect every report. Each change is saved in the activity log." (logged as `REPORT_CONFIG_UPDATED`, BR-BC-05). |
| 2 | Info banner | "Response and resolution targets are not set here" with the link *Open SLA policies* (SCR-WF-02). |
| 3 | Card header meta | "Last changed by Thu Le · 3 Nov 2026 09:12". |
| 4 | **Wait before counting as solved** (N) | Read-only input, tag "Set in ticket settings" (F03 reopen window). |
| 5 | **Target for tickets solved by AI**, **Minimum share of users who rate**, **Maximum report period** | Number inputs with unit, allowed range and a help sentence. |
| 6 | Export row limit, formats | "Bigger exports are stopped and the user is asked to choose a shorter period" (`EXPORT_TOO_LARGE`); formats CSV and Excel. |
| 7 | **Weekly summary e-mail** (EML-BC-01) | Schedule badge "Every Monday at 8:00", recipients as removable chips, *Send me a test* with the last test time. |

| State | Source | Behavior |
|---|---|---|
| default | Initial | As in the PNG, apart from the error shown on one field. |
| validation error | Validation (master IA §2) | Red border and "Enter a value between 1 and 100." under the field; checked on blur. Shown in the PNG on *Low-response threshold*. |
| saving | Saving / Processing | *Save changes* disabled with "Saving…". |
| saved | Success | Toast "Report settings saved" for 5 s; the meta line shows the new author and time. |
| technician | `FORBIDDEN` | Page not in the sidebar; URL opens SCR-90. |

### SCR-BC-10 Export (modal) — on any tab · Admin · [PNG over the page](../../image/admin/12_SCR-BC-10_Export.png) · [PNG states](../../image/admin/13_SCR-BC-10_Export-states.png)

Purpose: download the figures of the current tab with its filters as CSV or XLSX (US-BC-06).

![SCR-BC-10 over the AI tab](../../image/admin/12_SCR-BC-10_Export.png)

![SCR-BC-10 states](../../image/admin/13_SCR-BC-10_Export-states.png)

| # | Element | Note |
|---|---|---|
| 1 | Filters | The filters of the tab, read-only; the export uses exactly these. |
| 2 | Format | Two radio cards, "CSV file" and "Excel file". |
| 3 | Rows to export | Count before export; red when above the limit. |
| 4 | Note | "Internal notes, comments and passwords are never included. Each export is saved in the activity log." (logged as `REPORT_EXPORTED`). |

| State | Source | Behavior |
|---|---|---|
| Ready | Initial | 4,812 rows, *Download Excel file* enabled. |
| Too large | `EXPORT_TOO_LARGE` (Flow 3, branch 3) | 60,214 rows; banner "Too many rows to export" asks to narrow the period or add a filter; the download button is disabled. |
| Cannot record | `AUDIT_UNAVAILABLE` (Flow 3, branch 4) | Banner "Export not possible right now": no file is created, try again in a few minutes; the primary button becomes *Try again*. |
| exporting | Saving / Processing | Button "Exporting…" disabled; the modal stays open until the file starts downloading. |

Accessibility: focus is trapped in the modal, Esc closes it, and focus returns to *Export* (UI states §5).
