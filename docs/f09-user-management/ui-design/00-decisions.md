# UI Design Decisions — F09 User Management

> Design status: UI DRAFT (for team review)
>
> Related: [UI design overview](README.md) · [Screen notes](01-screen-notes.md)

## Decisions

| ID | Decision | Why |
|---|---|---|
| D-UM-UI-01 | Users and Teams are two pages under the **Admin** module, not tabs of one page. | Different objects and actions; the existing template frame *Admin · Users & roles* already uses this sidebar. |
| D-UM-UI-02 | Users is a list with row selection and a **bulk action bar** (Enable, Disable, Change role, Add to team). | Admins change many technicians at once when teams change. |
| D-UM-UI-03 | A user is edited in a **drawer** over the list, not on its own page. | The form has five groups but no history; the list stays visible. Every change is still logged (*View log for this user*). |
| D-UM-UI-04 | The access scope is a **tree of teams and sub-groups** with checkboxes (a parent shows a partial mark). | Matches how F05 routes: group → sub-group → member. |
| D-UM-UI-05 | Disabling a user is a red zone at the bottom of the drawer that says how many open tickets go back to which queue. | The effect is immediate and affects tickets; the Admin must see it before confirming. |
| D-UM-UI-06 | Teams is a two-pane page: the structure tree on the left, the selected team on the right with its subcategories, AI agent, calendar, counters and members. A sub-group with no members shows a warning in the tree and in the detail. | An empty sub-group silently sends its tickets to the parent group; making it visible avoids surprise routing. |

## Assumptions to confirm with the F09 specification

| Assumption | Used on |
|---|---|
| Name, e-mail and department come from the directory and are read-only | SCR-UM-02 |
| Roles are Requester, Technician, Administrator (no Team Lead) | SCR-UM-01, SCR-UM-02 |
| Each team or sub-group declares the subcategories it handles (`team_categories`) and has an AI agent as first handler | SCR-UM-03 |
| Maximum open tickets is per member and used by the workload score | SCR-UM-02, SCR-UM-03 |
| Disabling signs the user out at once and returns open tickets to the team queue | SCR-UM-01, SCR-UM-02 |

## Not drawn yet

| Item | Reason |
|---|---|
| Roles & permissions page | The template frame *Admin · Users & roles* already shows the read-only role matrix. |
| Directory sync, Service accounts | Not in this assignment. |
| Add user, New team, Add sub-group forms | Same drawer pattern as SCR-UM-02. |
| Confirmation dialog for *Disable user* | To be drawn with the F09 acceptance criteria. |
