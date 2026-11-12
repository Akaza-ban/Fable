# UI Design Decisions — F08 Activity Log

> Design status: TARGET V1 (draft for team review)
>
> Related: [UI design overview](README.md) · [Screen notes](01-screen-notes.md)

## Decisions

| ID | Decision | Why |
|---|---|---|
| D-NK-UI-01 | The activity log is a table with a **side panel showing what changed** for the selected row (selected row: blue background and a 3 px left bar). | Matches UI states §4 (≥ 1200 px); the Admin compares entries without leaving the list. |
| D-NK-UI-02 | A grey pill "🔒 Append-only · no edit or delete" sits in the page header, and the panel ends with "This entry cannot be edited or deleted". | Makes D-NK-IA-05 visible instead of only hiding controls. |
| D-NK-UI-03 | Corrections are ordinary rows with the action `ENTRY_CORRECTION` and the text "Corrects entry #…" linking to the old entry. | The old entry stays unchanged (UI states §1, *Correction*). |
| D-NK-UI-04 | Workflow runs opens with a danger banner when one connector causes most failures, with *Filter these runs*. | Flow 3: the Admin finds the broken connector in one click. |
| D-NK-UI-05 | Run detail tells the story as a timeline of sentences ("Tried…", "Refused…", "Handed to a technician") followed by **What to do next**; request and response are only in a collapsed "Technical details for the IT developer team" row. | The Admin needs the reason and the next step, not the HTTP exchange. |
| D-NK-UI-06 | Run detail has no *Retry* or *Resolved manually* button; a note points to the ticket's Automation tab. | F05 owns those actions (D-NK-IA-03, SCR-WF-20 / 21). |
| D-NK-UI-07 | **Plain language only.** The Audit pages are read by people who are not developers. Labels, messages and changes are written as short sentences ("Access given", "Not allowed by the directory"); internal codes (event names, error codes, metric ids, screen ids, category codes, raw ids, request / response bodies) are not shown. Codes stay in the data for filtering and in these docs for traceability; where a developer needs them, they sit behind a collapsed "Technical details" row. | Feedback on the first draft: the screens looked like developer logs. |

## Not drawn yet

| Item | Reason |
|---|---|
| SEC-NK-01 ticket history and SEC-NK-02 approval chain | Sections of SCR-T-02, drawn with the F03 ticket page. |
| Activity log for widths < 1200 px (diff below the row, cards on mobile) | Frames are drawn at 1280 px; rules in [UI states §4](../information-architecture/03-ui-states-and-accessibility.md#4-responsive-layout). |
| Loading skeletons as separate frames | Described in the screen notes; the Activity log frame shows a *States* card with EMPTY, LOADING and ERROR. |
