# Features — F08 Activity Log

> Design status: TARGET V1
>
> Related: [Use cases](02-use-cases.md) · [User stories](03-user-stories-invest.md) · [Acceptance criteria](04-acceptance-criteria.md)

## Context

Most of the ~8,000 daily tickets are touched first by the AI (F04) and automation (F05): classification, questions, KB answers, approval within policy, API calls, verification. Technicians step in only when the agent cannot resolve, for system bugs and for software requirements. That only works if every step is recorded and can be shown later, to the right person, without leaking secrets.

**V1 goal:** one append-only audit trail written by every branch, a ticket history for everyone involved, an approval chain for access requests, a search over all actions for the Admin, and a list of workflow runs with their errors.

## Feature list

| ID | Feature | Main user | MoSCoW |
|---|---|---|---|
| F08.1 | **Ticket history.** Timeline of every change on a ticket (`#history` tab of SCR-T-02): actor (person, AI, automation), time, before / after, evidence links; requester sees an allowlist only | Technician, Requester, Admin | Must |
| F08.2 | **Activity log search.** Filter all entries by actor type, actor, action, target, evidence id, period; before / after diff | Admin | Must |
| F08.3 | **Approval chain.** Timeline Request → AI understanding → policy check → decision → execution → verification → confirmation → revoke, each step with result and reason; full decision detail stays in F05 SCR-WF-31; technician sees a summary with the reason, requester a status line | Admin, Technician, Requester | Must |
| F08.4 | **Workflow runs and errors.** Admin pages to list and filter runs across tickets by status, error type, operation, connector; attempts with masked request / response. Technicians see runs in the F05 `#automation` tab | Admin | Must |
| F08.5 | **AI trail.** Every AI analysis version, correction, KB auto-reply, escalation class and agent, with model and prompt version | Admin | Should |
| F08.6 | **Append-only writing with masking.** Nobody edits or deletes entries; secrets are masked before writing | System (all branches) | Must |
| F08.7 | **No external change without a log.** An access grant is executed only if its "execution started" entry was written | System (F05) | Must |

## Out of scope for F08

| Item | Where it belongs |
|---|---|
| Deciding, approving, retrying or resolving a workflow run | F05 UC-WF-07 … UC-WF-09 |
| The AI decision detail page (each policy condition with compared values, configuration version, confidence) | F05 SCR-WF-31 (F08 links to it) |
| The `#automation` tab of SCR-T-02, the run endpoints and what technicians see of a run | F05 (F08's SCR-NK-05 reuses the same component) |
| AI analysis versions as a tab of the decision page | F04 SEC-AI-15 |
| Ticket pages and their other tabs | F03 |
| Figures and rates computed from the log | F07 |
| Defining the audit record and the service accounts | F01 BR-09, BR-13 |
| Exporting the audit log | Not in V1 (Q-NK-10) |
| SIEM integration, alerting on suspicious activity | Future work |
