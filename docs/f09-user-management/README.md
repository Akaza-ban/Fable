# F09 User Management

> Design status: UI DRAFT ONLY — specification not written
>
> Implementation status: NOT STARTED

This folder will specify the **User Management** branch (F09): the user directory, roles and access scope, and the support teams with their sub-groups and members. Other branches already depend on it: F05 routes a ticket to the **group** of its domain and the **sub-group** of its subcategory and picks a member who is `ON` and least loaded (F05 UC-WF-04, UC-WF-05, BR-WF-09); F08 logs every change to users and teams.

The use cases, user stories, acceptance criteria and information architecture are **not written yet**. Only the UI draft below exists. It was derived from the [master information architecture](../information-architecture.md) (`/admin/users-rbac`, role matrix), the governance screen split (*Users and teams: add, edit, enable / disable users; roles and access scope; teams, sub-groups and members*) and the group / sub-group rules of [F05](../f05-workflow-automation/02-use-cases.md). It must be checked against the F09 specification when that is written.

## Documents

| File | Content |
|---|---|
| [ui-design/](ui-design/README.md) | UI draft: Users, Edit user drawer, Teams & sub-groups (SCR-UM-01 … 03), screen notes and decisions |

## Open questions

| ID | Question | Proposed answer |
|---|---|---|
| Q-UM-01 | Are requesters created only by directory sync (Azure AD / Google), or can the Admin add any user? | Sync creates everyone; *Add user* is for staff not in the directory. |
| Q-UM-02 | Is the access scope of a technician a list of teams and sub-groups, or of domains? | Teams and sub-groups (matches F05 routing). |
| Q-UM-03 | Who sets a technician `ON` / `OFF`: the technician (top bar, F05 US-WF-11), the Admin, or both? | Both; the Admin sees the reason. |
| Q-UM-04 | What happens to open tickets when a user is disabled? | Back to the team queue at once; logged. |

## Progress

- [x] UI draft (Figma, PNG, screen notes)
- [ ] Use cases, user stories, acceptance criteria
- [ ] Information architecture (screen IDs, routes, permissions, flows, UI states)
- [ ] Decide Q-UM-01 … Q-UM-04

**Related:** [README](../../README.md) · [Previous: F08 Activity Log](../f08-activity-log/README.md)
