# Screen Notes and States — F09 User Management

> Design status: UI DRAFT (for team review)
>
> Related: [UI design overview](README.md) · [Decisions](00-decisions.md) · [Master IA §2 states](../../information-architecture.md#2-standard-screen-hierarchy--component-states)

Each screen lists its purpose, its layout, the elements worth noting and its states. F09 has no state contract of its own yet, so the **Source** column names the [standard states of the master IA](../../information-architecture.md#2-standard-screen-hierarchy--component-states) or the rule of another branch behind it.

## People

### SCR-UM-01 Users — `/admin/users` · Admin · [PNG](../../image/admin/17_SCR-UM-01_Users.png)

Purpose: find users, see their role, access scope and availability, and enable or disable them alone or in bulk.

**Layout:** page header (*Sync now*, *Add user*); filter bar; bulk action bar (when rows are selected); table with paging; note.

![SCR-UM-01](../../image/admin/17_SCR-UM-01_Users.png)

| # | Element | Note |
|---|---|---|
| 1 | Header meta | "Copied from the company directory · last updated today 10:02". |
| 2 | Filters | Name or e-mail, role, department, team, status. In the PNG: *Role: Technician*. |
| 3 | Bulk action bar | "2 selected" with Enable, Disable, Change role, Add to team, *Clear selection*. Selected rows are blue. |
| 4 | Table | Checkbox, name + e-mail, department, role badge, teams, availability ("Available · 7 tickets" or "Away · on leave"), status badge, last sign-in. |
| 5 | Row actions | *Edit* (opens SCR-UM-02), *Disable* / *Enable*, *Log* (SCR-NK-02 filtered on the user). |
| 6 | Note | Disabling signs the user out and returns open tickets to the team queue; *Add user* is for staff not in the directory. |

| State | Source | Behavior |
|---|---|---|
| default | Initial | As in the PNG, two rows selected. |
| loading | Loading | Skeleton rows; filters stay usable. |
| empty | Empty | "No users match these filters" with *Clear filters*. |
| sync failed | Error / Network fail | Banner "Directory sync failed at 10:02" with *Retry*; the list shows the last synced data. |
| saved | Success | Toast "2 users disabled" for 5 s. |
| not admin | 403 | SCR-90. |

### SCR-UM-02 Edit user (drawer) — on `/admin/users` · Admin · [PNG](../../image/admin/18_SCR-UM-02_Edit-user-drawer.png)

Purpose: change a user's role, access scope and assignment settings, or disable the account.

**Layout:** drawer 480 px from the right over a dimmed list; header with name and e-mail; five groups; sticky footer.

![SCR-UM-02](../../image/admin/18_SCR-UM-02_Edit-user-drawer.png)

| # | Element | Note |
|---|---|---|
| 1 | Profile | Display name and department read-only (from the directory). |
| 2 | Role | Three radio cards: Requester, Technician, Administrator, each with one line on what it allows. |
| 3 | Teams they work for (access scope) | Tree of teams and sub-groups with checkboxes; a parent with some children checked shows a partial mark. Required for Technician. |
| 4 | Assignment | Toggle *Can receive new tickets automatically* and *Most tickets at once*. |
| 5 | Account status | Red zone: "Disabling signs Minh out now and returns 7 open tickets to Network L1 / L2 queues", *Disable user*. |
| 6 | Footer | *View log for this user →*, *Cancel*, *Save changes*. |

| State | Source | Behavior |
|---|---|---|
| default | Initial | As in the PNG. |
| validation | Validation | Technician with no team: "Choose at least one team" under the tree; *Save* stays enabled and shows the error on click. |
| saving | Saving / Processing | *Save changes* disabled, "Saving…". |
| saved | Success | Drawer closes; toast "Minh Tran updated"; the row shows the new values. |
| conflict | Error | "This user was changed by someone else. Reload to see the latest values." |

Accessibility: focus moves into the drawer and is trapped; Esc closes it and returns focus to the row's *Edit*.

## Teams

### SCR-UM-03 Teams & sub-groups — `/admin/teams` · Admin · [PNG](../../image/admin/19_SCR-UM-03_Teams.png)

Purpose: maintain the structure that routing uses: teams, sub-groups, the subcategories each handles, and their members (F05 UC-WF-04, UC-WF-05).

**Layout:** page header (*New team*); two panes: *Structure* tree (290 px) and the selected team.

![SCR-UM-03](../../image/admin/19_SCR-UM-03_Teams.png)

| # | Element | Note |
|---|---|---|
| 1 | Header text | "New tickets go to the right team first, then to the least busy person who is available." |
| 2 | Structure tree | Teams with member count; sub-groups indented; the selected node has a blue bar. *Triage queue* "catches unclear tickets". |
| 3 | Empty sub-group | Badge *No members* in the tree (Wi-Fi). |
| 4 | Team header | Name, "Part of Network", "Level 1"; *View log*, *Add sub-group*, *Edit*. |
| 5 | Properties | "Handles requests about" VPN, Office network, Website names (DNS) — category names, never codes; "AI assistant (tries first)"; working hours. |
| 6 | Counters | Available now, open tickets ("5 not yet picked up"), how busy the team is ("Normal") instead of a numeric workload score. |
| 7 | Members | Add by name; table with availability toggle ("Available" / "Away · on leave"), tickets now (bar, red above 85%), busy level (Light / Normal / High), *Remove*. |
| 8 | Warning | "Wi-Fi has no members yet. Wi-Fi tickets go to Network L1 until someone is added." |

| State | Source | Behavior |
|---|---|---|
| default | Initial | Network L1 selected. |
| empty team selected | Empty | "No members yet" with *Add member*, and the warning of #8. |
| subcategory taken | Validation | Adding a request type already handled by another sub-group: "VPN is handled by Network L2. Move it here?" |
| saving | Saving / Processing | Member row shows a spinner while the change is saved. |
| not admin | 403 | SCR-90. |
