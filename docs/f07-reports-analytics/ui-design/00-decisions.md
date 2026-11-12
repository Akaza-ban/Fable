# UI Design Decisions — F07 Reports & Analytics

> Design status: TARGET V1 (draft for team review)
>
> Related: [UI design overview](README.md) · [Screen notes](01-screen-notes.md)

## Decisions

| ID | Decision | Why |
|---|---|---|
| D-BC-UI-01 | Reports sit under the **Dashboard** module tab of the top bar, with their own sidebar (Reports: Tickets, SLA, AI, Satisfaction; Settings: Report settings, Weekly digest). The four tabs are also shown on the page. | The template has no *Reports* tab; the Dashboard frame already lists reports in its sidebar. Keeps the top bar unchanged for the other branches. |
| D-BC-UI-02 | The AI tab opens with five tiles: M-04 (last full week), under observation, and the three routes of M-08 (A, B, C). | The Admin's weekly question (Flow 1) is answered before scrolling; the route split explains the M-04 figure. |
| D-BC-UI-03 | M-04 is a weekly line against a dashed **Target 50%** line, never a daily figure. | F05 / F07 agreement: self-resolution is weekly only (IA §8). |
| D-BC-UI-04 | Route C reasons are a ranked list where each reason is a link with a bar and its count. | The reason opens SCR-T-04 with the same filters (D-BC-IA-04); a ranked list reads faster than a chart. |
| D-BC-UI-05 | Report settings is one form in three cards (*How figures are computed*, *Export*, *Weekly digest*) with *Save changes* in the header. N is shown read-only with the tag "Read-only · from F03". | One audited change (`REPORT_CONFIG_UPDATED`); N belongs to F03 (BR-BC-05). |
| D-BC-UI-06 | The export modal shows the active filters, the format, and the row count **before** export; the Export button is disabled when the count is above the limit. | Users learn about `EXPORT_TOO_LARGE` before they wait for a file (Flow 3). |
| D-BC-UI-07 | **Plain language only.** The Reports are read by people who are not developers. Labels, messages and changes are written as short sentences ("Access given", "Not allowed by the directory"); internal codes (event names, error codes, metric ids, screen ids, category codes, raw ids, request / response bodies) are not shown. Codes stay in the data for filtering and in these docs for traceability; where a developer needs them, they sit behind a collapsed "Technical details" row. | Feedback on the first draft: the screens looked like developer logs. |

## Not drawn yet

| Item | Reason |
|---|---|
| SCR-BC-01 Tickets, SCR-BC-02 SLA, SCR-BC-04 Satisfaction | Not in this assignment; they reuse the filter bar, tiles and table of SCR-BC-03. |
| Technician view (scope only, no AI tab) | Same layout without the AI tab and without *Export* (D-BC-IA-03). |
| EML-BC-01 weekly digest e-mail | E-mail template, not a page. |
| Responsive widths (< 1200 px) | Frames are drawn at 1280 px; the rules are in [UI states §4](../information-architecture/03-ui-states-and-accessibility.md#4-responsive-layout). |
